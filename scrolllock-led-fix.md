# Scroll Lock LED Fix — Documentation

## The problem
Wanted the scroll-lock LED to turn on automatically after login, without typing a password, and without hardcoding a specific input device number — since plugging/unplugging an external keyboard changes which `inputN::scrolllock` node is assigned each boot.

## Baseline manual command (worked, but with sudo password)
```bash
ls /sys/class/leds/*scrolllock
echo 1 | sudo tee /sys/class/leds/input::scrolllock/brightness
```

## Goal
Automate this at login, avoid hardcoding a device number, no password prompt.

## Attempts that did NOT work

### Attempt 1 — systemd boot-time service (root, no password)
Created `/etc/systemd/system/scrolllock-led.service`, looping over `/sys/class/leds/*scrolllock*` to catch any device number, enabled at boot.
- Worked — LED lit up at the login greeter (xmessage/greeter screen).
- **Turned off again once the desktop session finished loading.** The desktop environment resets keyboard LED state on session start, overriding the boot-time write.
- Disabled and later deleted this service once the real fix was found — boot-time was the wrong trigger point.

### Attempt 2 — same script via Session & Startup (`sudo tee`, on login)
Moved the fix to fire after desktop load (matching the working bluetooth approach) instead of at boot.
- **`sudo` produced no prompt and no effect at all** — Session & Startup autostart entries have no TTY/password-agent attached, so `sudo` fails silently in the background with nothing shown to the user.

### Attempt 3 — udev rule with `MODE="0666"` / `RUN+= chmod`
Theory: grant the user's own account direct write permission to the LED sysfs file, avoiding sudo entirely.
- Confirmed via `udevadm test` that the rule file was read and the device correctly matched (`SUBSYSTEM=="leds"`, `KERNEL=="*scrolllock*"`).
- Checked `getfacl` — no ACL granted (this device is tagged `seat` but LED class devices aren't covered by systemd-logind's dynamic uaccess ACLs).
- Permissions on `brightness` never changed, write still failed as plain user even after a full reboot.
- **Root cause: `MODE=` in udev rules only applies to `/dev/*` device nodes udev creates itself — it has no authority over sysfs attribute files like `brightness`, which are permissioned by the kernel driver directly.** Confirmed this is a hard limitation, not a syntax mistake — deleted the udev rule.

### Attempt 4 — sudoers rule with inline shell command
```
hagure ALL=(root) NOPASSWD: /bin/sh -c 'for led in /sys/class/leds/*scrolllock*; do echo 1 > "$led/brightness"; done'
```
- Worked once, immediately after setup.
- **Failed again after reboot** — sudoers does literal argument-by-argument matching against the exact command string. Inline shell one-liners with quoting are fragile; a re-parse/normalization mismatch (spacing, quote style) between what's stored and what's actually invoked causes sudo to silently reject the match and fall back to asking for a password.

## Fix that worked
Stopped using an inline shell command in sudoers — pointed sudoers at a **fixed script file path** instead, which sudoers matches reliably.

**`/usr/local/bin/scrolllock-on.sh`** (root-owned):
```bash
#!/bin/bash
for led in /sys/class/leds/*scrolllock*; do
  echo 1 > "$led/brightness"
done
```
```bash
sudo chmod +x /usr/local/bin/scrolllock-on.sh
```

**`/etc/sudoers.d/scrolllock-led`** (via `visudo`, never edited directly):
```
hagure ALL=(root) NOPASSWD: /usr/local/bin/scrolllock-on.sh
```

**`~/scrolllock-fix.sh`** (Session & Startup entry, `on login`):
```bash
#!/bin/bash
sleep 5
sudo /usr/local/bin/scrolllock-on.sh
```

## Key lesson
File-path-based sudoers rules are reliable; inline shell command strings in sudoers are fragile and can silently stop matching after things like a reboot re-parsing the file. Prefer pointing sudoers at a real script file for anything beyond a trivial one-liner.

## Cleanup performed
- Removed `/etc/systemd/system/scrolllock-led.service` (wrong trigger point — fires before desktop resets LED state)
- Removed `/etc/udev/rules.d/99-scrolllock-led.rules` (udev has no control over sysfs attribute file permissions)
- Kept: `/usr/local/bin/scrolllock-on.sh`, `/etc/sudoers.d/scrolllock-led`, `~/scrolllock-fix.sh` + Session & Startup entry

## Result
Confirmed working after reboot — scroll-lock LED turns on automatically ~5 seconds after desktop login, no password prompt, holds regardless of which `inputN::scrolllock` number gets assigned to internal/external keyboards.
