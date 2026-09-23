# Windows Ghost Spectre → Debian Dual Boot + Waydroid Session

Session dates: August–September 2026

## 1. Initial laptop state

Windows laptop:
- Dell Inspiron 3138
- Intel Celeron N2815 @ 1.86 GHz
- 4 GB RAM
- HDD WDC WD5000LPVX, ~500 GB
- Windows 10 Ghost Spectre Superlite
- Defender removed
- Ghost Toolbox available
- GPU/iGPU: Intel HD Graphics
- Disk mode: GPT / UEFI

Original goal:
- Try WSL2 on Ghost Spectre.
- WSL2 failed, and some attempts caused Windows to enter Recovery / a recovery loop.
- The WSL plan was dropped after that.
- Plan changed to dual-booting Debian alongside Windows.
- Waydroid would run on Debian instead, since it was judged more reliable than forcing an Android environment onto Ghost Spectre.

## 2. Failed WSL2 experiment

Attempted:
```powershell
wsl --install -d Debian
```

Symptoms:
- The Debian install process ran.
- Windows repeatedly entered Recovery / a recovery loop.
- A DISM attempt was also followed by a blue screen / recovery.
- `wsl --status`, `wsl -l`, and `wsl.exe --version` did not return normal WSL version output; `wsl.exe` just showed help text.
- Windows Ghost Spectre likely removed or modified components WSL depends on.

Feature check:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

Result:
```text
State : Disabled
```

And:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

Result:
```text
State : Disabled
```

`where.exe wsl`:
```text
C:\Windows\System32\wsl.exe
```

`wsl -l -o` could still list distros:
```text
Ubuntu
Debian
kali-linux
OracleLinux_7_9
OracleLinux_8_10
OracleLinux_9_5
SUSE-Linux-Enterprise-15-SP6
openSUSE-Tumbleweed
```

### Leftover WSL files found

A search for VHDX files found:

```text
C:\Users\Administrator\AppData\Local\Temp\5FD0D0BA-4024-4BEC-9D52-3E961D4E81D7\swap.vhdx
~36 MiB

C:\Users\Administrator\AppData\Local\wsl\{0ae89ccd-3bdb-48e5-b8f3-4f4150b0c7b2}\ext4.vhdx
300 MiB

C:\Users\Administrator\AppData\Local\wsl\{7ef6f0f0-4790-40e9-aef8-a155dc169da1}\ext4.vhdx
12 MiB
```

The WSL directory only contained those two GUID folders and their VHDX files.

Given that:
```text
Microsoft-Windows-Subsystem-Linux = Disabled
VirtualMachinePlatform = Disabled
```

there was no evidence of an actively working WSL distro that could be listed normally.

WSL session conclusion:
- WSL2 was not pursued further.
- Don't treat Ghost Spectre as a standard Windows environment.
- Modifying low-level Windows features risks triggering recovery/BSOD.
- Plan shifted to a Debian dual boot instead.

## 3. Pre-dual-boot Windows checks

BitLocker:

```powershell
Get-BitLockerVolume -MountPoint C:
```

Result:
```text
VolumeStatus : FullyDecrypted
Encryption   : 0
Protection   : Off
```

So BitLocker wasn't a blocker.

Disk:

```powershell
Get-PhysicalDisk | Select-Object FriendlyName,MediaType,HealthStatus,Size
```

Result:
```text
WDC WD5000LPVX-75V0TT0
HDD
Healthy
500107862016
```

Filesystem:

```powershell
chkdsk C: /scan
```

Key result:
```text
Windows has scanned the file system and found no problems.
No further action is required.
0 bad sectors
```

Space on C::

```text
Size: ~465 GB
SizeRemaining: ~447 GB
```

Available restore points:
```text
5/13/2026  Driver Booster : Microsoft ...
5/19/2026  Driver Booster : Dell Touchpad
8/3/2026   Windows Modules Installer
```

## 4. Dual boot partitioning

DiskPart showed:

```text
Disk 0   465 GB   GPT
Volume 0   C:      NTFS   ~465 GB   Boot
Volume 1           FAT32  100 MB     System
Volume 2           NTFS   649 MB     Hidden
```

C: was then successfully shrunk by:

```text
163840 MiB
```

Result:

```text
Windows C:        ~305 GB
Unallocated       160 GB
```

Windows EFI partition:
```text
100 MB FAT32
```

Recovery:
```text
649 MB NTFS
```

Both had to be preserved.

Fast Startup/hibernation was disabled:

```powershell
powercfg /h off
```

`powercfg /a` confirmed:
```text
Hibernate: unavailable
Fast Startup: unavailable
Hybrid Sleep: unavailable
Standby S3: available
```

## 5. Debian layout

Since the Debian installer requires a minimum EFI System Partition of about 300 MiB, the existing 100 MiB Windows EFI partition couldn't be reused for Debian.

Draft layout from the 160 GiB of free space:

```text
EFI       512 MiB    FAT32       /boot/efi
Root    61440 MiB    ext4        /
Swap     4096 MiB    linux-swap
Home    98304 MiB    ext4        /home
```

Total:
```text
512 + 61440 + 4096 + 98304 = 164352 MiB
```

Important note:
- The numbers above were a draft layout given at the time.
- If total free space was actually 163840 MiB, the 512 MiB EFI partition would need to be accounted for within that total. Because of this, the final layout needed to be adjusted by the installer so it didn't exceed available free space.
- The fixed goals stayed the same: 60 GiB root, 4 GiB swap, home using the remainder, and a 512 MiB Debian EFI partition.

Flags:
```text
EFI: boot, esp
Root: none
Swap: swap
Home: none
```

Windows EFI (100 MiB):
```text
KEEP
NO FORMAT
```

Windows C:
```text
KEEP
NO FORMAT
```

Recovery (649 MiB):
```text
KEEP
NO FORMAT
```

## 6. Debian clock issue after install

The initial `apt update` hit a signature error along the lines of:

```text
Not live until 2026-08-14T07:38:50Z
```

Cause:
- Debian's clock wasn't synced.
- `timedatectl` showed:
```text
System clock synchronized: no
NTP service: n/a
```

`timedatectl set-ntp true` returned:
```text
NTP not supported
```

Temporary fix:
```bash
sudo date -s "YYYY-MM-DD HH:MM:SS"
```

Once the clock was correct:
```bash
sudo apt update
```

Signature verification should not be disabled to work around this.

Dual boot note:
- The hardware RTC is shared between Windows and Debian.
- After installation, RTC/time synchronization settings need to be checked so Windows and Debian's clocks don't drift apart from each other.

## 7. Waydroid on Debian

Once Debian was installed, the repositories already had:

```text
waydroid:
Installed: (none)
Candidate: 1.6.3+ds-2~bpo13+1
Version:
1.6.3+ds-2~bpo13+1
http://deb.debian.org/debian trixie-backports/main amd64 Packages
```

Waydroid was installed from Debian backports:

```bash
sudo apt install -t trixie-backports waydroid
```

Version:

```bash
waydroid --version
```

Result:
```text
1.6.3
```

Kernel:

```bash
uname -r
```

Result:
```text
6.12.101+deb13-amd64
```

## 8. Checking the kernel Binder config

Command:

```bash
grep -E 'CONFIG_ANDROID_BINDER|CONFIG_MEMFD_CREATE' /boot/config-$(uname -r)
```

Result:

```text
CONFIG_MEMFD_CREATE=y
CONFIG_ANDROID_BINDER_IPC=m
# CONFIG_ANDROID_BINDERFS is not set
CONFIG_ANDROID_BINDER_DEVICES="binder"
# CONFIG_ANDROID_BINDER_IPC_SELFTEST is not set
```

Meaning:
- Binder is available as a kernel module.
- BinderFS was not compiled in.
- The default device is only `binder`.

Initially:

```bash
ls -l /dev/binder* /dev/hwbinder /dev/vndbinder 2>/dev/null
```

returned no devices.

Then:

```bash
sudo modprobe binder_linux
```

succeeded.

```bash
lsmod | grep binder
```

Result:
```text
binder_linux  237568  0
```

Only this device appeared:

```text
crw------- 1 root root 10, 261 ... /dev/binder
```

Missing:
```text
/dev/hwbinder
/dev/vndbinder
```

## 9. Attempt to change Binder devices

Tried:

```bash
sudo modprobe -r binder_linux
sudo modprobe binder_linux devices=binder,hwbinder,vndbinder
```

But the unload failed:

```text
modprobe: ERROR: could not remove 'binder_linux': Device or resource busy
```

Checked:

```bash
sudo fuser -v /dev/binder
```

No output.

```bash
sudo lsof /dev/binder
```

No process was using binder. There was a generic warning:
```text
can't stat() fuse.portal file system /run/user/1000/doc
```

Parameter:

```bash
cat /sys/module/binder_linux/parameters/devices
```

Result:
```text
binder
```

Parameter directory:

```text
alloc_debug_mask
debug_mask
devices
stop_on_user_error
```

`devices` is read-only:

```text
-r--r--r-- ... devices
```

`modinfo` wasn't available yet:

```text
bash: modinfo: command not found
```

But the module itself was found:

```text
/lib/modules/6.12.101+deb13-amd64/kernel/drivers/android/binder_linux.ko.xz
```

No `binder_linux` configuration was found in:

```text
/etc/modprobe.d/
/usr/lib/modprobe.d/
```

Command:

```bash
sudo modprobe -n -v binder_linux devices=binder,hwbinder,vndbinder
```

produced no output since the module was already loaded.

## 10. Next steps drafted for Waydroid

Not yet run:

```bash
waydroid init
```

Planned next steps:
1. Install `kmod` so `modinfo` is available.
2. Inspect module metadata:
   ```bash
   sudo apt install kmod
   modinfo binder_linux | grep -E '^(filename|parm|description)'
   modinfo binder_linux
   ```
3. Check module configuration:
   ```bash
   grep -R "binder_linux" /etc/modprobe.d/ /usr/lib/modprobe.d/ 2>/dev/null
   ```
4. Confirm the module file:
   ```bash
   find /lib/modules/$(uname -r) -type f -name 'binder_linux.ko*' -print
   ```
5. Don't create `/dev/hwbinder` or `/dev/vndbinder` manually with `mknod` before understanding Debian's Binder mechanism.
6. Don't force-unload the module.
7. Only move on to `waydroid init` once Binder is sorted out.

## 11. Waydroid configuration notes

Since this laptop is only:
```text
Celeron N2815
4 GB RAM
HDD
Intel HD Graphics
```

Priorities:
- Try vanilla Waydroid first.
- Avoid GAPPS at the initial stage.
- Test a minimal Android session.
- Only consider MicroG/GAPPS later, once stable, and only if genuinely needed.
- Waydroid is likely to be heavy on this machine, especially given the 4 GB RAM and HDD.
- The main goal is a usable Android/WhatsApp session, not high performance.

## 12. Mistakes / lessons learned

### WSL2
- Ghost Spectre is not a standard Windows 10 install.
- WSL2 depends on Windows features and virtualization components that a custom Windows build can remove or modify.
- `wsl.exe` still being present doesn't mean the whole WSL stack works.
- The WSL2 attempt caused Recovery/BSOD, so the experiment was stopped.
- Don't assume DISM/SFC are safe for repairing a custom Windows build without understanding what Ghost Spectre modified.

### Dual boot
- Don't delete the 100 MiB Windows EFI partition.
- Don't format the Windows EFI partition.
- Don't format Windows C: or Recovery.
- Shrinking Windows from Windows Disk Management is safer than letting the Linux installer do the resize.
- Leave the free space as Unallocated and let the Debian installer use it.

### APT
- A `Not live until` signature error can be caused by an incorrect system clock.
- Don't disable signature verification to work around a clock error.
- NTP may not be available on some installs; the clock can be set manually as a temporary fix.

### Waydroid
- Check the kernel's Binder config before running `waydroid init`.
- `CONFIG_ANDROID_BINDER_IPC=m` means Binder is a module.
- `CONFIG_ANDROID_BINDERFS` isn't available on this kernel.
- The default `CONFIG_ANDROID_BINDER_DEVICES="binder"` only produces `/dev/binder`.
- Don't create device nodes manually without understanding the driver/kernel configuration first.

## Status at end of this session

Windows:
```text
Boot normal
Ghost Spectre
C: ~305 GB
Debian space: ~160 GB
Fast Startup: disabled
BitLocker: disabled
Filesystem: clean
HDD health: reported Healthy
```

Debian:
```text
Kernel: 6.12.101+deb13-amd64
Waydroid: 1.6.3
Binder module: loaded
/dev/binder: exists
/dev/hwbinder: absent
/dev/vndbinder: absent
```

Waydroid:
```text
NOT initialized yet
Binder investigation in progress
```

Next command:
```bash
sudo apt install kmod
```

## 13. Update — Debian reinstall

After everything above, Debian on this laptop was reinstalled. The reason wasn't a problem with Waydroid or Binder — it was a partition layout mistake: swap and root ended up assigned to the wrong device nodes (`sda2` and `sda3` were swapped relative to the intended layout). My other laptop's Debian layout was already set up consistently, so this one was reinstalled to match that same layout for easier upkeep across machines.

After the reinstall, the setup arrived back at the same checkpoint as before: ready to try Waydroid again.
