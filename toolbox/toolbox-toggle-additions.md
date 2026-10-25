# Toolbox Script — Toggle Additions (LED + WA Bot)

Third round of updates to `~/toolbox.sh`. Covers two new toggle-style features, plus a standalone keyboard shortcut.

## 1. Scroll Lock LED — toggle script + shortcut

Previously, the only automated LED option was "always force ON" (`scrolllock-on.sh`, used both at login and as toolbox option 7). Since external keyboard caps lock presses can knock the LED back off, a true on/off toggle was added instead of only ever forcing it on.

**`/usr/local/bin/scrolllock-toggle.sh`** (root-owned):
```bash
#!/bin/bash
STATE=0
for led in /sys/class/leds/*scrolllock*; do
  val=$(cat "$led/brightness")
  [ "$val" = "1" ] && STATE=1
done

if [ "$STATE" = "1" ]; then
  for led in /sys/class/leds/*scrolllock*; do
    echo 0 > "$led/brightness"
  done
else
  for led in /sys/class/leds/*scrolllock*; do
    echo 1 > "$led/brightness"
  done
fi
```
```bash
sudo chmod +x /usr/local/bin/scrolllock-toggle.sh
```

**`/etc/sudoers.d/scrolllock-toggle`** (same reliable file-path pattern as the original LED fix, via `visudo`):
```
hagure ALL=(root) NOPASSWD: /usr/local/bin/scrolllock-toggle.sh
```

**Keyboard shortcut** — bound via Xfce's Keyboard → Application Shortcuts panel:
- Command: `sudo /usr/local/bin/scrolllock-toggle.sh`
- Shortcut: `Ctrl+Alt+S`
- "Use startup notification" left unchecked (same reasoning as the toolbox shortcut — terminal-launched scripts don't reliably send the window-registration signal this waits for)

**Added to toolbox** as option 8, "Toggle Scroll Lock LED" — sits alongside the original "Re-enable Scroll Lock LED" (option 7, always-forces-on). Both coexist safely since they're separate scripts with separate sudoers entries acting on the same LED file; nothing conflicts.

**Still a draft idea (not built yet):** a desktop notification showing the LED's new state (on/off) each time the toggle runs.

## 2. Yurishy (WhatsApp bot) — toggle in toolbox

Added as option 11, "Toggle Yurishy (WA Bot)":
```bash
if systemctl --user is-active --quiet wabot; then
  systemctl --user stop wabot
  echo "Yurishy (wabot) stopped."
else
  systemctl --user start wabot
  echo "Yurishy (wabot) started."
fi
```
This assumes the systemd user service unit is named `wabot` (matching the bot's existing manual on/off setup — the bot intentionally has no auto-start at boot, by the user's earlier choice, and already had shell aliases `boton`/`botoff`/`botrestart`/`botstatus`/`botlog` plus a separate one-click toggle icon outside the toolbox). This toolbox option gives a second way to toggle it without needing the aliases or the separate icon.

## Updated toolbox menu (full, current)

**Daily:** Start game server · System update · SMART status now · Read .dat file · Organize .md files · Test internet connection
**Emergency Fixes:** Re-enable Scroll Lock LED · Toggle Scroll Lock LED · Restart Bluetooth · Restart WiFi
**Bot:** Toggle Yurishy (WA Bot)
**System:** Power Options (Suspend/Hibernate/Hybrid Sleep/Reboot/Shutdown/Switch User/Logout) · Open New Terminal · ADB Check · Task Manager (btop/top/GUI) · GParted · Quit

## Result
Confirmed working — LED toggle flips state correctly both via the toolbox and the `Ctrl+Alt+S` shortcut; Yurishy toggle starts/stops the bot from the toolbox menu.
