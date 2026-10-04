Status: draft, not applied

# zram hybrid mode (draft, not applied)

## Idea
- Primary algorithm: lz4 (fast, low CPU for hot pages)
- Idle pages: recompress with zstd (better ratio)
- RAM-only, never touches the HDD

## Check first
- Kernel 6.0+ and `/sys/block/zram0/recomp_algorithm` exists
- zram-tools (/etc/default/zramswap) can't do this, so it needs a script/timer

## Rough steps
1. echo zstd > /sys/block/zram0/recomp_algorithm   (before the device is initialised)
2. Mark idle pages: echo 600 > /sys/block/zram0/idle
3. Recompress: echo "type=idle" > /sys/block/zram0/recompress
4. Run steps 2-3 from a systemd timer (e.g. every 10 min)

## Decisions so far
- zswap: no (would double-compress on top of zram)
- Swap file: no (same speed as the partition, and home is at 94%)
- Current setup stays: zram zstd PERCENT=75 + swap partition as overflow
- If zram keeps hitting its limit: raise to PERCENT=85, don't add layers

## Watch
- `swapon --show`: sda2 usage should stay near zero
