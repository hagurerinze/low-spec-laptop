# GRUB Reimu Theme — Menu Position & Spacing Session

## Status

In progress — visual layout is close to the intended design.

## Theme

Theme directory:

```text
/boot/grub/themes/Reimu/
```

Main file:

```text
/boot/grub/themes/Reimu/theme.txt
```

Background:

```text
background.png
```

The background image is **not being modified**.

## Current Layout Baseline

Current `boot_menu` position:

```text
left = 100
top = 53%
```

Spacing values remain:

```text
item_height = 45
item_spacing = 40
```

Current intended structure:

```text
Choose an operating
system to start
        ↓
Debian GNU/Linux
        ↓
Advanced options for Debian GNU/Linux
        ↓
UEFI Firmware Settings
        ↓
Boot in X seconds
```

The Debian → Advanced → UEFI spacing is currently considered close enough.

The gap between the background's `Choose...` text and Debian is slightly smaller, while the gap between UEFI and the timeout text is slightly larger. Further fine-tuning is postponed.

## Timeout Label

The timeout label was initially around:

```text
top = 84%
```

After moving the boot menu downward, the timeout text became invisible because of its position relative to the boot menu.

This confirmed that the timeout disappearance was primarily a **position/overlap issue**, not simply a text-color issue.

Current planned timeout position:

```text
left = 100
top = 89%
align = "left"
```

Current timeout text:

```text
Boot in %d seconds
```

Current planned color:

```text
#3A3A3A
```

## Colors

`item_color` was temporarily changed from:

```text
#cccccc
```

to:

```text
#3A3A4A
```

A rollback to `#cccccc` is still being considered/tested.

Selected item text remains:

```text
selected_item_color = "#FFFFFF"
```

The selection bar is controlled by:

```text
selected_item_pixmap_style = "select_*.png"
```

Existing selection PNGs:

```text
select_c.png
select_e.png
select_w.png
```

No background regeneration is planned.

## GRUB Configuration Verification

The system uses the Reimu theme:

```text
/etc/default/grub:
GRUB_THEME="/boot/grub/themes/Reimu/theme.txt"
```

Generated GRUB configuration contains:

```text
set theme=($root)/boot/grub/themes/Reimu/theme.txt
```

After editing the theme:

```bash
sudo update-grub
```

## `grub-emu` Testing

`grub-emu` was tested as a possible preview method.

This build does not support the `-c` configuration-file option.

Although `grub-emu` launches, it displayed the default blue GRUB screen instead of reliably reproducing the EFI boot environment. Therefore it is not being used as the final visual validator.

Actual reboot/boot-screen testing remains the reliable verification method.

A temporary test configuration created earlier can be removed with:

```bash
rm -f /tmp/reimu-emu.cfg
```

## Current Baseline

```text
boot_menu:
    left = 100
    top = 53%

item_height = 45
item_spacing = 40

timeout label:
    left = 100
    top = 89%
    align = "left"
```

## Next Session

1. Fine-tune the timeout position if necessary.
2. Decide the final `item_color`.
3. Adjust the `Choose an operating system to start` visual/color relationship if needed.
4. Fine-tune the selected Debian entry position/appearance.
5. Compare the final visual gaps again.
6. Keep `background.png` unchanged.

## Important Constraint

The background artwork is treated as fixed.

Current work is focused on `theme.txt`, especially:

```text
left
top
item_color
selected_item_color
item_height
item_spacing
timeout label position/color
```
