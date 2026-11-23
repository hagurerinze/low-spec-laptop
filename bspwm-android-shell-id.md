# Bspwm Android-Style Shell

Catatan lengkap sesi 22 November 2026. Isinya: ide, langkah, masalah yang muncul beserta perbaikannya, dan kode akhir.

Target layar: 1366x768, Debian, bspwm + Polybar + picom.

---

## 1. Ide

Seluruh layar desktop dibuat seperti layar Android, dua sisi yang selalu kelihatan:

- **Kiri (home screen):** jam dan tanggal besar, kotak search Brave, favorit aplikasi, widget CPU / RAM / Disk / Baterai.
- **Kanan (app drawer):** kotak search aplikasi dan grid ikon semua aplikasi yang bisa di-scroll.
- **Tombol `[≡]` di Polybar:** klik kiri untuk sembunyikan atau tampilkan drawer, klik kanan untuk menu power (Shutdown, Restart, Logout).
- Shell muncul di semua desktop dan berada di belakang jendela lain.

Bahan awalnya prototype GTK3 dari GPT (semua elemen masih `Gtk.Label`), lalu dikembangkan di sesi ini.

---

## 2. Yang dibutuhkan

```sh
sudo apt install python3-gi gir1.2-gtk-3.0 gir1.2-gdkx11-3.0 picom feh polybar
```

Struktur file:

```
~/bspwm-shell/bspwm_shell.py
~/.config/bspwm/bspwmrc
~/.config/polybar/config.ini
~/Desktop/*.desktop        <- sumber favorit aplikasi
```

---

## 3. Urutan pengerjaan

1. Prototype dari GPT jalan, tapi semua masih label.
2. Search jadi `Gtk.Entry`, drawer jadi `FlowBox` di dalam `ScrolledWindow`, daftar app dari `Gio.AppInfo.get_all()`, jam hidup.
3. Perbaikan error dan tampilan (lihat bagian 4).
4. Favorit aplikasi dari folder `~/Desktop`.
5. Window dijadikan window biasa supaya keyboard dan klik bisa masuk, lalu dipaksa ke belakang pakai aturan bspwm.
6. Fokus awal tidak nyangkut ke search.
7. Widget CPU, RAM, Disk.
8. Tombol `[≡]` dengan sinyal ke skrip.
9. Widget baterai, dan perbaikan lebar akibat widget keempat.

---

## 4. Masalah dan perbaikan

| Masalah | Penyebab | Perbaikan |
|---|---|---|
| `TypeError: Argument 0 does not allow None` | Ada aplikasi tanpa ikon, `app.get_icon()` ngasih `None` | Ikon cadangan `application-x-executable` |
| `SyntaxError: 'return' outside function` | Indentasi hilang saat paste | Ganti seluruh fungsi sekaligus, cek dengan `python3 -m py_compile` |
| Teks nempel ke tepi panel, jarak tengah besar | `set_border_width` itu margin di luar panel | Pakai `padding` lewat CSS |
| Background hitam | Tidak ada wallpaper dan compositor | `feh` untuk wallpaper, `picom --backend xrender` |
| picom error "Backend not specified" | picom versi baru wajib pilih backend | Tambahkan `--backend xrender` |
| Panel ketutup Polybar | Window mulai dari y=0 | Window dipindah ke y=30 dengan tinggi 738 |
| Search tidak bisa diketik, drawer tidak bisa di-scroll atau diklik | Window bertipe `DESKTOP` tidak dikelola dan tidak dapat fokus bspwm | Tipe jadi `NORMAL`, lalu aturan `bspc rule` supaya floating, di belakang, dan sticky |
| Window tidak ke belakang | Aturan memakai class yang salah | Cek dengan `xprop WM_CLASS`, class-nya `Bspwm_shell.py` |
| Kursor otomatis di search Brave | GTK memberi fokus ke entry pertama | Kotak utama jadi tempat fokus kosong, klik area kosong atau `Esc` untuk kembali idle |
| Hasil search sedikit, kotak memanjang | `FlowBox` meregang | `grid.set_valign(Gtk.Align.START)` |
| Baris terakhir drawer terpotong | Tinggi baris tidak pas dengan tinggi area | Tinggi baris dihitung dari tinggi area scroll (`fit_rows`) |
| Bawah panel kepotong | Tinggi isi lebih besar dari window | Rapatkan padding dan spacing, tinggi window 738 |
| Menu power tidak muncul saat klik kanan | Tombol mouse masih tertekan saat menu dibuka | Jeda 200 ms sebelum menu dibuka |
| Gap di Polybar `[≡]   1` | `module-margin` dan `padding-left` | `module-margin = 0`, `padding-left = 0`, `content-padding = 2` |
| `bspc node -c` menutup shell | Window tidak dikunci | `bspc node <id> -g locked=on` dari dalam skrip, setelah window muncul |
| `locked=on` di `bspc rule` bikin shell jadi window biasa | Aturan dengan `locked=on` tidak dipakai di setup ini | Kunci lewat skrip, bukan lewat aturan |
| Semua kegeser keluar layar setelah widget baterai | Lebar minimum progress bar, empat widget bikin panel kiri melebar | `min-width` progress bar dikecilkan dan font nilai jadi 14px |

---

## 5. Konfigurasi

### `~/.config/bspwm/bspwmrc` (bagian yang relevan)

```sh
# Polybar
pkill -x polybar 2>/dev/null || true
polybar -c "$HOME/.config/polybar/config.ini" main &

# Wallpaper
feh --bg-fill "$HOME/Pictures/wallpaper.jpg" &

# Compositor (untuk transparansi)
pkill -x picom 2>/dev/null || true
picom --backend xrender &

# Aturan window shell (harus sebelum skripnya jalan)
bspc rule -a Bspwm_shell.py state=floating layer=below sticky=on border=off

# Jalankan shell
pkill -f "[b]spwm_shell.py" 2>/dev/null || true
python3 "$HOME/bspwm-shell/bspwm_shell.py" &
```

Jangan taruh `locked=on` di aturan. Kunci node dilakukan dari skrip.

### `~/.config/polybar/config.ini` (bagian yang berubah)

```ini
[bar/main]
padding-left = 0
module-margin = 0

[module/launcher]
type = custom/text
content = [≡]
content-foreground = #e8eaed
content-padding = 2
click-left = pkill -USR1 -f "[b]spwm_shell.py"
click-right = pkill -USR2 -f "[b]spwm_shell.py"
```

Penulisan `[b]spwm_shell.py` itu sengaja, supaya `pkill` tidak ikut menembak shell yang menjalankan perintahnya sendiri.

---

## 6. Kode akhir: `bspwm_shell.py`

```python
#!/usr/bin/env python3

import glob
import os
import shutil
import signal
import subprocess
from urllib.parse import quote_plus

import gi
gi.require_version("Gtk", "3.0")
gi.require_version("GdkX11", "3.0")
from gi.repository import Gtk, Gdk, Gio, GLib, GdkX11

CSS = b"""
window {
    background-color: transparent;
}

.left-panel {
    background-color: rgba(37, 42, 53, 0.55);
    border-radius: 16px;
    padding: 24px;
}

.right-panel {
    background-color: rgba(25, 28, 35, 0.55);
    border-radius: 16px;
    padding: 24px;
}

.clock {
    color: #ffffff;
    font-size: 64px;
    font-weight: bold;
}

.date {
    color: #dfe5ee;
    font-size: 18px;
}

.heading {
    color: #ffffff;
    font-size: 24px;
    font-weight: bold;
}

.description {
    color: #dfe5ee;
    font-size: 14px;
}

entry.search {
    background-color: #353b47;
    color: #ffffff;
    border-radius: 24px;
    padding: 12px 18px;
    font-size: 16px;
}

button.app {
    background: transparent;
    border: none;
    box-shadow: none;
    border-radius: 12px;
    padding: 8px;
}

button.app:hover {
    background-color: rgba(255, 255, 255, 0.15);
}

button.app label {
    color: #ffffff;
    font-size: 12px;
}

scrolledwindow {
    background: transparent;
}

.main:focus {
    outline: none;
}

.widget {
    background-color: rgba(255, 255, 255, 0.08);
    border-radius: 12px;
    padding: 12px;
}

.widget-title {
    color: #dfe5ee;
    font-size: 12px;
}

.widget-value {
    color: #ffffff;
    font-size: 14px;
    font-weight: bold;
}

progressbar trough {
    min-height: 6px;
    min-width: 10px;
    background-color: rgba(255, 255, 255, 0.15);
    border-radius: 3px;
}

progressbar progress {
    min-height: 6px;
    min-width: 0px;
    background-color: #5294e2;
    border-radius: 3px;
}
"""

BROWSER = "brave-browser"  # ganti kalau nama perintahnya beda


def apply_css():
    provider = Gtk.CssProvider()
    provider.load_from_data(CSS)
    Gtk.StyleContext.add_provider_for_screen(
        Gdk.Screen.get_default(),
        provider,
        Gtk.STYLE_PROVIDER_PRIORITY_APPLICATION,
    )


def make_label(text, css_class, align=Gtk.Align.START):
    lbl = Gtk.Label(label=text)
    lbl.get_style_context().add_class(css_class)
    lbl.set_halign(align)
    return lbl


def make_search(placeholder):
    entry = Gtk.Entry()
    entry.set_placeholder_text(placeholder)
    entry.get_style_context().add_class("search")
    return entry


# ---------- Favorit dari ~/Desktop ----------

def load_desktop_favorites():
    apps = []
    folder = os.path.expanduser("~/Desktop")
    for path in sorted(glob.glob(os.path.join(folder, "*.desktop"))):
        app = Gio.DesktopAppInfo.new_from_filename(path)
        if app is not None:
            apps.append(app)
    return apps


def make_app_button(app, size=56):
    btn = Gtk.Button()
    btn.get_style_context().add_class("app")
    btn.set_relief(Gtk.ReliefStyle.NONE)

    box = Gtk.Box(orientation=Gtk.Orientation.VERTICAL, spacing=4)
    gicon = app.get_icon()
    if gicon is None:
        gicon = Gio.ThemedIcon.new("application-x-executable")
    icon = Gtk.Image.new_from_gicon(gicon, Gtk.IconSize.DIALOG)
    icon.set_pixel_size(size)
    name = Gtk.Label(label=app.get_name())
    name.set_ellipsize(3)  # Pango.EllipsizeMode.END
    name.set_max_width_chars(12)
    box.pack_start(icon, False, False, 0)
    box.pack_start(name, False, False, 0)
    btn.add(box)
    btn.connect("clicked", lambda b, a=app: a.launch([], None))
    return btn


# ---------- Widget sistem ----------

_last_cpu = [0, 0]


def read_cpu():
    with open("/proc/stat") as f:
        vals = list(map(int, f.readline().split()[1:]))
    idle = vals[3] + vals[4]
    total = sum(vals)
    d_total = total - _last_cpu[0]
    d_idle = idle - _last_cpu[1]
    _last_cpu[0], _last_cpu[1] = total, idle
    if d_total <= 0:
        return 0.0
    return 100.0 * (1 - d_idle / d_total)


def read_ram():
    info = {}
    with open("/proc/meminfo") as f:
        for line in f:
            key, val = line.split(":")
            info[key] = int(val.split()[0])
    total = info["MemTotal"]
    used = total - info["MemAvailable"]
    return used / total, used / 1048576, total / 1048576


def read_disk():
    d = shutil.disk_usage("/")
    return d.used / d.total, d.used / 1e9, d.total / 1e9


def find_battery():
    for p in sorted(glob.glob("/sys/class/power_supply/BAT*")):
        return p
    return None


def read_battery(path):
    try:
        with open(path + "/capacity") as f:
            cap = int(f.read())
        with open(path + "/status") as f:
            status = f.read().strip()
    except (OSError, ValueError):
        return None
    return cap, status


def make_widget(title):
    box = Gtk.Box(orientation=Gtk.Orientation.VERTICAL, spacing=6)
    box.get_style_context().add_class("widget")
    t = make_label(title, "widget-title")
    v = make_label("-", "widget-value")
    bar = Gtk.ProgressBar()
    box.pack_start(t, False, False, 0)
    box.pack_start(v, False, False, 0)
    box.pack_start(bar, False, False, 0)
    return box, v, bar


def build_widgets():
    row = Gtk.Box(orientation=Gtk.Orientation.HORIZONTAL, spacing=12)
    row.set_homogeneous(True)

    cpu_box, cpu_val, cpu_bar = make_widget("CPU")
    ram_box, ram_val, ram_bar = make_widget("RAM")
    disk_box, disk_val, disk_bar = make_widget("DISK")

    row.pack_start(cpu_box, True, True, 0)
    row.pack_start(ram_box, True, True, 0)
    row.pack_start(disk_box, True, True, 0)

    bat_path = find_battery()
    if bat_path:
        bat_box, bat_val, bat_bar = make_widget("BATTERY")
        row.pack_start(bat_box, True, True, 0)

    def update():
        cpu = read_cpu()
        cpu_val.set_text("%d%%" % cpu)
        cpu_bar.set_fraction(cpu / 100.0)

        frac, used, total = read_ram()
        ram_val.set_text("%.1f / %.1f GB" % (used, total))
        ram_bar.set_fraction(frac)

        frac, used, total = read_disk()
        disk_val.set_text("%d / %d GB" % (used, total))
        disk_bar.set_fraction(frac)

        if bat_path:
            r = read_battery(bat_path)
            if r:
                cap, status = r
                mark = " +" if status == "Charging" else ""
                bat_val.set_text("%d%%%s" % (cap, mark))
                bat_bar.set_fraction(cap / 100.0)
        return True

    read_cpu()  # inisialisasi pembanding CPU
    update()
    GLib.timeout_add_seconds(2, update)
    return row


# ---------- Panel kiri ----------

def build_left():
    left = Gtk.Box(orientation=Gtk.Orientation.VERTICAL, spacing=14)
    left.get_style_context().add_class("left-panel")

    clock = make_label("", "clock")
    date = make_label("", "date")

    def tick():
        now = GLib.DateTime.new_now_local()
        clock.set_text(now.format("%H:%M"))
        date.set_text(now.format("%A, %d %B %Y"))
        return True

    tick()
    GLib.timeout_add_seconds(1, tick)

    web = make_search("Search the web with Brave")

    def on_web_enter(entry):
        q = entry.get_text().strip()
        if q:
            url = "https://search.brave.com/search?q=" + quote_plus(q)
            subprocess.Popen([BROWSER, url])
            entry.set_text("")

    web.connect("activate", on_web_enter)

    favorites = make_label("Favorite applications", "description")

    fav_grid = Gtk.FlowBox()
    fav_grid.set_max_children_per_line(4)
    fav_grid.set_min_children_per_line(4)
    fav_grid.set_selection_mode(Gtk.SelectionMode.NONE)
    fav_grid.set_homogeneous(True)
    for app in load_desktop_favorites():
        fav_grid.add(make_app_button(app))

    widgets = build_widgets()
    widgets.set_valign(Gtk.Align.CENTER)
    widgets.set_margin_top(10)

    left.pack_start(clock, False, False, 0)
    left.pack_start(date, False, False, 0)
    left.pack_start(web, False, False, 0)
    left.pack_start(favorites, False, False, 0)
    left.pack_start(fav_grid, False, False, 0)
    left.pack_start(widgets, True, True, 0)
    return left


# ---------- Panel kanan (app drawer) ----------

def build_right():
    right = Gtk.Box(orientation=Gtk.Orientation.VERTICAL, spacing=16)
    right.get_style_context().add_class("right-panel")

    title = make_label("APP DRAWER", "heading")
    search = make_search("Search applications")

    scroll = Gtk.ScrolledWindow()
    scroll.set_policy(Gtk.PolicyType.NEVER, Gtk.PolicyType.AUTOMATIC)

    grid = Gtk.FlowBox()
    grid.set_valign(Gtk.Align.START)
    grid.set_max_children_per_line(4)
    grid.set_min_children_per_line(4)
    grid.set_selection_mode(Gtk.SelectionMode.NONE)
    grid.set_homogeneous(True)
    grid.set_row_spacing(8)
    grid.set_column_spacing(8)

    apps = [a for a in Gio.AppInfo.get_all() if a.should_show()]
    apps.sort(key=lambda a: a.get_name().lower())

    buttons = []
    for app in apps:
        btn = Gtk.Button()
        btn.get_style_context().add_class("app")
        btn.set_relief(Gtk.ReliefStyle.NONE)

        box = Gtk.Box(orientation=Gtk.Orientation.VERTICAL, spacing=4)
        gicon = app.get_icon()
        if gicon is None:
            gicon = Gio.ThemedIcon.new("application-x-executable")
        icon = Gtk.Image.new_from_gicon(gicon, Gtk.IconSize.DIALOG)
        icon.set_pixel_size(48)
        name = Gtk.Label(label=app.get_name())
        name.set_ellipsize(3)
        name.set_max_width_chars(12)
        box.pack_start(icon, False, False, 0)
        box.pack_start(name, False, False, 0)
        btn.add(box)
        btn.connect("clicked", lambda b, a=app: a.launch([], None))
        buttons.append(btn)

        child = Gtk.FlowBoxChild()
        child.add(btn)
        child.app_name = app.get_name().lower()
        grid.add(child)

    def filter_func(child):
        q = search.get_text().strip().lower()
        return q in child.app_name

    grid.set_filter_func(filter_func)
    search.connect("changed", lambda e: grid.invalidate_filter())

    def on_search_enter(entry):
        for child in grid.get_children():
            if filter_func(child):
                child.get_child().clicked()
                entry.set_text("")
                break

    search.connect("activate", on_search_enter)

    scroll.add(grid)

    # Tinggi baris dihitung dari tinggi area scroll, supaya baris
    # terakhir (pada posisi scroll paling atas) tidak terpotong.
    ROW_GAP = 8  # harus sama dengan grid.set_row_spacing
    BASE_ROW = 98
    last = [0]

    def fit_rows(widget, alloc):
        h = alloc.height
        if h < BASE_ROW:
            return
        n = max(1, h // BASE_ROW)
        row_h = int((h - (n - 1) * ROW_GAP) / n)
        if row_h == last[0]:
            return
        last[0] = row_h
        for b in buttons:
            b.set_size_request(-1, row_h)

    scroll.connect("size-allocate", fit_rows)

    right.pack_start(title, False, False, 0)
    right.pack_start(search, False, False, 0)
    right.pack_start(scroll, True, True, 0)
    return right


# ---------- Main ----------

def main():
    GLib.set_prgname("bspwmshell")
    apply_css()

    window = Gtk.Window(title="Bspwm Shell Prototype")
    visual = Gdk.Screen.get_default().get_rgba_visual()
    if visual is not None:
        window.set_visual(visual)

    window.set_decorated(False)
    window.set_type_hint(Gdk.WindowTypeHint.NORMAL)
    window.move(0, 30)  # 30 = tinggi Polybar
    window.set_default_size(1366, 738)
    window.connect("destroy", Gtk.main_quit)

    main_box = Gtk.Box(orientation=Gtk.Orientation.HORIZONTAL, spacing=8)
    main_box.set_homogeneous(True)  # dua panel lebarnya sama
    main_box.set_can_focus(True)
    main_box.get_style_context().add_class("main")
    main_box.set_border_width(8)
    window.add(main_box)

    main_box.pack_start(build_left(), True, True, 0)
    right = build_right()
    main_box.pack_start(right, True, True, 0)

    window.show_all()

    # Fokus awal ke kotak kosong, bukan ke search
    window.set_focus(main_box)

    def on_click(w, event):
        window.set_focus(main_box)
        return False

    def on_key(w, event):
        if event.keyval == Gdk.KEY_Escape:
            window.set_focus(main_box)
            return True
        return False

    window.add_events(Gdk.EventMask.BUTTON_PRESS_MASK)
    window.connect("button-press-event", on_click)
    window.connect("key-press-event", on_key)

    # Tombol [≡] di Polybar: klik kiri = SIGUSR1, klik kanan = SIGUSR2
    def toggle_drawer():
        right.set_visible(not right.get_visible())
        return True

    power_menu = Gtk.Menu()
    for label, cmd in [
        ("Shutdown", ["systemctl", "poweroff"]),
        ("Restart", ["systemctl", "reboot"]),
        ("Logout", ["bspc", "quit"]),
    ]:
        item = Gtk.MenuItem(label=label)
        item.connect("activate", lambda i, c=cmd: subprocess.Popen(c))
        power_menu.append(item)
    power_menu.show_all()

    def show_power_menu():
        def popup():
            rect = Gdk.Rectangle()
            rect.x, rect.y, rect.width, rect.height = 0, 0, 1, 1
            power_menu.popup_at_rect(
                window.get_window(), rect,
                Gdk.Gravity.NORTH_WEST, Gdk.Gravity.NORTH_WEST, None
            )
            return False
        GLib.timeout_add(200, popup)  # jeda supaya menu tidak langsung tertutup
        return True

    GLib.unix_signal_add(GLib.PRIORITY_DEFAULT, signal.SIGUSR1, toggle_drawer)
    GLib.unix_signal_add(GLib.PRIORITY_DEFAULT, signal.SIGUSR2, show_power_menu)

    # Kunci node supaya `bspc node -c` tidak menutup shell
    def lock_node():
        xid = window.get_window().get_xid()
        subprocess.Popen(["bspc", "node", hex(xid), "-g", "locked=on"])
        return False

    GLib.timeout_add(1000, lock_node)

    Gtk.main()


if __name__ == "__main__":
    main()
```

---

## 7. Perintah yang sering dipakai

```sh
# Cek syntax
python3 -m py_compile ~/bspwm-shell/bspwm_shell.py

# Restart shell
pkill -f "[b]spwm_shell.py"
python3 ~/bspwm-shell/bspwm_shell.py &

# Tes tombol tanpa Polybar
pkill -USR1 -f "[b]spwm_shell.py"    # sembunyikan atau tampilkan drawer
pkill -USR2 -f "[b]spwm_shell.py"    # menu power

# Cek class dan ukuran window
xprop WM_CLASS
xwininfo -name "Bspwm Shell Prototype" | grep -E "Width|Height"

# Cek aturan bspwm
bspc rule -l

# Reload bspwm
bspc wm -r
```

---

## 8. Catatan

- Aturan `bspc rule` hanya berlaku untuk window yang baru muncul, jadi urutannya harus aturan dulu, baru skrip.
- Favorit dibaca dari `~/Desktop` hanya saat skrip start. Kalau menambah shortcut baru, restart skripnya.
- Shutdown dan Restart langsung jalan tanpa konfirmasi.
- `bspc node -k` (kill paksa) tetap bisa mematikan shell, dan itu sengaja sebagai jalan keluar kalau skripnya hang.
- Widget baterai otomatis tidak muncul kalau tidak ada `BAT*` di `/sys/class/power_supply/`.
- Disk yang dibaca adalah partisi `/`.
- Masih belum dikerjakan: konfirmasi untuk Shutdown dan Restart, widget cuaca, dan `Esc` untuk mengosongkan kotak search drawer.
