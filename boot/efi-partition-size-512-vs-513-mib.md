# EFI System Partition: Why 512 MiB Can Appear as 513 MiB

## Question

Under what conditions can an EFI System Partition (ESP) appear to grow on its own from **512 MiB to 513 MiB**?

## Conclusion

An EFI System Partition generally **does not grow on its own because of the FAT32 filesystem**.

If a partition that previously showed `512 MiB` later shows `513 MiB`, the more likely causes are:

- partition alignment;
- rounding by the partitioning tool;
- differences in how a program displays units;
- the installer intentionally creating a slightly larger size than the target.

Changes to the contents of the EFI filesystem **do not automatically enlarge the partition boundary**.

---

## 1. Partition Alignment

A partition's actual boundaries are determined by **sectors**, not the MiB number shown in a GUI.

Tools like:

- `parted`
- `fdisk`
- GParted
- Linux installers

can align partitions to a certain boundary.

As a result, when asked to create a partition around 512 MiB, the sector boundary chosen can end up slightly above 512 MiB.

Conceptual example:

```text
Target:
512 MiB

Result after alignment:
≈ 513 MiB
```

This isn't the EFI filesystem growing — the partition boundary itself was created slightly larger.

---

## 2. Installers Can Allocate Extra Space

An OS installer may request a minimum or target size, then apply alignment on top of that.

As a result, an ESP intended to be around 512 MiB can appear as:

```text
513 MiB
```

This is still normal, as long as the size is a genuine result of partition creation/alignment.

---

## 3. Unit Differences

Worth distinguishing:

```text
1 MiB = 1,048,576 bytes
1 MB  = 1,000,000 bytes
```

Different programs can use or display different units.

Because of this, the number shown in one program isn't always identical to the number shown in another.

---

## 4. EFI Contents Don't Make the Partition Bigger

For example, an ESP might have:

```text
Partition: 513 MiB
Used:       20 MiB
Free:      493 MiB
```

Files such as:

```text
EFI/BOOT/
EFI/debian/
EFI/Microsoft/
```

can increase space usage.

But:

```text
Used ↑
```

does not mean:

```text
Partition size ↑
```

The FAT32 filesystem does not normally change a partition boundary from 512 MiB to 513 MiB just because files were added.

---

## 5. How to Confirm Whether the Partition Actually Changed

Use:

```bash
lsblk -o NAME,SIZE,START,END,FSTYPE,PARTTYPENAME,MOUNTPOINTS
```

Then:

```bash
sudo fdisk -l /dev/sda
```

To see the size based on sectors more precisely:

```bash
sudo parted /dev/sda unit s print
```

Pay particular attention to:

```text
Start
End
Size
```

If the sector count and ESP boundary have genuinely changed, then **the partition size has actually changed**.

If only the number displayed by the GUI/tool differs:

```text
512 MiB
vs
513 MiB
```

while the sector boundary is the same, it's most likely just a **rounding or display difference**.

---

## Summary

| Condition | Can it appear as 512 → 513 MiB? |
|---|---|
| Partition alignment | Yes |
| Rounding by the partitioning tool | Yes |
| Installer allocating slightly more | Yes |
| MB vs MiB difference | Yes |
| EFI files growing | Should not |
| FAT32 auto-enlarging the partition boundary | No |
| Checking sectors with `parted unit s` | Most accurate method |

### Key principle

**Filesystem size and partition boundary size are two different things.**

To find out whether an ESP has actually changed size, don't just look at the `MiB` number. Check `Start` and `End` in **sectors**.

```bash
sudo parted /dev/sda unit s print
```
