# Bluetooth OBEX Fix — Documentation

## The problem
After boot, sending files from phone to laptop via Bluetooth OBEX would fail immediately. Bluetooth itself connected fine (pairing, audio, etc. all worked) — only OBEX file transfer was broken.

## What worked before this project (baseline)
Manually running this in a terminal — including entering the sudo password when prompted — fixed it every time, right before sending a file:
```bash
sh -c "sleep 5 && systemctl restart bluetooth"
```
This was already set up in **Session and Startup → Application Autostart**, triggered `on login`. It worked, but asked for a password every login.

## Goal
Keep the fix, remove the password prompt.

## Attempts that did NOT work

### Attempt 1 — systemd boot-time service (root, no password)
Created `/etc/systemd/system/bt-restart-fix.service` to restart `bluetooth.service` as root, ~5s after boot, via `systemctl enable`.
- Ran successfully every time (`Active: inactive (dead)` = success for `oneshot`).
- Bluetooth itself came up fine.
- **OBEX still failed.**

### Attempt 2 — increase delay to 30s, then 45s
Theory: adapter/USB not fully initialized yet at boot.
- Bumped `ExecStartPre=/bin/sleep 5` → `30` → tested again.
- Confirmed via `bluetoothctl show` that `Powered: yes` well before the restart fired.
- **Still failed.** Ruled out timing/delay as the cause.

### Attempt 3 — also restart `obex.service` (user-level) after bluetooth
Theory: `obexd` needed to be restarted too, and in the right order relative to bluetooth's restart.
- Added `~/obex-fix.sh` (`sleep N && systemctl --user restart obex.service`) to Session & Startup.
- Tried firing it before *and* after the systemd bluetooth restart (10s, then 45s delay to land after the 30s bluetooth restart).
- Checked `journalctl --user -u obex.service` to confirm actual fire order.
- **Still failed**, regardless of order. Ruled out restart ordering as the cause.

## Root cause (found via elimination)
The user pointed out that whenever the **password-prompted** manual command ran, it worked — every time. That was the real clue.

The systemd `bt-restart-fix.service` ran as **root**, outside the user's login session, with no D-Bus/session context. The manual command (typed by the user, escalated via polkit password prompt) ran **inside the user's own session**. Even though both ultimately call the same `systemctl restart bluetooth`, OBEX only came back working when the restart happened through the user's authenticated session — not from a bare root systemd unit.

Conclusion: **it was never about timing or restart order** — it was about *which context* performed the restart.

## Fix that worked
Kept the original Session & Startup command exactly as-is (`sh -c "sleep 5 && systemctl restart bluetooth"`, on login), and removed the password prompt using a **scoped polkit rule** — instead of a systemd service.

`/etc/polkit-1/rules.d/49-bluetooth-restart.rules`:
```js
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.systemd1.manage-units" &&
        action.lookup("unit") == "bluetooth.service" &&
        subject.isInGroup("hagure")) {
        return polkit.Result.YES;
    }
});
```
```bash
sudo systemctl restart polkit
```

This lets the restart still run inside the user's own session (same mechanism that always worked) but skips the password prompt — scoped only to `bluetooth.service`, and only for the `hagure` user/group. Other user accounts on the same machine still get the normal password prompt for this action.

## Cleanup performed
- Removed `/etc/systemd/system/bt-restart-fix.service` (didn't fix the issue, no longer needed)
- Removed `~/obex-fix.sh` and its Session & Startup entry (chasing wrong theory, not needed)
- Kept: original Session & Startup Bluetooth restart entry + new polkit rule

## Result
Confirmed working after reboot — Bluetooth restarts silently on login, no password prompt, and OBEX file transfer from phone works immediately without any manual intervention.
