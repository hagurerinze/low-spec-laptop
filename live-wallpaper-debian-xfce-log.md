# Live Wallpaper on Debian XFCE — Project Log

**Hardware:** Debian, XFCE, 2GB RAM, Celeron N3xxx
**Stack:** xwinwrap + mpv (VAAPI hardware decode)
**Video:** `/home/hagure/Downloads/reimu_720p30.mp4`

---

## Current Working Setup (as of latest test)

### Wallpaper launch command
```bash
xwinwrap -b -s -fs -st -sp -nf -ov -fdt -ni -- mpv -wid WID \
  --input-ipc-server=/tmp/mpv-hagure/wallpaper.sock \
  --no-audio --hwdec=vaapi --vo=gpu --profile=fast --no-osc \
  --no-input-default-bindings --loop-file=inf \
  /home/hagure/Downloads/reimu_720p30.mp4
```

**Flag meanings:**
| Flag | Meaning |
|---|---|
| `-b` | Below other windows |
| `-s` | Sticky (all workspaces) |
| `-fs` | Fullscreen |
| `-st` | Skip taskbar |
| `-sp` | Skip pager |
| `-nf` | No focus |
| `-ov` | Override-redirect (window bypasses WM stacking control) |
| `-fdt` | Force window to report as `_NET_WM_WINDOW_TYPE_DESKTOP` |
| `-ni` | Ignore input (lets most clicks pass through to windows below) |

### Known limitations of this setup
- **Desktop icons stay hidden** — unresolved on XFCE (see "Failed Attempts" below)
- **Left-click-hold / drag doesn't work** — single-click, double-click, right-click, middle-click all pass through fine; sustained hold+drag does not
- Both limitations trace back to the same root cause: xfdesktop owns rendering *and* input handling for the icon/desktop layer, and can't cleanly coexist with an external override-redirect window. Confirmed via XFCE developer forum discussion — not a config mistake, a real architecture gap. There's an open, unresolved feature request for native video wallpaper support in xfdesktop.
- **Decision:** migrate to Openbox + `pcmanfm --desktop` to resolve both issues (icons live in a normal managed window there instead of being baked into the DE), rather than continuing to fight xfdesktop.

---

## Autostart Scripts (current, working)

### `~/.local/bin/livewallpaper-start`
```bash
#!/bin/bash
sleep 5
pkill -f livewallpaper-controller 2>/dev/null
mkdir -p /tmp/mpv-hagure

xwinwrap -b -s -fs -st -sp -nf -ov -fdt -ni -- mpv -wid WID \
  --input-ipc-server=/tmp/mpv-hagure/wallpaper.sock \
  --no-audio --hwdec=vaapi --vo=gpu --profile=fast --no-osc \
  --no-input-default-bindings --loop-file=inf \
  /home/hagure/Downloads/reimu_720p30.mp4
```
The `sleep 5` avoids racing XFCE's own startup (panel/xfdesktop). The `pkill` guard kills any leftover controller from a previous session before a fresh one starts — added after a stale-process bug (see Failed Attempts).

### `~/.local/bin/livewallpaper-controller`
```bash
#!/bin/bash
SOCK="/tmp/mpv-hagure/wallpaper.sock"
FLAG="/tmp/mpv-hagure/manual_pause"

mkdir -p /tmp/mpv-hagure
[ -f "$FLAG" ] || echo 0 > "$FLAG"

is_browser() {
    local class
    class=$(xprop -id "$1" WM_CLASS 2>/dev/null)
    echo "$class" | grep -Eiq \
        '"(google-chrome|chromium|firefox|librewolf|brave-browser|microsoft-edge)"'
}

is_tlauncher() {
    local class title
    class=$(xprop -id "$1" WM_CLASS 2>/dev/null)
    title=$(xdotool getwindowname "$1" 2>/dev/null)
    echo "$class" | grep -Eiq '"tlauncher"' && return 0
    echo "$title" | grep -Eiq 'tlauncher' && return 0
    return 1
}

is_fullscreen_or_maximized() {
    local state
    state=$(xprop -id "$1" _NET_WM_STATE 2>/dev/null)
    echo "$state" | grep -Eq \
        '_NET_WM_STATE_FULLSCREEN|_NET_WM_STATE_MAXIMIZED_VERT|_NET_WM_STATE_MAXIMIZED_HORZ'
}

pause_wallpaper() {
    printf '%s\n' '{"command":["set_property","pause",true]}' \
        | socat - UNIX-CONNECT:"$SOCK" >/dev/null 2>&1
}
resume_wallpaper() {
    printf '%s\n' '{"command":["set_property","pause",false]}' \
        | socat - UNIX-CONNECT:"$SOCK" >/dev/null 2>&1
}

while true; do
    MANUAL_PAUSE=$(cat "$FLAG" 2>/dev/null || echo 0)

    if [ "$MANUAL_PAUSE" -eq 1 ]; then
        pause_wallpaper
        sleep 1
        continue
    fi

    ACTIVE=$(xdotool getactivewindow 2>/dev/null)
    if [ -z "$ACTIVE" ]; then
        sleep 1
        continue
    fi

    CLASS=$(xprop -id "$ACTIVE" WM_CLASS 2>/dev/null)
    if echo "$CLASS" | grep -q '"mpvk", "mpv"'; then
        resume_wallpaper
        sleep 1
        continue
    fi

    if is_browser "$ACTIVE"; then
        pause_wallpaper
    elif is_tlauncher "$ACTIVE"; then
        pause_wallpaper
    elif is_fullscreen_or_maximized "$ACTIVE"; then
        pause_wallpaper
    else
        resume_wallpaper
    fi

    sleep 1
done
```
Auto-pauses on: browser windows, tlauncher, any fullscreen/maximized window. Auto-resumes otherwise — unless manually paused via the flag file.

### `~/.local/bin/livewallpaper-toggle` (bound to `Ctrl+Alt+P`)
```bash
#!/bin/bash
FLAG="/tmp/mpv-hagure/manual_pause"
SOCK="/tmp/mpv-hagure/wallpaper.sock"
LOCK="/tmp/mpv-hagure/toggle.lock"

mkdir -p /tmp/mpv-hagure
[ -f "$FLAG" ] || echo 0 > "$FLAG"

if [ -f "$LOCK" ]; then
    LAST=$(stat -c %Y "$LOCK" 2>/dev/null || echo 0)
    NOW=$(date +%s)
    if [ $((NOW - LAST)) -lt 1 ]; then
        exit 0
    fi
fi
touch "$LOCK"

CURRENT=$(cat "$FLAG")

if [ "$CURRENT" -eq 0 ]; then
    echo 1 > "$FLAG"
    printf '%s\n' '{"command":["set_property","pause",true]}' \
        | socat - UNIX-CONNECT:"$SOCK" >/dev/null 2>&1
else
    echo 0 > "$FLAG"
    printf '%s\n' '{"command":["set_property","pause",false]}' \
        | socat - UNIX-CONNECT:"$SOCK" >/dev/null 2>&1
fi
```
1-second debounce lock added to absorb keyboard key-repeat double-firing.

### Autostart `.desktop` entries
`~/.config/autostart/livewallpaper.desktop`:
```ini
[Desktop Entry]
Type=Application
Name=Live Wallpaper
Exec=/home/hagure/.local/bin/livewallpaper-start
X-GNOME-Autostart-enabled=true
NoDisplay=false
```

`~/.config/autostart/livewallpaper-controller.desktop`:
```ini
[Desktop Entry]
Type=Application
Name=Live Wallpaper Controller
Exec=/home/hagure/.local/bin/livewallpaper-controller
X-GNOME-Autostart-enabled=true
NoDisplay=false
```

---

## Failed Attempts Log

### 1. `-ovr` instead of `-ov` (with `-fs`)
```bash
xwinwrap -b -s -fs -st -sp -nf -ovr -fdt -- mpv -wid WID ...
```
**Result:** Made things worse — wallpaper took over the entire screen, blocked all input, and made another display disappear. Reverted immediately.

### 2. Dropping `-ov` entirely (kept `-fdt`)
```bash
xwinwrap -b -s -fs -st -sp -nf -fdt -ni -- mpv -wid WID ...
```
**Result:** Window became visible to `wmctrl -l` (unlike the `-ov` version), but conky disappeared, desktop icons stayed hidden, and all mouse input stopped working. Because `-fdt` marks the window as `_NET_WM_WINDOW_TYPE_DESKTOP`, and without `-ov` the WM treats it as *the* desktop window — effectively replacing xfdesktop's role instead of sitting below it.

### 3. `xfdesktop --reload`
```bash
sleep 1.5
xfdesktop --reload
```
**Result:** Safe (didn't break anything), but had zero effect on icon visibility. Confirmed the icon layer doesn't re-evaluate its stacking position on reload.

### 4. `xfdesktop --quit` + restart
```bash
xfdesktop --quit
sleep 0.3
xfdesktop &
```
**Result:** Killed the wallpaper entirely. Since `-fdt` marks the video window as desktop-type, xfdesktop's quit routine tears down desktop-type windows/pixmaps as part of its own cleanup — collateral damage.

### 5. `xdotool`/`wmctrl` targeting the wallpaper window directly
```bash
xdotool search --class "xwinwrap"
xdotool search --name "mpv"
wmctrl -l
```
**Result:** Wallpaper window never appears in either tool's output. This is expected X11 behavior — override-redirect (`-ov`) windows are invisible to `_NET_CLIENT_LIST`, which both tools query. There was never a valid window ID to grab this way.

### 6. Raising xfdesktop's own "Desktop" window via wmctrl
```bash
wmctrl -a "Desktop"
```
**Result:** No visible change.

```bash
wmctrl -r "Desktop" -b add,above
```
**Result:** Broke the whole session's window stacking — wallpaper became the only visible thing, panels disappeared, open apps stopped rendering (though still alive per Alt+Tab). Had to recover via TTY:
```bash
export DISPLAY=:0
wmctrl -r "Desktop" -b remove,above
xfwm4 --replace &
```
This restored normal stacking without a full logout.

### 7. Stale controller process bug
**Symptom:** Manual `set_property pause true` commands would pause the wallpaper for about one second, then it would resume on its own — even with no keyboard shortcut involved.
**Cause:** An old copy of `livewallpaper-controller` (from before the `FLAG`-file logic was added) was still running in the background from an earlier session/autostart launch. Its original loop called `resume_wallpaper()` every second regardless of manual pause state, since it had no concept of the flag file — it was simply overriding any pause within ~1 second, every time.
**Fix:**
```bash
pkill -f livewallpaper-controller
~/.local/bin/livewallpaper-controller &
```
Root-caused and now prevented going forward by the `pkill -f livewallpaper-controller` guard added to the top of `livewallpaper-start`.

### 8. Keyboard shortcut — "Permission denied"
**Symptom:** XFCE popup: `Failed to launch shortcut "<Primary><Alt>p" — Failed to execute child process "/home/hagure/.local/bin/livewallpaper-toggle" (Permission denied)`
**Cause:** Script lost/never had execute permission after an edit.
**Fix:**
```bash
chmod +x ~/.local/bin/livewallpaper-toggle
```

---

## Verdict

The core wallpaper + pause/resume/toggle system is stable and working on XFCE as of this log. The two remaining unresolved issues (hidden desktop icons, blocked click-hold) are architectural limitations of xfdesktop, not bugs in this setup — confirmed against XFCE developer commentary and an open, unresolved feature request. Planned resolution: migrate desktop environment to **Openbox + pcmanfm --desktop**, which handles desktop icons as a normal managed window instead of baking them into the DE, removing the conflict entirely. The wallpaper command itself should carry over to Openbox largely unchanged.
