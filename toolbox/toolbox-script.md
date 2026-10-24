# Manual Toolbox Script — Documentation

## The goal
A double-clickable menu for on-demand tasks that shouldn't run automatically at boot/login: starting the game dev server, running a full system update, and checking the latest SMART status. Unlike the other three projects, this one is meant to be triggered manually, not automated.

## Final script

**`~/toolbox.sh`**:
```bash
#!/bin/bash
# ~/toolbox.sh — double-click launcher

PS3="Pick an action: "
options=("Start game server" "System update" "SMART status now" "Quit")
select opt in "${options[@]}"; do
  case $opt in
    "Start game server")
      python3 -m http.server 8000 --directory ~/game
      break
      ;;
    "System update")
      sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y
      break
      ;;
    "SMART status now")
      cat /var/log/smart-check.log
      break
      ;;
    "Quit") break ;;
  esac
done

read -p "Press Enter to close..."
```
```bash
chmod +x ~/toolbox.sh
```

Notes on choices made:
- `python3 -m http.server 8000 --directory ~/game` replaces the original `cd ~/game && python3 -m http.server 8000` — same result, one line, no `cd` needed.
- "SMART status now" reads the existing `/var/log/smart-check.log` (written daily by the [[smart-hdd-check]] boot service) instead of running `smartctl` live again — avoids duplicating a check that already runs automatically once a day.
- `read -p "Press Enter to close..."` at the end prevents the terminal window from instantly closing after a task finishes when launched by double-click.
- `apt full-upgrade` and `apt autoremove` require the interactive sudo password — intentionally left as-is, since this script is meant to be run manually and watched, unlike the other three automated fixes.

## Making it double-clickable

### Attempt 1 — double-click the .sh file directly
Opened in Mousepad (text editor) instead of running. This is expected default behavior for `.sh` files in most file managers — they're treated as text, not executables, for safety.

### Fix — .desktop launcher

**`~/.local/share/applications/toolbox.desktop`**:
```ini
[Desktop Entry]
Name=My Toolbox
Comment=Game server / system update / SMART status
Exec=x-terminal-emulator -e /home/hagure/toolbox.sh
Type=Application
Terminal=false
Icon=utilities-terminal
```
Confirmed `x-terminal-emulator` resolves correctly on this system (`/usr/bin/x-terminal-emulator`).

Launches correctly from the **applications menu** immediately.

### Getting a Desktop icon (not just applications menu)
```bash
cp ~/.local/share/applications/toolbox.desktop ~/Desktop/
chmod +x ~/Desktop/toolbox.desktop
```
Desktop icons on this environment required one manual right-click step the first time — **"Allow Launching"** (or similarly worded trust option) — before double-click would run it instead of showing it as inert/text. This is a one-time trust step per icon, not something scriptable via terminal.

## Result
Both launch paths confirmed working: from the applications menu (search "My Toolbox") and as a double-click icon on the Desktop, after the one-time "Allow Launching" trust step.
