# USB Flashdisk Test and Setup

This is my USB flashdisk test and setup on Debian.

My laptop only have 3 native USB port, so sometimes I use docking/hub.
Because I dont want test one by one too much, I test the flashdisk directly on native USB port.

## Initial SMART check

I used:

```bash
for d in /dev/sdb /dev/sdc /dev/sdd /dev/sde; do
    echo
    echo "========== $d =========="
    sudo smartctl -d scsi -x "$d"
done
```

Some USB flashdisk give this:

```text
response length too short
Terminate command early due to bad response to IEC mode page
A mandatory SMART command failed
```

This is not enough to say the flashdisk is bad. USB flashdisk controller often dont give full SMART information.

SanDisk Cruzer Blade was different:

```text
SMART support is: Available
SMART support is: Enabled
SMART Health Status: OK
```

But USB SMART is still limited.

## USB devices found

The devices was:

```text
/dev/sdb  Kingston DT 101 G2       14.7G
/dev/sdc  SanDisk Cruzer Blade     14.9G
/dev/sdd  Sony / USB Disk           2G
```

Later the VGEN USB was:

```text
/dev/sdd  USB Disk                   7.5G
serial: 8888801600001868
```

Important: USB device names can change after unplug and reconnect. So I always check `lsblk` before destructive command.

## First F3 problem

At first I tried:

```bash
f3write "$(findmnt -n -o TARGET /dev/sdb1)"
f3read "$(findmnt -n -o TARGET /dev/sdb1)"
```

It gave permission denied because the mounted directory was owned by root.

I fixed it with:

```bash
sudo chown "$USER:$USER" "$(findmnt -n -o TARGET /dev/sdb1)"
sudo chown "$USER:$USER" "$(findmnt -n -o TARGET /dev/sdc1)"
```

Then I checked:

```bash
ls -ld "$(findmnt -n -o TARGET /dev/sdb1)" "$(findmnt -n -o TARGET /dev/sdc1)"
```

## Kingston F3 result

Kingston was tested with F3.

```bash
f3write "$(findmnt -n -o TARGET /dev/sdb1)"
f3read "$(findmnt -n -o TARGET /dev/sdb1)"
```

Result:

```text
Data OK: 13.66 GB
Data LOST: 0.00 Byte
Corrupted: 0.00 Byte
Slightly changed: 0.00 Byte
Overwritten: 0.00 Byte
```

Average write speed was about:

```text
6.64 MB/s
```

Average read speed:

```text
17.11 MB/s
```

This was on USB 2.0, so speed was limited.

## SanDisk F3 result

SanDisk was tested:

```bash
f3write "$(findmnt -n -o TARGET /dev/sdc1)"
f3read "$(findmnt -n -o TARGET /dev/sdc1)"
```

Result:

```text
Data OK: 13.81 GB
Data LOST: 0.00 Byte
Corrupted: 0.00 Byte
Slightly changed: 0.00 Byte
Overwritten: 0.00 Byte
```

Average write speed:

```text
4.08 MB/s
```

Average read speed:

```text
18.37 MB/s
```

Also tested on USB 2.0.

## Sony F3 result

Sony was a 2 GB USB.

It was tested with F3 and gave very bad result:

```text
Data OK: 7.88 MB
Data LOST: 235.68 MB
Corrupted: 24.12 MB
Slightly changed: 0.00 Byte
Overwritten: 211.56 MB
```

This means Sony is not reliable storage.

I already made multiple backup images before destroying/reformatting it, so I was okay to make it usable again.

I will not use Sony as main storage.

## VGEN / Ventoy F3 probe

Before restoring Ventoy, I removed Ventoy and made the USB ext4 only for testing.

First:

```bash
sudo f3probe --destructive /dev/sdd
```

Result:

```text
Good news: The device `/dev/sdd' is the real thing

Usable size: 7.47 GB
Announced size: 7.47 GB
Module: 8.00 GB
Approximate cache size: 0.00 Byte
Physical block size: 512.00 Byte
```

So VGEN was real capacity, not fake capacity.

Then I formatted it for F3:

```bash
sudo mkfs.ext4 -F /dev/sdd
sudo mkdir -p /mnt/vgen-test
sudo mount /dev/sdd /mnt/vgen-test
sudo chown "$USER:$USER" /mnt/vgen-test
```

Then:

```bash
f3write /mnt/vgen-test
f3read /mnt/vgen-test
```

Result:

```text
Data OK: 6.87 GB
Data LOST: 0.00 Byte
Corrupted: 0.00 Byte
Slightly changed: 0.00 Byte
Overwritten: 0.00 Byte
```

Average write speed:

```text
4.22 MB/s
```

Average read speed:

```text
21.49 MB/s
```

So VGEN passed the destructive F3 test.

## Why F3 instead of only SMART

USB flashdisk SMART support is often very limited or not available.

F3 is useful because it actually writes test data and reads it back.

The test can show:

- real usable capacity
- corrupted data
- lost data
- changed data
- overwritten data

It is a better practical test for these flashdisk conditions.

## Filesystem decision

After testing, I decided to use different filesystems depending on the job.

### SanDisk

```text
SanDisk -> ext4
```

Use it for Linux-only storage.

I formatted it with:

```bash
sudo mkfs.ext4 -F -L "Haraguro SanDisk" /dev/sdc1
```

Then mounted it:

```bash
udisksctl mount -b /dev/sdc1
```

And made it writable by my user:

```bash
sudo chown "$USER:$USER" "$(findmnt -n -o TARGET /dev/sdc1)"
```

### Kingston

I first accidentally formatted Kingston as ext4:

```bash
sudo mkfs.ext4 -F -L "Haraguro Kingst" /dev/sdb1
```

But later changed the plan.

Kingston should be exFAT because I want it as cross-platform storage.

I tried:

```bash
sudo mkfs.exfat -n "Haraguro Kingst" /dev/sdb1
```

But exFAT label was too long.

I tried:

```text
Haraguro Kingst
Haraguro Kings
Haraguro King
Haraguro Kin
```

They failed with:

```text
input string is too long
```

Finally this worked:

```bash
sudo mkfs.exfat -n "Haraguro Ki" /dev/sdb1
```

Then mounted it:

```bash
udisksctl mount -b /dev/sdb1
```

So final:

```text
Kingston -> exFAT
```

It is for cross-platform use.

### VGEN

VGEN is used for Ventoy.

I downloaded:

```text
ventoy-1.1.17-linux.tar.gz
```

Extracted:

```bash
cd ~/Downloads
tar -xzf ventoy-1.1.17-linux.tar.gz
cd ~/Downloads/ventoy-1.1.17
```

Checked installer:

```bash
ls -l Ventoy2Disk.sh
```

Then checked the target carefully:

```bash
lsblk -d -o NAME,SIZE,MODEL,SERIAL,TRAN /dev/sdd
```

Result:

```text
sdd  7.5G  USB Disk  8888801600001868  usb
```

Then installed Ventoy:

```bash
sudo ./Ventoy2Disk.sh -i /dev/sdd
```

After installation:

```bash
lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINTS /dev/sdd
```

Result:

```text
sdd     7.5G
├─sdd1  7.4G  exfat  Ventoy
└─sdd2   32M  vfat   VTOYEFI
```

Then mounted:

```bash
udisksctl mount -b /dev/sdd1
```

VGEN is now the Ventoy USB.

No more destructive test is needed for VGEN because it already passed F3.

### Sony

Sony was the bad USB.

Its F3 result was very bad, but the old data was already backed up with multiple images.

I decided to make it FAT32 for compatibility.

After checking that the connected device was really Sony:

```bash
lsblk -d -o NAME,SIZE,MODEL,SERIAL,TRAN
```

Result:

```text
sdb  2G  Disk  00000000AF8B  usb
```

Then unmounted:

```bash
sudo umount /dev/sdb1
```

Then formatted:

```bash
sudo mkfs.fat -F 32 -n "Haraguro So" /dev/sdb1
```

It completed with this warning:

```text
Warning: lowercase labels might not work properly on some systems
```

The filesystem format still succeeded.

Then mounted:

```bash
udisksctl mount -b /dev/sdb1
```

Sony is now FAT32.

## Final USB map

```text
SanDisk  -> ext4   -> Linux only
Kingston -> exFAT  -> Cross-platform
VGEN     -> Ventoy -> Boot / ISO
Sony     -> FAT32  -> Disposable / experimental
```

Important:

Sony should NOT be trusted for important data because it already failed F3 badly.

Kingston and SanDisk passed F3 with 0 corrupted/lost data.

VGEN passed `f3probe` and full F3 write/read with 0 corrupted/lost data.

## About flashdisk write cycles

F3 does write a lot of data.

So yes, F3 consumes some NAND program/erase life. But one full F3 test is normally not enough to meaningfully destroy a healthy flashdisk.

F3 writing test files is not the same as one simple filesystem write. It writes many GB, so it does count as NAND writes.

Flashdisk has finite write endurance because NAND flash has a limited number of program/erase cycles. The controller uses wear leveling to spread writes across the NAND.

The important thing is that flashdisk endurance is not normally measured like HDD spin cycles. NAND has write endurance.

F3 should be used as a diagnostic test, not something to run every day.

## Important commands used

Check all disks:

```bash
lsblk -o NAME,SIZE,MODEL,SERIAL,FSTYPE,LABEL,MOUNTPOINTS
```

Check USB disks:

```bash
lsblk -d -o NAME,SIZE,MODEL,SERIAL,TRAN
```

Check SMART:

```bash
sudo smartctl -d scsi -x /dev/sdX
```

F3 filesystem test:

```bash
f3write /mount/path
f3read /mount/path
```

F3 destructive device probe:

```bash
sudo f3probe --destructive /dev/sdX
```

Unmount:

```bash
udisksctl unmount -b /dev/sdX1
```

Mount:

```bash
udisksctl mount -b /dev/sdX1
```

Power off USB:

```bash
udisksctl power-off -b /dev/sdX
```

## Safety lesson

Always check the device before using:

```bash
sudo mkfs...
sudo dd...
sudo f3probe --destructive...
sudo ./Ventoy2Disk.sh -i ...
```

Never assume `/dev/sdb` is always the same USB.

After reconnecting USB devices, Linux can assign a different `/dev/sdX`.

The serial number and model from `lsblk` are safer to identify the target.

---

# Part 2: Sony counterfeit diagnosis and boot experiment

This part is a follow-up done later, going deeper into why Sony failed F3 so badly,
and an experiment trying to turn Sony into a bootable rescue/live USB instead of
throwing it away.

## f3probe on Sony (deeper check)

Ran a proper destructive probe instead of just write/read:

```bash
sudo f3probe --destructive --time-ops /dev/sdb
```

Result:

```text
Bad news: The device `/dev/sdb' is a counterfeit of type limbo

Usable size: 250.96 MB (513962 blocks)
Announced size: 1.95 GB (4096000 blocks)
Module: 2.00 GB (2^31 Bytes)
Approximate cache size: 1.00 MB (2048 blocks), need-reset=no
Physical block size: 512.00 Byte
```

So Sony is a counterfeit "limbo" type chip. It reports 2GB to the OS but the real
NAND behind it is only about 251MB. Past that point the firmware wraps around and
silently overwrites earlier sectors instead of giving an error. That explains the
old F3 write/read result above (7.88MB OK / 211.56MB overwritten / 24.12MB corrupted
adds up to roughly the same ~230-250MB real size) — the corruption was already
there before this session, this just measured and confirmed it properly.

Also explains something I didn't understand before: I used this drive normally
last year and it seemed to hold 2GB fine. That's expected for this kind of fake
chip — it works completely normally as long as total data stays under the real
~250MB boundary. Only once you write past that does it start silently overwriting
earlier files. I remember moving about 1.7GB onto it once and only recovering
about 300MB intact afterward, which matches this pattern exactly.

### Locking the real boundary with f3fix

```bash
sudo f3fix --last-sec=513961 /dev/sdb
```

This edits the partition table so partition-aware tools only see the real usable
region, stopping them from writing into the fake zone. It does NOT change what the
firmware reports at the raw device level — `lsblk`, `dd`, `fdisk -l` etc will still
always show 2GB, because that number comes from the controller's `READ CAPACITY`
response, which only a vendor-specific low-level repair tool could change (and this
drive is a no-name/generic "USB Disk" controller, so no legitimate vendor tool
exists for it).

### Re-probing after the fix

Ran `f3probe --destructive` again just to confirm, and got a smaller number than before:

```text
Usable size: 233.36 MB (477920 blocks)
```

So it shrank from 250.96MB to 233.36MB just from that one extra destructive pass.
Lesson: running `f3probe --destructive` repeatedly wears the drive further, so it
should only be used sparingly (e.g. milestone checks), not as a repeated monitoring
method.

## Deciding what to do with a ~230MB counterfeit drive

Options considered:

- **Use it as ~200MB normal storage** — rejected. Even inside the real boundary,
  risk of losing files again wasn't worth it for a drive already proven flaky.
- **Dedicated degradation-logging experiment** — considered interesting but not
  necessary as a separate project. Decided to just fold monitoring into whatever
  normal use it gets instead of running a dedicated stress-test project.
- **Physical teardown / chip inspection** — good idea, but saved for after the
  drive fully dies rather than doing it now while it's still usable.
- **Turn it into a low-write rescue/boot USB** — the one chosen. Reasoning:
  since it's disposable anyway, get some use out of it as a small dedicated
  boot tool, while keeping writes minimal so it lasts as long as possible.

Also re-confirmed the other 3 drives don't need any of this: Kingston and SanDisk
already passed full-capacity `f3write`/`f3read` with 0 lost/corrupted bytes, and
V-Gen passed `f3probe --destructive` as "the real thing" at real capacity. A
generic/vague model name like V-Gen's "USB Disk" by itself means nothing — what
matters is the actual F3 result, and all three passed cleanly.

## Picking an OS for the ~230MB drive

Considered Puppy Linux, Porteus, and Slax first, but modern Puppy builds run
300-500MB+ which doesn't fit. Settled on **Tiny Core Linux**, specifically the
TinyCore (GUI, FLTK/FLWM) variant:

```text
Core (CLI only)             ~17MB
TinyCore (GUI)               ~23-26MB   <- picked this one
CorePlus (installer variant) ~248MB     (too big, barely fits real capacity)
```

TinyCore leaves ~200MB of headroom on the real ~230MB drive and runs entirely in
RAM after boot, so drive I/O basically stops once it's loaded.

### Verifying the ISO before writing

```bash
md5sum TinyCore-current.iso
ls -la TinyCore-current.iso
```

Checked against the official checksum published at:
`http://www.tinycorelinux.net/17.x/x86/release/TinyCore-17.1.iso.md5.txt`

```text
42db5663757add090857059c096a6801  TinyCore-17.1.iso
```

File size also matched exactly: 27262976 bytes. Both matched, so the download was good.

### Writing it

```bash
lsblk -d -o NAME,SIZE,MODEL,SERIAL,TRAN   # reconfirm sdb is really Sony
sudo umount /dev/sdb1 2>/dev/null
sudo dd if=TinyCore-current.iso of=/dev/sdb bs=4M status=progress conv=fsync
sync
```

Result:

```text
6+1 records in
6+1 records out
27262976 bytes (27 MB, 26 MiB) copied, 6.39256 s, 4.3 MB/s
```

Confirmed the write took:

```bash
lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINTS /dev/sdb
```

```text
sdb      2G iso9660 TinyCore
└─sdb1  26M iso9660 TinyCore
```

Filesystem changed from vfat to iso9660, label TinyCore, partition size matches
the ISO — clean write.

Note: since the boot partition is iso9660 (read-only by design), a writable
"canary" file for health monitoring can't go there. Instead the plan is to
periodically re-read the boot partition and compare its hash against the known-good
ISO hash, which needs zero extra writes:

```bash
sha256sum TinyCore-current.iso > ~/sony_reference.sha256
sudo head -c 27262976 /dev/sdb | sha256sum   # compare against the reference later
```

## BIOS boot attempt (failed)

Laptop is an Acer Aspire, BIOS is InsydeH2O. Checked the BIOS thoroughly:

- No CSM / Legacy-Boot toggle found anywhere in Security, Main, or Boot tabs —
  this BIOS appears to be legacy-style only, so UEFI/CSM wasn't actually the issue.
- Secure Boot: already Disabled.
- Boot priority order tab shows raw device-type entries (`HDDO`, `USB HDD`,
  `ATAPI CDROM`, `USB FDD`, `USB CDROM`, etc). Entries only show a model name when
  a device is actually detected in that slot — `USB HDD:` showed **no name**, with
  Sony plugged in before power-on. So BIOS's own POST scan isn't detecting it as a
  bootable device at all.

This likely traces back to something seen much earlier during initial testing —
`dmesg` showed `usb 1-2.1.1: device not accepting address 25, error -71` before
the drive finally enumerated under Linux. Linux's kernel retries failed USB
enumeration automatically, which is why it always works once already booted into
Debian. A legacy BIOS's USB stack during POST typically does not retry — if
enumeration fails on the first attempt, it's silently skipped from the boot list.

## GRUB-level check (inconclusive, but consistent)

Tried checking from GRUB's own command line during normal Debian boot (safe to
test — nothing here touches disk or config, it's all in-memory for that session):

```text
grub> insmod usb
grub> insmod usbms
grub> insmod ohci
grub> insmod uhci
error: ... not found
grub> insmod ehci
error: ... not found
grub> ls
(proc) (memdisk) (hd0) (hd0,msdos1) (hd1) (hd1,gpt4) (hd1,gpt3) (hd1,gpt2) (hd1,gpt1) (cd0)
```

No `(usb0)` or similar shows up — only internal HDD and the CD drive. The
`ohci`/`uhci`/`ehci` "not found" errors are expected on this laptop though: those
drivers are for old USB 1.1/2.0-era controllers, and this laptop (like most
post-2012 hardware) uses a single xHCI controller instead. Debian's legacy
`grub-pc` package typically doesn't bundle an `xhci` module at all (that's mostly
a `grub-efi` thing), so this GRUB test was structurally incomplete — it doesn't
prove Sony specifically failed here, just that this GRUB build can't talk to any
USB device on this hardware.

## Conclusion

Even though the GRUB test itself was inconclusive, three independent boot paths
gave the same result:

- BIOS's own raw POST scan doesn't see Sony as bootable (confirmed with drive
  plugged in beforehand)
- Ventoy + antiX failed on this same drive previously
- Direct `dd` + isolinux (TinyCore) also isn't picked up by BIOS

Same root cause each time: Sony's counterfeit controller has flaky USB
enumeration (the `error -71` from way back at the start), and this laptop's BIOS
doesn't tolerate that during POST, regardless of what's actually on the drive.

**Decision:** stop trying to get Sony to self-boot standalone on this laptop.
It still works completely normally once mounted from an already-running OS, so
the practical use going forward is:

- Access/verify the TinyCore image via a VM instead of bare metal:
  ```bash
  sudo qemu-system-x86_64 -hda /dev/sdb -m 256
  ```
  This reads the raw device directly through Linux, bypassing BIOS/USB
  enumeration entirely, so it actually confirms the TinyCore write is good.
- Otherwise, use Sony as a secondary drive: plug it in after booting from
  something else, mount and use it normally from within that OS.
- Physical teardown/inspection of the chip is saved for whenever the drive
  fully dies.
- Periodic hash comparison (see above) instead of repeated destructive
  `f3probe` for tracking any further degradation.
