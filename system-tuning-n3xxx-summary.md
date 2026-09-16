# System Tuning Project — 2GB RAM Celeron N3xxx (Debian XFCE)

**Status:** Closed (temporarily, reopened once for a follow-up) — 2026-09-14

**Hardware:** HDD storage, 2GB RAM, Celeron N3xxx, Intel HD Graphics 500 (Apollo Lake), existing swap partition (`/dev/sda2`, 4G)

---

## Display driver
- Found a `bochs` DRM kernel module loaded alongside `i915`, despite no VM present (`systemd-detect-virt` → `none`).
- Confirmed via `modinfo` it's a stock in-tree kernel module matching QEMU/Bochs virtual VGA IDs — loaded by Debian's broad initramfs module set (`MODULES=most`), not anything installed manually.
- Blacklisted it: `/etc/modprobe.d/blacklist-bochs.conf` → `blacklist bochs`, then `sudo update-initramfs -u`.
- Verified after reboot: `lsmod | grep bochs` returns nothing, `i915` still loaded and driving the display, VA-API (`vainfo`) confirmed working via the Intel iHD driver.

## Swap strategy: zram vs zswap vs swap partition vs swapfile
- **Decision: zram + swap partition** (kept the existing setup rather than adding zswap).
- Reasoning: zswap sits *in front of* whichever swap device is highest priority. Since zram is priority 100 here, zswap would compress pages, then those pages would eventually get decompressed and handed to zram — which recompresses them. Redundant CPU work for zero RAM benefit on a weak CPU.
- zswap's real value is protecting a wear-sensitive, fast SSD from writes. This laptop has an HDD (not wear-sensitive, already slow) — so zswap's upside doesn't apply here.
- Partition beats swapfile on HDD specifically because of (1) guaranteed contiguous sectors — no fragmentation-driven extra seeks, and (2) no filesystem-layer indirection on every I/O. Both effects are minor on SSD but real on HDD.
- **General device-type tier list (best → worst for this HDD/weak-CPU profile):** zram → zswap → swap partition → swapfile.

## zram resize
- Changed `/etc/default/zramswap`:
  - `ALGO`: `lz4` → `zstd` (better compression ratio, more CPU cost — worthwhile since RAM was the scarcer resource)
  - `PERCENT`: `50` → `75` (raises the ceiling from ~901M to ~1.3G; actual RAM used is only the *compressed* size of what's stored, not this ceiling)
  - `PRIORITY` (100) and `SIZE` (unused, overridden by `PERCENT`) left unchanged
- Applied via `sudo systemctl restart zramswap` (no reboot needed).
- Noted for future reference: restarting zram while RAM is already tight forces some pages temporarily onto disk during the transition (observed ~270M land on `/dev/sda2` right after the restart). Best done right after a fresh reboot next time, when swap usage is near zero.
- Verified active algorithm: `cat /sys/block/zram0/comp_algorithm` → `[zstd]`.

## Swappiness / cache pressure
- Added `/etc/sysctl.d/99-zram-tuning.conf`:
  ```
  vm.swappiness=100
  vm.vfs_cache_pressure=50
  ```
- Applied via `sudo sysctl --system`.
- Verified effect: free RAM increased (188Mi → 284Mi) and disk swap usage actively decreased as the kernel moved pages from disk back into the now-larger, better-compressing zram device.

## CPU governor
- Checked with `cpupower frequency-info` (required installing `linux-cpupower`).
- Already running **schedutil** — the only alternative available on this hardware (`intel_cpufreq` driver) is `performance`. No change needed.
- Turbo Boost is active; left enabled since the laptop has adequate cooling (dedicated fan + laptop stand for airflow).

## GPU / display
- i915 confirmed as the sole active display driver after the bochs cleanup.
- VA-API confirmed working (H.264, HEVC, VP8/VP9 profiles all present via the Intel iHD driver).
- **Checked Framebuffer Compression (FBC) status *before* touching GRUB:**
  ```
  sudo cat /sys/kernel/debug/dri/0/i915_fbc_status
  → FBC enabled
  → Compressing: yes
  ```
- **Highlight: did NOT add `i915.enable_fbc=1` to `GRUB_CMDLINE_LINUX_DEFAULT`.** FBC was already active by default on this hardware without the parameter — adding it would have been purely redundant, so it was deliberately left out. `GRUB_CMDLINE_LINUX_DEFAULT` remains `""` (plain text boot, no splash), matching the boot-theming project's existing setup.

---

## Final state summary

| Component | Setting | Status |
|---|---|---|
| zram | 1.3G ceiling, zstd, priority 100 | ✅ Active, absorbing more over time |
| Disk swap | `/dev/sda2`, priority -2 | ✅ Fallback only, usage trending down |
| Swappiness | 100 | ✅ Applied |
| VFS cache pressure | 50 | ✅ Applied |
| CPU governor | schedutil | ✅ Already optimal, unchanged |
| Turbo Boost | Active | ✅ Left on (adequate cooling) |
| Display driver | i915 only (bochs blacklisted) | ✅ Verified after reboot |
| FBC | Enabled by default | ✅ Confirmed — **no GRUB change made** |

## Follow-up: real RAM hog found (mariadb)
- Noticed a gap between login-baseline RAM under lz4 (~700MB) vs zstd (~1.1GB) that didn't make sense from the algorithm alone — compression can't affect RAM before anything's actually been swapped.
- Investigated via `ps aux --sort=-rss | head -15` taken right after login, before opening Firefox.
- Ruled out bspwm leftovers, autostart `.desktop` entries, and user-level `systemd --user` services — all accounted for and expected (conky, fcitx5, live wallpaper scripts, bluetooth/scroll-lock/SMART scripts).
- Found the real cost: **`mariadbd`** (MariaDB/MySQL), a system-level service (not user-session), running idle and holding **~155MB RSS** at every boot regardless of whether it was actually being used.
- This is genuine active memory — not something zram/swappiness tuning could ever touch, since it was never swapped out.
- Disabled its autostart so it only runs when manually started (`sudo systemctl start mariadb` when actually needed for a project).
- **Result: baseline RAM usage dropped to ~900MB** — the single biggest win of the whole project, bigger than the zram/swappiness changes combined.

## Possible future follow-ups (not started)
- Revisit zram sizing if usage patterns change (e.g., heavier multitasking).
- Re-check swap/zram balance after installing anything memory-heavy.
- If a project needs mariadb running persistently again, re-enable its autostart (`sudo systemctl enable mariadb`) rather than starting it manually each time.
