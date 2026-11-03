# MTP Setup and Troubleshooting on Debian XFCE (Dell Inspiron 3138)

## 1. Environment

- **OS:** Debian GNU/Linux 13 (Trixie)
- **Desktop Environment:** XFCE
- **File Manager:** Thunar
- **Device:** TECNO POVA Neo 3
- **USB ID:** `0e8d:2008`
- **USB Manufacturer:** MediaTek
- **MTP Backend:** GVFS / libmtp

## 2. Install MTP Packages

Install the required packages for MTP support:

```bash
sudo apt update

sudo apt install -y \
    gvfs \
    gvfs-backends \
    gvfs-fuse \
    mtp-tools \
    libmtp-runtime
```

These packages provide:

- GVFS integration for removable devices.
- MTP support for Android devices.
- MTP diagnostic tools.
- Runtime support for libmtp.

## 3. Detect the Android Device

Connect the phone using a USB cable.

Check whether Debian detects the USB device:

```bash
lsusb
```

The phone appeared as:

```text
Bus XXX Device XXX: ID 0e8d:2008 MediaTek Inc. Cyrus Technology CS 24
```

Run the MTP detection tool:

```bash
mtp-detect
```

The device was successfully identified:

```text
Manufacturer: TECNO
Model: TECNO LH6n
```

This confirmed that libmtp could recognize the phone.

## 4. Check GVFS MTP Integration

Check the running GVFS processes:

```bash
ps aux | grep -E 'gvfs.*mtp'
```

The system had the following MTP volume monitor:

```text
/usr/libexec/gvfs-mtp-volume-monitor
```

Check the available GVFS mounts:

```bash
gio mount -li
```

The phone appeared as:

```text
TECNO POVA Neo 3
```

Its activation URI was:

```text
mtp://TECNO_TECNO_POVA_Neo_3_100692537L000664/
```

The volume was detected and marked as mountable.

## 5. Initial Mount Attempt

Try mounting the phone manually:

```bash
gio mount "mtp://TECNO_TECNO_POVA_Neo_3_100692537L000664/"
```

Check the GVFS mount directory:

```bash
ls -lah /run/user/1000/gvfs/
```

The phone mount appeared under:

```text
/run/user/1000/gvfs/mtp:host=TECNO_TECNO_POVA_Neo_3_100692537L000664
```

However, the mounted directory initially appeared empty.

Thunar also displayed the phone, but opening it did not work correctly.

## 6. MTP Diagnostic Checks

Check the mount information:

```bash
gio info \
"mtp://TECNO_TECNO_POVA_Neo_3_100692537L000664/"
```

The mount was recognized as a directory using the `mtpfs` filesystem type.

Check the user session logs:

```bash
journalctl --user -b | grep -iE 'mtp|gvfs|storage'
```

One relevant message was:

```text
Error 1: Get Storage information failed.
```

This indicated a problem while retrieving storage information from the phone.

### Important: Avoid Running MTP Tools in Parallel

During troubleshooting, `mtp-folders` and `mtp-files` were run in separate terminals.

This was not a good approach because multiple MTP clients may compete for access to the same USB interface.

One diagnostic message was:

```text
libusb_claim_interface() reports device is busy,
likely in use by GVFS
```

For future diagnostics, avoid running multiple MTP clients at the same time.

If GVFS is using the device, stop its access before running standalone MTP diagnostic tools.

## 7. Stop GVFS MTP Processes for Isolation

The GVFS mount was unmounted:

```bash
gio mount -u \
"mtp://TECNO_TECNO_POVA_Neo_3_100692537L000664/" \
2>/dev/null
```

Stop the MTP volume monitor:

```bash
systemctl --user stop gvfs-mtp-volume-monitor.service
```

Stop the GVFS MTP daemon if it is running:

```bash
pkill -f '[g]vfsd-mtp'
```

Check whether the processes are still running:

```bash
pgrep -af 'gvfs.*mtp'
```

At this stage, the MTP-related processes were no longer running.

These actions were temporary. No permanent systemd configuration or service changes were made.

## 8. Important Action: Logout and Login Again

After the previous troubleshooting steps, the device still needed further verification.

Instead of continuing to modify system configuration, the next action was to restart the XFCE user session.

Run:

```bash
xfce4-session-logout --logout
```

This logs out the current XFCE session.

Then:

1. Wait until the login screen appears.
2. Log in again with the same user account.
3. Connect the phone if it is not already connected.
4. Wait for GVFS to detect the device.
5. Open Thunar and check the sidebar.

### Result After Relog

After logging in again, the phone appeared correctly in the Thunar sidebar.

The device could now be opened and accessed.

Check the GVFS services:

```bash
systemctl --user is-active gvfs-daemon.service

systemctl --user is-active gvfs-mtp-volume-monitor.service
```

Both services reported:

```text
active
```

Check the available mounts again:

```bash
gio mount -li
```

The phone appeared as:

```text
TECNO POVA Neo 3
```

Its mount information was available again.

### Why Was Relog Important?

The exact cause of the initial problem was not confirmed.

However, logging out and logging in again restarted the user session and allowed the GVFS and Thunar integration to initialize again.

This likely cleared a stale session state or mount state.

The successful result does not prove that relog alone fixed the original problem. It was performed after several previous troubleshooting actions.

Still, relog was the final important action before the phone became accessible.

## 9. Thunar Volume Management Setting

During the troubleshooting process, the following XFCE setting was checked:

```bash
xfconf-query -c thunar -p /misc-volume-management
```

Initially, the property did not exist in the user's XFConf configuration.

It was then created and set to `true`:

```bash
xfconf-query -c thunar \
    -p /misc-volume-management \
    -n -t bool -s true
```

The final value was verified:

```bash
xfconf-query -c thunar -p /misc-volume-management
```

Output:

```text
true
```

The setting was kept enabled.

It relates to Thunar's volume-management integration. Keeping it enabled is the selected configuration for this system.

The exact default value was not independently confirmed. An absent XFConf property does not necessarily mean that the default value is `false`.

## 10. Final Configuration and Verification

Installed MTP packages:

```text
gvfs
gvfs-backends
gvfs-fuse
mtp-tools
libmtp-runtime
```

Final checks:

```bash
systemctl --user is-active gvfs-daemon.service

systemctl --user is-active gvfs-mtp-volume-monitor.service

xfconf-query -c thunar -p /misc-volume-management

gio mount -li
```

Expected results:

- GVFS daemon: `active`
- GVFS MTP volume monitor: `active`
- Thunar volume management: `true`
- TECNO POVA Neo 3: detected and accessible

## 11. Changes and Rollback Information

| Item | Final decision |
|---|---|
| MTP packages | Keep installed |
| GVFS services | Allow normal session management |
| Thunar volume management | Keep `true` |
| Manual GVFS mount | No permanent change |
| Temporary process termination | No rollback required |
| XFCE session relog | Completed |
| `/etc/fstab` | Not modified |
| Udev rules | Not modified |
| Kernel configuration | Not modified |
| Persistent systemd overrides | Not created |

If the Thunar property needs to be removed and restored to its previous unset state:

```bash
xfconf-query -c thunar \
    -p /misc-volume-management \
    -r
```

This is not necessary for the current working configuration.

## 12. Conclusion

MTP support was installed and the TECNO POVA Neo 3 was successfully detected by Debian.

The initial problem was not a USB detection failure. The device was visible to the system and recognized by libmtp, but GVFS and Thunar could not access its storage correctly.

Several diagnostic steps were performed, including checking GVFS, inspecting mount information, and isolating the MTP processes.

The most important final action was logging out of the XFCE session and logging in again.

After relog, the phone appeared correctly in Thunar and became accessible.

The final decision is to keep `/misc-volume-management` set to `true` and leave the working GVFS configuration unchanged.

**Status: MTP working on Dell Inspiron 3138 — Debian 13 XFCE.**
