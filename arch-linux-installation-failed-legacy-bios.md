# Arch Linux Installation — Failed Boot Attempt

## Result

**Installation result: FAILED TO BOOT**

The Arch Linux installation itself completed successfully, including:

- Partitioning
- Formatting
- Mounting EFI, swap, root, and home
- `pacstrap`
- `genfstab`
- `arch-chroot`
- Timezone configuration
- Locale configuration
- Hostname configuration
- Root password
- User creation
- `sudo` configuration
- NetworkManager enablement
- GRUB installation
- GRUB configuration generation

GRUB reported:

```text
Installation finished. No error reported.
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/initramfs-linux.img
```

However, after rebooting, the firmware reported:

```text
No bootable devices
```

## Root Cause

The laptop uses a **legacy BIOS firmware**, not a true UEFI firmware.

The Arch installation was configured for **UEFI boot**:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

That installs the **UEFI version of GRUB** and expects the firmware to provide UEFI boot support.

This laptop does not provide that boot mode. Therefore, the firmware cannot boot the installed EFI/UEFI bootloader.

The important distinction is:

> **64-bit CPU / 64-bit OS support does not mean the firmware is UEFI.**

This laptop can run a 64-bit operating system while still using traditional **Legacy BIOS** boot.

## Terminology / What This Hardware Can Be Called

Depending on the documentation or discussion, this situation may be described informally as:

- Legacy BIOS with 64-bit OS support
- Legacy BIOS on a 64-bit system
- 64-bit-capable laptop with legacy firmware
- BIOS-only 64-bit system
- Legacy boot, but capable of running x86_64 operating systems
- "Hybrid/weird BIOS" (informal description)
- "32-bit BIOS, 64-bit CPU" — **only if the firmware itself is actually confirmed to be 32-bit**
- "UEFI but 64-bit" — **not correct for this machine if firmware is confirmed to be Legacy BIOS**
- "Legacy but 64-bit" — useful shorthand, but technically it describes the boot firmware and CPU/OS architecture as two separate properties

The safest technical description is:

> **A 64-bit-capable system using Legacy BIOS firmware rather than UEFI.**

Do not assume that the BIOS is 32-bit merely because it is Legacy BIOS. BIOS/UEFI boot mode and CPU/OS architecture are separate concepts.

## Why the Installation Did Not Boot

The installed system used:

```text
CPU/OS architecture: x86_64
Firmware boot mode: Legacy BIOS
Installed bootloader: UEFI GRUB
```

The problem is the mismatch:

```text
Legacy BIOS firmware
        ↓
cannot execute
        ↓
x86_64 UEFI GRUB
```

The correct configuration for this laptop is:

```text
Legacy BIOS firmware
        ↓
BIOS/Legacy GRUB
        ↓
Arch Linux x86_64
```

The operating system can remain **64-bit**. Only the bootloader installation method needs to change.

## Important Correction

The EFI partition and FAT32 filesystem are not inherently what make Arch Linux 64-bit or 32-bit.

The EFI partition was created because the original installation procedure assumed UEFI.

For a Legacy BIOS installation, the disk can instead use a BIOS-compatible GRUB installation. The exact partition layout can still include separate root, home, and swap partitions.

If using GPT with Legacy BIOS, GRUB may additionally require a small **BIOS Boot** partition. If using an MBR partition table, a BIOS Boot partition is generally not required.

## Current State

The installed Arch system should **not be considered corrupted** based solely on the boot failure.

The evidence indicates that the installation reached the bootloader stage successfully, but the firmware could not boot the UEFI bootloader.

The next step is to boot the Arch ISO again and install/configure **GRUB for Legacy BIOS** instead of `x86_64-efi`.

## Key Lesson

```text
64-bit CPU
≠
UEFI firmware

64-bit OS
≠
UEFI boot

Legacy BIOS
≠
32-bit CPU

Legacy BIOS + 64-bit CPU
→
perfectly capable of running a 64-bit Arch Linux installation
```

The correct target for this laptop is therefore:

```text
Arch Linux x86_64
+
Legacy BIOS boot
+
BIOS GRUB
```
