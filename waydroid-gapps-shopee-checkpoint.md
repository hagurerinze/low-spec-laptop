# Waydroid GAPPS on Debian 13 (Dell Inspiron) - Checkpoint

Date: 2026-10-31
Status: **Paused.** Shopee is installed, but it is **not running yet**.

## Goal

Run Shopee on a low-spec laptop with Waydroid (Android in a container), using the GAPPS image so Play Store works.

## My setup

| Item | Value |
|---|---|
| Laptop | Dell Inspiron |
| CPU | Intel Celeron N2815 |
| RAM | about 3.7 GB |
| GPU | Intel HD Graphics (Bay Trail), Mesa 25.0.7 |
| OS | Debian 13 (trixie) |
| Kernel | 6.12.113+deb13-amd64 |
| Waydroid | 1.6.3 |
| Weston | 14.0.2 |
| Image | LineageOS 20 GAPPS (system), MAINLINE (vendor) |

I log in to Weston alone (without XFCE) to save RAM.

## What I did (works)

### 1. Check the system first

- GPU uses hardware acceleration (not `llvmpipe`).
- Binder is a kernel module (`CONFIG_ANDROID_BINDER_IPC=m`). `BINDERFS` is not set.

### 2. Install packages and load binder

```bash
sudo apt update
sudo apt install -y weston waydroid
sudo modprobe binder_linux devices=binder,hwbinder,vndbinder
ls -l /dev/binder /dev/hwbinder /dev/vndbinder
```

Load the module on every boot:

```bash
echo "binder_linux" | sudo tee /etc/modules-load.d/binder.conf
echo "options binder_linux devices=binder,hwbinder,vndbinder" | sudo tee /etc/modprobe.d/binder.conf
```

### 3. Add user groups

```bash
sudo usermod -aG input,render hagure
```

Then log out and log in again.

### 4. Init Waydroid with GAPPS

```bash
sudo waydroid init -s GAPPS -f
sudo systemctl enable --now waydroid-container
```

### 5. Run it in Weston

From a TTY (Ctrl+Alt+F3), run `weston`. Then, in a terminal inside Weston:

```bash
waydroid session start &
waydroid show-full-ui
```

### 6. Check the network (it was fine)

The internet inside Android worked. These checks were OK:

- `waydroid0` bridge is UP (`192.168.240.1/24`)
- IP forwarding is `1`
- No `ufw`; Waydroid's own nftables rules allow traffic
- `waydroid shell -- ping 8.8.8.8` and `ping google.com` both work

### 7. Register the device with Google

GAPPS needs the device to be registered before Play Store login.

1. Get the Android ID from `sudo waydroid shell` (the `gservices.db` database).
2. Register it at `https://www.google.com/android/uncertified/`.
3. Wait a while, clear data of Play Store and Play Services, restart Waydroid, then sign in.

Device was registered and Play Store opened.

### 8. Shopee Lite only in Play Store

Play Store only showed **Shopee Lite**, not the main Shopee app. My guess: the main app has only ARM builds, and this Waydroid is x86_64. (This is a guess, not proven.)

I installed `libhoudini` (ARM translation) with `waydroid_script`:

```bash
waydroid session stop
sudo systemctl stop waydroid-container

sudo apt install -y python3-venv git lsb-release
git clone https://github.com/casualsnek/waydroid_script
cd waydroid_script
python3 -m venv venv
venv/bin/pip install -r requirements.txt
sudo venv/bin/python main.py install libhoudini

sudo systemctl start waydroid-container
```

The install finished. Shopee was then installed.

## Problems I met

| Problem | What happened | Fix |
|---|---|---|
| Vendor image hash error | `waydroid init` stopped: "vendor image hash doesn't match". Slow download was cut. | Run `waydroid init -s GAPPS -f` again. It worked. |
| `modinfo` / `sysctl` not found | `/sbin` is not in PATH for a normal user. | Ignore, or use `cat /proc/sys/net/ipv4/ip_forward`. |
| gservices.db cannot open | The path was cut when I pasted. It must be `com.google.android.gsf`. | Use the full path. |
| Screen glitches when the mouse moves in Weston | Not fixed. | Skipped for now. |
| Shopee: "isn't responding" | The app installs but I cannot get in. | **Not solved.** See below. |

## Not finished (TODO)

- [ ] **Shopee running.** It is only installed. I have no proof that it works yet. Ideas to try:
  - Press **Wait** (not Close) on the "isn't responding" dialog. First open may take 1-2 minutes.
  - Clear data and force stop only the Shopee app.
  - Check RAM with `free -h` while Shopee opens.
  - Check that ARM is on: `sudo waydroid shell -- getprop ro.product.cpu.abilist` (I have not posted this result).
  - If it is too slow on this CPU, use **Shopee Lite** or the website.
- [ ] **Setup notification in the Android notification bar.** I skipped it once because there was no connection. I want to know what it was. Two commands are **not run yet**:
  ```bash
  sudo waydroid shell -- pm list packages | grep -i -E "setup|wizard"
  sudo waydroid shell -- cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.SETUP_WIZARD
  ```
  Also check Settings > Notifications > Notification history. I will not open the Setup Wizard yet, because it may change settings.
- [ ] Time zone inside Android (`Asia/Jakarta`). Commands were given, result not checked.
- [ ] Weston mouse glitch.

## Notes

- A factory reset in Waydroid means deleting the `data` folder. Images, `libhoudini` and config stay. But apps and Google login are lost and the **Android ID changes**, so I must register the device again.
- `libhoudini` is a third-party script that changes the Waydroid image. Read its README before use.
