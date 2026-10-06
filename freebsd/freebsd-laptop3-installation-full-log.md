# FreeBSD 15.1 on Laptop 3 — Installation, Recovery, XFCE Setup, and Drafts

## Status

**Current status:** Paused after successfully entering XFCE.

**Machine:** Laptop 3  
**CPU:** Intel Core i3-3217U, 2 cores / 4 threads, 1.80 GHz  
**RAM:** 2 GB  
**Storage:** WDC WD5000LPVX-22V0TT0, 465.8 GB HDD  
**Existing OS:** Debian GNU/Linux 13 trixie, x86_64  
**FreeBSD:** 15.1-RELEASE  
**Filesystem:** UFS  
**Desktop:** XFCE  
**Display:** Xorg + Intel `i915kms` + LightDM  
**Wi-Fi:** Atheros `ath0` / `wlan0`

This document records the experiment from partition planning through the working XFCE desktop, including deliberate user-directed risky decisions, recovery actions, unfinished drafts, and future investigations.

---

## 1. Initial Machine Context

Laptop 3 was an older low-spec machine with:

- Intel Core i3-3217U
- 2 GB RAM
- 500 GB-class HDD
- Existing Debian GNU/Linux 13 trixie
- EFI boot

The machine was deliberately used as an experimental OS platform rather than as a conservative daily-driver installation.

The broader plan was:

```text
Debian
  ↓
FreeBSD
  ↓
Android-x86 / FydeOS experiments
  ↓
Further BSD / Unix-like exploration
```

---

## 2. Original Debian Partition Layout

The Debian layout before the FreeBSD experiment was approximately:

```text
ada0p1   512M    EFI
ada0p2     4G    Swap
ada0p3    60G    Debian Root
ada0p4   140G    Debian Home
```

Additional free/unallocated space was available after partition adjustment.

The later FreeBSD layout became:

```text
ada0p1   512M    EFI
ada0p2     4G    Debian Swap
ada0p3    60G    Debian Root
ada0p4   140G    Debian Home
ada0p5   136G    FreeBSD UFS Root
ada0p6     4G    FreeBSD Swap
```

Approximately 121 GB remained unallocated.

The existing Debian EFI partition was shared instead of creating a completely separate EFI partition.

---

## 3. Partition Adjustment

The user considered multiple shrink-and-rearrange layouts before committing to the FreeBSD experiment, including:

```text
140G → 260G
140G → 140G + 120G
120G → 20G + 140G + 120G
120G → 160G + 120G
120G → 120G + 40G + 120G
```

The objective was to create enough contiguous space for another operating system while retaining Debian.

### User-directed high-risk decision

The user personally chose to modify the existing disk layout and continue with the FreeBSD installation despite the risk of:

- partition damage
- filesystem damage
- bootloader problems
- data loss

This was an intentional experimental decision, not a routine recommendation.

---

## 4. FreeBSD Installation

### Version

```text
FreeBSD 15.1-RELEASE
```

The installation was performed using Ventoy.

Target partitions:

```text
/dev/ada0p5 → FreeBSD UFS root
/dev/ada0p6 → FreeBSD swap
```

### Filesystem choice

UFS was selected instead of ZFS because the machine has:

```text
2 GB RAM
+
HDD
+
lightweight experimental desktop
```

The goal was to keep the base storage setup relatively simple and appropriate for the hardware.

---

## 5. Installer Filesystem Problem

During installation, FreeBSD encountered mounting/filesystem problems involving the target partition.

The user chose to continue troubleshooting rather than abandon the installation.

The target UFS filesystem was initialized with:

```sh
newfs -j /dev/ada0p5
```

After this, installation was able to continue.

This was another deliberate high-risk user action because `newfs` is destructive when applied to the wrong device.

---

## 6. FreeBSD User Creation

The main user was created as:

```text
hagure
```

The account used:

- home: `/home/hagure`
- shell: `/bin/sh`
- group: `wheel`

FreeBSD does not provide `sudo` by default, so root access initially used:

```sh
su -
```

---

## 7. Basic System Configuration

The initial FreeBSD configuration included:

- hostname
- timezone
- user account
- networking
- package bootstrap
- package update/upgrade

Package management was successfully initialized:

```sh
pkg bootstrap
pkg update
pkg upgrade
```

---

## 8. Network Configuration

The available network interfaces were:

```sh
ifconfig -l
```

Result:

```text
re0 lo0 wlan0
```

The wireless device was identified with:

```sh
sysctl net.wlan.devices
```

Result:

```text
ath0
```

Therefore:

```text
ath0
 ↓
wlan0
```

was the wireless configuration.

---

## 9. Wi-Fi Configuration

Working Wi-Fi network:

```text
SSID: [REDACTED]
BSSID: [REDACTED]
```

The WPA configuration was structured as:

```text
network={
    ssid="[REDACTED]"
    psk=<hashed-password>
    bssid=[REDACTED]
}
```

Permissions were restricted:

```sh
chmod 600 /etc/wpa_supplicant.conf
```

The relevant `/etc/rc.conf` configuration was:

```text
hostname="hagure"

wlans_ath0="wlan0"

ifconfig_wlan0="WPA DHCP"

ifconfig_wlan0_ipv6="inet6 accept_rtadv"

create_args_wlan0="country ID regdomain APAC"
```

The connection successfully associated and obtained an address using:

```sh
dhclient wlan0
```

Internet connectivity was verified with `ping`.

---

## 10. Wi-Fi Driver Recovery Incident

At one point the wireless interface was manually reset:

```sh
pkill wpa_supplicant
ifconfig wlan0 down
ifconfig wlan0 ssid ""
ifconfig wlan0 up
```

Afterward, scanning stopped working.

The kernel log contained:

```text
ath_edma_recv_tasklet: sc_inreset_cnt > 0; skipping
```

The user rebooted.

After reboot:

- Atheros driver recovered
- Wi-Fi scanning worked again
- association worked
- DHCP worked
- Internet connectivity worked

This demonstrated that the problem could be temporary driver/kernel state rather than a persistent configuration error.

---

## 11. XFCE Installation

The user selected XFCE as the first desktop.

Installed:

```sh
pkg install xfce
```

Then:

```sh
pkg install dbus xfce4-session
```

D-Bus was enabled:

```sh
sysrc dbus_enable="YES"
```

---

## 12. LightDM Installation

Installed:

```sh
pkg install lightdm lightdm-gtk-greeter
```

Enabled:

```sh
sysrc lightdm_enable="YES"
```

The graphical login was then tested.

---

## 13. First Major Xorg Failure

LightDM initially failed because Xorg was missing.

The LightDM log showed:

```text
XServer 0: Can't launch X server, X not found in path
```

Xorg was installed:

```sh
pkg install xorg
```

After that, LightDM could find Xorg, but Xorg still failed.

---

## 14. Second Xorg Failure — Framebuffer Mode

The graphical screen failed and the console showed:

```text
(EE) Cannot run in framebuffer mode.
Please specify busID for all framebuffer devices

(EE) Please also check the log file at
"/var/log/Xorg.0.log"
for additional information.

(EE) Server terminated with error (1).
Closing log file.
```

PCI information showed Intel graphics.

The problem had therefore moved from:

```text
missing XFCE
```

to:

```text
Xorg graphics initialization
```

---

## 15. User Decision to Continue

The user chose to continue troubleshooting instead of abandoning XFCE.

This was a deliberate experimental decision because:

- the machine has only 2 GB RAM
- the graphics hardware is old
- Xorg had already failed
- simpler fallback options existed

The user nevertheless chose to continue with the Intel graphics path.

---

## 16. Intel DRM / KMS

The FreeBSD Intel graphics path was configured with `drm-kmod`.

Installed:

```sh
pkg install drm-kmod
```

Enabled:

```sh
sysrc kld_list+=i915kms
```

The important kernel module is:

```text
i915kms.ko
```

The graphical user was also intended to have access through the `video` group.

---

## 17. Reboot and KMS Verification

After enabling Intel KMS, the system was rebooted.

The module was later verified:

```sh
kldstat | grep i915
```

Output showed:

```text
i915kms.ko
```

This confirmed that Intel KMS was loaded.

---

## 18. XFCE Successfully Started

After the KMS configuration and reboot:

```text
FreeBSD
  ↓
LightDM
  ↓
Xorg
  ↓
i915kms
  ↓
XFCE
```

successfully reached the desktop.

The previous Xorg framebuffer error was resolved without needing a manual Xorg BusID configuration.

This is the current known-good graphical state.

---

## 19. `xfce4-session --version` Confusion

The following was tested from a root shell:

```sh
xfce4-session --version
```

It returned:

```text
xfce4-session: Cannot open display:
```

This did not indicate a broken XFCE installation.

The command was executed from a `su` root environment without the graphical session's display environment.

The actual desktop was already functioning.

---

## 20. Current RAM Baseline Task

The next task is to measure idle RAM before performing additional optimization.

Install:

```sh
pkg install xfce4-taskmanager
```

Launch:

```sh
xfce4-taskmanager
```

Then wait approximately 1–2 minutes after login without launching additional applications.

Record:

- RAM usage
- major processes
- background services
- approximate idle footprint

The purpose is to establish a baseline, not to benchmark the system.

### Current pause point

```text
FreeBSD 15.1
+ XFCE
+ LightDM
+ Xorg
+ i915kms
+ 2 GB RAM
↓
RAM baseline pending
```

---

# 21. User's Deliberate / Risky Decisions

This section explicitly separates personal decisions from normal installation steps.

### A. Repartitioning Debian's disk

The user chose to modify the existing Debian disk layout to make room for FreeBSD.

Risk included:

```text
partition damage
filesystem damage
bootloader damage
data loss
```

### B. Continuing after filesystem problems

The user continued after installer mount/filesystem problems and used:

```sh
newfs -j /dev/ada0p5
```

This was intentionally performed on the FreeBSD target.

### C. Continuing after Xorg failure

The user chose to keep pursuing XFCE after:

```text
Cannot run in framebuffer mode
```

instead of immediately switching to another desktop or abandoning the GUI installation.

### D. Continuing with old Intel graphics

The user chose to install and enable:

```text
drm-kmod
i915kms
```

on an old 2 GB machine.

### E. Using Laptop 3 as an experimental OS machine

The overall decision was to treat this machine as an operating-system laboratory:

```text
Debian
FreeBSD
Android-x86
FydeOS
BSD experimentation
```

rather than keeping it as a single-purpose stable machine.

---

# 22. Draft — Continue FreeBSD Adjustment

After recording the RAM baseline, continue gradually:

```text
XFCE working
    ↓
RAM baseline
    ↓
Audio
    ↓
Touchpad
    ↓
Display / resolution
    ↓
Network applet
    ↓
Power management
    ↓
XFCE compositor
    ↓
Fonts / themes
    ↓
Lightweight applications
    ↓
General cleanup
```

Do not optimize everything at once.

The purpose is to preserve a known baseline and identify which change affects behavior.

---

# 23. Draft — Add FreeBSD to GRUB

The planned boot architecture is:

```text
UEFI
 ↓
shared EFI partition
 ↓
Debian GRUB
 ├── Debian
 └── FreeBSD
```

Before modifying GRUB:

1. Confirm FreeBSD boots normally.
2. Confirm Debian still boots normally.
3. Inspect EFI boot entries.
4. Identify the FreeBSD EFI loader.
5. Test GRUB detection.
6. Add a FreeBSD entry if required.
7. Reboot.
8. Test Debian.
9. Test FreeBSD.

Possible tools to investigate:

```sh
efibootmgr
grub-mkconfig
grub-probe
```

Do not manually edit `/boot/grub/grub.cfg` as the first approach.

Prefer Debian's normal GRUB generation mechanism.

---

# 24. Draft — Sudden Crash / Shutdown Investigation

A sudden unexpected shutdown occurred during the broader XFCE setup process.

The user deliberately chose:

```text
finish getting XFCE working first
        ↓
investigate crash logs later
```

The investigation is therefore deferred.

Possible questions:

- Was it power-related?
- Was it kernel-related?
- Was it a graphics/DRM event?
- Was it storage-related?
- Was it ACPI-related?
- Was it a driver event?
- Was it an external power interruption?

Potential commands:

```sh
dmesg
```

```sh
dmesg -a
```

```sh
tail -100 /var/log/messages
```

```sh
grep -iE 'panic|error|fail|crash|drm|i915|ath|ata|ada|acpi' /var/log/messages
```

No cause should be assumed before inspecting the logs.

---

# 25. Draft — Card Reader Error

A hardware/system sensor reported an error associated with the card reader.

This is currently only a draft investigation item.

Do not immediately conclude that the physical card reader is defective.

Possible explanations include:

```text
hardware failure
driver absence
unsupported controller
firmware issue
power-management behavior
harmless device-probe error
```

First identify the controller:

```sh
pciconf -lv
```

Then inspect relevant kernel messages:

```sh
dmesg | grep -iE 'card|reader|sd|mmc'
```

or:

```sh
dmesg -a | grep -iE 'card|reader|mmc|sd'
```

The investigation should distinguish:

```text
sensor / diagnostic error
```

from:

```text
actual card-reader functionality failure
```

before any hardware conclusion is made.

---

# 26. Future Experimental OS Plan

After FreeBSD is sufficiently stable, the remaining free disk space may be used for further experiments.

Current conceptual plan:

```text
Debian
FreeBSD
Android-x86
FydeOS
```

Approximately 121 GB was left unallocated after the FreeBSD partitioning.

No additional partitioning should be performed until the current FreeBSD installation is confirmed stable.

---

# 27. Current Known-Good State

At the pause point:

```text
FreeBSD 15.1-RELEASE
        │
        ├── UFS root
        ├── FreeBSD swap
        ├── Atheros Wi-Fi
        │      └── wlan0
        ├── pkg working
        ├── Xorg installed
        ├── drm-kmod installed
        ├── i915kms loaded
        ├── LightDM enabled
        └── XFCE working
```

The graphical login is functional.

The immediate next task is only:

```text
measure idle RAM
```

No further optimization is required until that baseline is recorded.

---

# 28. Project Method

This installation is an experimental low-spec Unix/BSD project.

The working method is:

```text
understand
    ↓
test
    ↓
observe
    ↓
document
    ↓
adjust
```

rather than:

```text
change everything
    ↓
lose the baseline
    ↓
guess what caused the result
```

The user's deliberate risky decisions are documented separately so future troubleshooting can distinguish:

```text
planned experimentation
```

from:

```text
unexpected failure
```

and:

```text
normal configuration
```

from:

```text
user-chosen high-risk intervention
```

---

## Current checkpoint

**FreeBSD 15.1 + XFCE is successfully running on the 2 GB Intel i3 laptop.**

**Next:** record idle RAM with `xfce4-taskmanager`.

**Deferred drafts:**

- FreeBSD adjustment
- FreeBSD → GRUB integration
- sudden crash-log investigation
- card-reader error investigation
- additional OS experiments
