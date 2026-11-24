# Debian: Turn Off Services I Don't Need (Low-Spec Laptop)

My laptop has 2GB RAM and a normal HDD. It runs Debian 13 with systemd.
I wanted less RAM use and a faster boot, so I checked every service one by one.

Date: November 2026

> These numbers are from my own laptop. Your results will be different.

---

## Result (honest numbers)

I measured with `systemd-analyze` after each step.

| Step | Firmware | Loader | Kernel | Userspace | Total |
|---|---|---|---|---|---|
| Start | 9.1s | 20.0s | 13.6s | 32.0s | 1 min 14.8s |
| After turning off services | 9.2s | 15.5s | 13.4s | 30.8s | 1 min 8.9s |
| After smaller initramfs | 9.5s | 8.9s | 11.3s | 31.8s | 1 min 1.5s |

What I learned:

- The **loader** time changed a lot between boots (20s, 15s, 9s). I did not change GRUB. So most of the total gain is **not** from my changes.
- The **kernel** time went down about 2 seconds after the smaller initramfs. This one is real.
- The **userspace** time stayed about the same (30-32s). The services I turned off were small.
- The boot is slow mostly because the **HDD is slow**. Many services wait for the disk at the same time. An SSD is the only big fix.
- `NetworkManager-wait-online` took 5.4s in the blame list, but it did not make the login screen slower in a big way.
- For RAM, the bigger gain was closing apps (fluidsynth, KDE Connect), not services.

---

## Step 1: See what is running

Running now:

```
systemctl list-units --type=service --state=running | cat
```

Starts at boot:

```
systemctl list-unit-files --type=service --state=enabled | cat
```

User services (only for your user):

```
systemctl --user list-units --type=service --state=running | cat
```

Adding `| cat` stops the output from being cut off.

---

## Step 2: Turn off what I don't need

```
sudo systemctl disable --now NAME.service
```

`disable` means it will not start at boot. `--now` stops it right away.
To turn it back on: `sudo systemctl enable --now NAME.service`

### What I turned off

| Service | What it does | Why I turned it off |
|---|---|---|
| apache2 | web server | I don't run a website |
| winbind | Samba domain login | not used at home |
| lxc-monitord, lxcfs, lxc, lxc-net | containers | I only need LXC for Waydroid, and I start it by hand |
| cups, cups-browsed | printing | I don't print |
| avahi-daemon | network device discovery | not needed |
| switcheroo-control | switch between two GPUs | I have only one GPU |
| preload | loads apps into RAM early | not useful on 2GB RAM and an HDD |
| NetworkManager-wait-online | boot waits for network | not needed on a desktop |
| blueman-mechanism | Bluetooth manager helper | Blueman starts it again when needed |
| e2scrub_reap | checks old LVM snapshots | I don't use LVM |

`colord` (color profiles) started again by itself, so I used `mask`:

```
sudo systemctl mask colord
```

If `cups` starts again after disable, also run:

```
sudo systemctl disable --now cups.socket cups.path
```

### What I kept on purpose

- `dbus`, `systemd-*`, `polkit`, `udisks2`, `upower`, `accounts-daemon`
- `NetworkManager`, `NetworkManager-dispatcher`, `wpa_supplicant` (WiFi)
- `lightdm` (login screen), `cron`, `anacron`, `apparmor`
- `zramswap` (my zram)
- `smartmontools` and my own `smart-check` (HDD health)
- `vnstat` (network data counter, very small)
- `bluetooth` is disabled at boot. I use a Bluetooth headset, so I start it by hand when I need it (see Step 6)

Do **not** turn off `dbus`, `systemd-*`, `NetworkManager`, `udev`, or your display manager.

---

## Step 3: Read the boot time

```
systemd-analyze
systemd-analyze blame | head -20
systemd-analyze critical-chain
```

- `systemd-analyze` shows firmware, loader, kernel and userspace time.
- `blame` shows how long each service took. The times **overlap**, so you cannot add them.
- `critical-chain` shows what really delays the login screen. A service with a big number in `blame` may not be on this chain at all.

Example: `blueman-mechanism` was 16.8s in `blame`, but it was not on the critical chain.

---

## Step 4: Check RAM

```
free -h
ps aux --sort=-%mem | head -15
```

At the time (XFCE session), 925MB was used and 876MB was available. Swap was not used.

Biggest items I found (the numbers are approximate, they include shared memory):

| Program | RAM | What I did |
|---|---|---|
| fluidsynth | ~175MB | MIDI sound player. I only use MIDI sometimes, so I turned it off |
| kdeconnectd | ~89MB | KDE Connect. I removed it from autostart |
| blueman-applet + blueman-tray | ~100MB | Bluetooth tray. I keep Bluetooth, but only start it when needed |

`fluidsynth` was enabled for all users, so this did not work alone:

```
systemctl --user disable --now fluidsynth
```

This is the command that keeps it off:

```
sudo systemctl --global disable fluidsynth
```

To turn it back on: `sudo systemctl --global enable fluidsynth`

---

## Step 5: Smaller initramfs (faster kernel part)

The initramfs has many drivers by default (`MODULES=most`). `MODULES=dep` only puts in the drivers this laptop needs. A smaller file loads faster from an HDD.

Make a backup first:

```
sudo cp /boot/initrd.img-$(uname -r) ~/initrd.bak
```

Then:

```
grep ^MODULES /etc/initramfs-tools/initramfs.conf
sudo sed -i 's/^MODULES=most/MODULES=dep/' /etc/initramfs-tools/initramfs.conf
sudo update-initramfs -u
ls -lh /boot/initrd.img-$(uname -r)
```

My kernel time went from 13.4s to 11.3s.

**Risk:** if you move this HDD to a different laptop, it may not boot. To go back, set `MODULES=most` and run `sudo update-initramfs -u`.

**Safety:** `update-initramfs -u` only changes the kernel you are running now. Older kernels keep their old initramfs. If it fails, choose an older kernel in GRUB under *Advanced options*.

---

## Step 6 (optional): Bluetooth on and off with one key

Idea: keep Bluetooth off, and turn it on only when I need it. I use bspwm, so the shortcut is in `sxhkd`.

Needs: `sudo apt install rfkill dunst` (dunst is only for the popup message).

Script `~/.local/bin/bt-toggle`:

```
#!/bin/bash
if systemctl is-active --quiet bluetooth; then
    pkill -f blueman-applet
    pkill -f blueman-tray
    sudo /usr/bin/systemctl stop bluetooth
    notify-send "Bluetooth" "OFF" 2>/dev/null
else
    sudo /usr/bin/systemctl start bluetooth
    sudo /usr/sbin/rfkill unblock bluetooth
    (pgrep -f blueman-applet >/dev/null || blueman-applet >/dev/null 2>&1 &)
    notify-send "Bluetooth" "ON" 2>/dev/null
fi
```

Make it run:

```
chmod +x ~/.local/bin/bt-toggle
```

To avoid a password every time, I allow only these 3 exact commands. First check the paths with `command -v systemctl rfkill`. Then:

```
sudo visudo -f /etc/sudoers.d/bt-toggle
```

```
YOUR_USER ALL=(root) NOPASSWD: /usr/bin/systemctl start bluetooth, /usr/bin/systemctl stop bluetooth, /usr/sbin/rfkill unblock bluetooth
```

Change `YOUR_USER` to your username. Be careful with `sudoers`: only give the exact commands you need.

Key in `~/.config/sxhkd/sxhkdrc`:

```
super + alt + b
    ~/.local/bin/bt-toggle
```

Reload: `pkill -USR1 -x sxhkd`

`pkexec` did not work for me here, because bspwm has no graphical password window. That is why I used `sudoers`.

---

## Not finished yet

- **Waydroid** is not tested after I turned off `lxc` and `lxc-net`. If it has no internet or gives an error, run `sudo systemctl enable lxc-net`.
- **Bluetooth** is off at boot (`systemctl is-enabled bluetooth` says `disabled`). It was only active in one of my checks because I turned it on by hand to try my headset.

---

## Summary

- Turning off services is good for tidiness, but it gave a small gain on my HDD laptop.
- Use `critical-chain`, not only `blame`, to see what really slows the boot.
- Closing big apps gives more RAM than turning off small services.
- An SSD is the real fix for a slow boot.
