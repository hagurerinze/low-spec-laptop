# Daily SMART/HDD Health Check — Documentation

## The goal
Run `smartctl` automatically once after boot (10-year-old laptop HDD, wanted daily monitoring), with no terminal typing — just a notification showing key health numbers, including day-over-day trend on Reallocated_Sector_Ct specifically (since that's the earliest real warning sign on an aging drive, well before SMART's own PASSED/FAILED verdict changes).

## Baseline manual command
```bash
sudo smartctl -a /dev/sda
```

## Design (informed by lessons from bluetooth/LED projects)
Same two-part pattern as the LED fix:
- **Root-level systemd service** does the actual `smartctl` read + writes results to log files (no password needed, root already).
- **User-session Session & Startup script** reads those logs after login and pops a desktop notification (no sudo needed, since it's just reading files).

## Attempts and issues along the way

### Issue 1 — used `-H` (health-only) instead of `-a` (full attributes)
Initial version only captured the one-line PASSED/FAILED summary. Not enough for a 10-year-old drive — attributes like Reallocated_Sector_Ct, Current_Pending_Sector, and Offline_Uncorrectable can show real degradation long before the overall health verdict flips. Switched to `smartctl -a` to capture the full attribute table.

### Issue 2 — systemd service showed "failed" even though it worked
`smartctl` returns a non-zero exit code whenever any attribute is below its "ideal" threshold or similar conditions are detected — this is by design (meant for scripts checking the exit code), not an actual error. Since systemd treats non-zero exit as failure, the service showed "failed" even though the log was written correctly.
- **Fix:** appended `; exit 0` to the command so systemd always sees success once smartctl has run and the output is captured, regardless of smartctl's own exit code.

### Issue 3 — inline shell command in `ExecStart=` got messy for the history-tracking version
Once day-over-day trend tracking was added (needing to `awk` out one field and append a dated line), the `ExecStart=` one-liner required heavy escaping (`%%`, nested quotes) and became fragile/hard to edit.
- **Fix:** moved the logic into a dedicated script file, `/usr/local/bin/smart-check.sh`, and pointed `ExecStart=` at that file instead — same lesson learned from the LED sudoers issue: file paths are far more reliable and maintainable than inline shell one-liners embedded in config.

### Issue 4 — notify script failed with "Permission denied"
`~/smart-notify.sh` wasn't executable when first run — `chmod +x` either wasn't run yet or the file had been re-saved after chmod (some editors reset permissions on save).
- **Fix:** re-ran `chmod +x ~/smart-notify.sh`, confirmed with `ls -l` that the `x` flags were present, then it ran fine.

## Final working setup

**`/usr/local/bin/smart-check.sh`** (root-owned, called by systemd):
```bash
#!/bin/bash
LOGFILE="/var/log/smart-check.log"
HISTORY="/var/log/smart-history.log"

smartctl -a /dev/sda > "$LOGFILE" 2>&1

REALLOC=$(smartctl -A /dev/sda | awk '/Reallocated_Sector_Ct/{print $10}')
echo "$(date +%Y-%m-%d) $REALLOC" >> "$HISTORY"
```
```bash
sudo chmod +x /usr/local/bin/smart-check.sh
```

**`/etc/systemd/system/smart-check.service`** (boot-time, root):
```ini
[Unit]
Description=SMART check on boot
After=local-fs.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/smart-check.sh

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl enable smart-check.service
```

**`/var/log/smart-history.log`** — plain text, one line per day (`YYYY-MM-DD reallocated_count`), permission `666` so the user-session script can read it without sudo.

**`~/smart-notify.sh`** (Session & Startup, on login):
```bash
#!/bin/bash
sleep 10

LOGFILE="/var/log/smart-check.log"
HISTORY="/var/log/smart-history.log"

HEALTH=$(grep "SMART overall-health" "$LOGFILE" | awk -F': ' '{print $2}')
REALLOC=$(awk '/Reallocated_Sector_Ct/{print $10}' "$LOGFILE")
PENDING=$(awk '/Current_Pending_Sector/{print $10}' "$LOGFILE")
OFFLINE_UNC=$(awk '/Offline_Uncorrectable/{print $10}' "$LOGFILE")
POWER_HOURS=$(awk '/Power_On_Hours/{print $10}' "$LOGFILE" | grep -oE '^[0-9]+')

TODAY=$(tail -n 1 "$HISTORY" | awk '{print $2}')
YESTERDAY=$(tail -n 2 "$HISTORY" | head -n 1 | awk '{print $2}')

TREND="stable"
if [ -n "$YESTERDAY" ] && [ "$TODAY" -gt "$YESTERDAY" ]; then
  TREND="⚠️ increased (was $YESTERDAY)"
fi

if [ "$HEALTH" = "PASSED" ] && [ "$PENDING" = "0" ] && [ "$OFFLINE_UNC" = "0" ]; then
  URGENCY="normal"
  TITLE="HDD Health: OK"
else
  URGENCY="critical"
  TITLE="HDD Health: Check Needed ⚠️"
fi

notify-send -u "$URGENCY" "$TITLE" \
"Overall: $HEALTH
Reallocated Sectors: $REALLOC ($TREND)
Pending Sectors: $PENDING
Offline Uncorrectable: $OFFLINE_UNC
Power-On Hours: $POWER_HOURS"
```
Added to Session & Startup, on login, no sudo required (read-only on files the user already owns/has permission for).

## Baseline reading recorded during setup (2026-08-17)
For reference — drive is a Seagate ST500LT012-1DG142 (500GB, 5400rpm, 2.5"):
- Overall health: PASSED
- Reallocated_Sector_Ct: 1160 (nonzero — real wear, worth the daily trend tracking)
- Current_Pending_Sector: 0
- Offline_Uncorrectable: 0
- Power_On_Hours: ~27306

Reallocated sector count is elevated but not at threshold — this is exactly why day-over-day trend tracking matters more than the single PASSED/FAILED verdict for a drive this age.

## Result
Confirmed working after reboot — SMART check runs silently at boot, history logged daily, and a desktop notification pops ~10 seconds after login showing overall health, key attribute values, and whether Reallocated_Sector_Ct increased since the previous day.
