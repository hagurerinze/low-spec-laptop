# Live Wallpaper System — Documentation

Setup: XFCE, xwinwrap + mpv, video wallpaper with auto-pause on
maximize/fullscreen, plus two independent manual toggles.

## Components

| Script | Path | Role |
|---|---|---|
| `livewallpaper-start` | `~/.local/bin/livewallpaper-start` | Launches xwinwrap + mpv. Runs on login via autostart. |
| `livewallpaper-controller` | `~/.local/bin/livewallpaper-controller` | Background loop. Auto-pauses on maximize/fullscreen, auto-resumes otherwise. Runs on login via autostart. |
| `livewallpaper-pause-toggle` | `~/.local/bin/livewallpaper-pause-toggle` | Manual pause/resume. Does **not** kill mpv — RAM usage stays the same. |
| `livewallpaper-toggle` | `~/.local/bin/livewallpaper-toggle` | Full on/off. Kills xwinwrap+mpv+controller entirely (RAM drops to ~0) or relaunches the whole chain. When off, XFCE's own static wallpaper shows through. |

## Runtime files (all in `/tmp/mpv-hagure/`, wiped on reboot)

- `wallpaper.sock` — mpv's IPC socket, created by mpv itself on start. Scripts
  pipe JSON commands into it via `socat` to control the running mpv instance.
- `manual_pause` — shared flag (`0`/`1`) between `pause-toggle` and
  `controller`. `pause-toggle` writes it; `controller`'s loop reads it every
  second and forces pause if it's `1`, overriding the maximize/fullscreen logic.
- `toggle.lock` — anti-spam timestamp for `pause-toggle`; ignores presses
  less than 1 second apart to avoid racing the controller loop.

Both runtime files live under `/tmp/` (not `~/.cache/`) on purpose: mpv's
socket is always recreated fresh each session anyway, so keeping
`manual_pause` alongside it means both share the same "resets every reboot"
lifecycle. `livewallpaper-start` always launches mpv unpaused with no logic
to read a leftover pause state — so a flag that persisted across reboots
(e.g. in `~/.cache/`) could get stuck at `1` and cause the controller to
immediately pause a freshly-started, unpaused mpv.

## Control flow

1. Login → `livewallpaper-start` launches xwinwrap+mpv → `wallpaper.sock` created.
2. Login → `livewallpaper-controller` starts its loop, watching the active
   window and `manual_pause`.
3. Priority inside the controller loop, checked in order:
   - `manual_pause == 1` → always pause, skip everything else
   - active window is mpv itself → resume
   - active window is fullscreen/maximized → pause
   - otherwise → resume
4. `livewallpaper-pause-toggle` (bound to a shortcut) flips `manual_pause`
   and also pokes `wallpaper.sock` directly for an instant pause/resume,
   instead of waiting up to 1s for the controller loop to notice.
5. `livewallpaper-toggle` (bound to a different shortcut) checks if
   `xwinwrap` is running:
   - if running → kills xwinwrap, mpv, and the controller; clears
     `manual_pause`; static XFCE wallpaper shows through; RAM freed.
   - if not running → resets `manual_pause` to `0`, relaunches
     `livewallpaper-start`, then starts a fresh `livewallpaper-controller`
     after a short delay (since `livewallpaper-start` only kills an
     existing controller, it doesn't start a new one).

## Known fix applied

- **Center conky disappearing on click-and-hold drag**: was missing
  `own_window_hints = 'undecorated,below,sticky,skip_taskbar,skip_pager'`
  in its `conky.config`. Without it, xfwm4 treats it as a normal, raisable/
  movable window, so a click-drag gesture grabs/lowers it. Fixed by adding
  the same `own_window_hints` line used in the working top-right conky.

## Alternatives considered but not used (pause-toggle)

Two other versions of `livewallpaper-pause-toggle` were compared during
development. Neither was adopted, for the reasons below.

### Version A — anti-spam only, no notify, no instant socket push

```bash
#!/bin/bash
SOCK="/tmp/mpv-hagure/wallpaper.sock"
FLAG="/tmp/mpv-hagure/manual_pause"

mkdir -p /tmp/mpv-hagure
[ -f "$FLAG" ] || echo 0 > "$FLAG"

LOCK="/tmp/mpv-hagure/toggle.lock"
NOW=$(date +%s)
if [ -f "$LOCK" ]; then
    LAST=$(cat "$LOCK")
    if [ $((NOW - LAST)) -lt 1 ]; then
        exit 0
    fi
fi
echo "$NOW" > "$LOCK"

CURRENT=$(cat "$FLAG" 2>/dev/null || echo 0)
if [ "$CURRENT" -eq 0 ]; then
    echo 1 > "$FLAG"
else
    echo 0 > "$FLAG"
fi
```

**Not used because:** it had the anti-spam lock (good) but no
`notify-send` feedback and no direct `socat` push to `wallpaper.sock` —
so the user gets no on-screen confirmation, and pause/resume waits up to
1 second for the controller loop to notice the flag instead of applying
instantly. Its useful pieces (the anti-spam lock) were merged into the
final version instead of using this file as-is.

### Version B — flag stored in `~/.cache/` instead of `/tmp/`

```bash
#!/bin/bash
FLAG="$HOME/.cache/mpv-wallpaper.pause"
LOCK="/tmp/mpv-hagure/toggle.lock"
NOW=$(date +%s)

if [ -f "$LOCK" ]; then
    LAST=$(cat "$LOCK")
    if [ $((NOW - LAST)) -lt 1 ]; then
        exit 0
    fi
fi
echo "$NOW" > "$LOCK"

CURRENT=$(cat "$FLAG" 2>/dev/null || echo 0)
if [ "$CURRENT" -eq 0 ]; then
    echo 1 > "$FLAG"
    notify-send -t 1000 "Live Wallpaper" "⏸️ Paused"
else
    echo 0 > "$FLAG"
    notify-send -t 1000 "Live Wallpaper" "▶️ Resumed"
fi
```

**Not used because:** two problems.
1. `FLAG` points at `$HOME/.cache/mpv-wallpaper.pause`, but
   `livewallpaper-controller` reads from `/tmp/mpv-hagure/manual_pause` —
   a different path entirely. As written, this script's toggle would
   never actually reach the controller, so mpv would never pause despite
   the notification saying it did.
2. Even with the path corrected, `~/.cache/` **persists across reboots**
   while `wallpaper.sock` does not (mpv recreates it fresh every start).
   `livewallpaper-start` always launches mpv unpaused with no logic to
   read a leftover pause state, so a flag stuck at `1` from a previous
   session would cause the controller to immediately re-pause a
   freshly-started, unpaused mpv. Keeping the flag in `/tmp/mpv-hagure/`
   alongside the socket means both share the same "resets every reboot"
   lifecycle, avoiding this mismatch.

## Suggested keybinds

- `livewallpaper-pause-toggle` → e.g. `Super+W` (quick freeze/resume, mpv stays loaded)
- `livewallpaper-toggle` → a separate shortcut (full off/on, frees RAM when off)

Bind both under Settings → Keyboard → Application Shortcuts, pointing
directly at each script's full path.
