# Hagure's Linux Lab Notes

Welcome to my learning and troubleshooting log.

## Purpose

- Document my learning process on old/low-spec hardware (2GB RAM Debian laptop, 10-year-old HDD, and others).
- Keep a record of real problems I found and how I solved (or failed to solve) them.
- Build a personal portfolio of practical Linux/system troubleshooting.

## Categories

- **System** — [Bluetooth OBEX Fix](./system/bluetooth-obex-fix.md), [Scroll Lock LED Fix](./system/scrolllock-led-fix.md), [SMART/HDD Health Check](./system/smart-hdd-check.md), [Manual Toolbox Script](./system/toolbox-script.md), [Quod Libet Crash Reset](./system/quodlibet_crash_reset_summary.md), [System Tuning — N3xxx](./system/system-tuning-n3xxx-summary.md)
- **Boot** — [Boot Chain Theming Recap](./boot/boot-chain-theming-recap.md), [Debian GRUB Cerydra Setup](./boot/debian-grub-cerydra-setup.md), [EFI Partition Size Explainer](./boot/efi-partition-size-512-vs-513-mib.md), [Arch Linux Legacy BIOS Boot Fail](./boot/arch-linux-installation-failed-legacy-bios.md), [GRUB Reimu Theme Session](./boot/grub-reimu-theme-session.md)
- **Desktop** — [Live Wallpaper — Debian XFCE Log](./desktop/live-wallpaper-debian-xfce-log.md), [Live Wallpaper System](./desktop/live-wallpaper-system.md), [XFWM4 Desktop Zoom Fix](./desktop/xfwm4-desktop-zoom-fix.md)
- **Storage** — [USB Flashdisk Test & Setup](./storage/usb_flashdisk_test_setup.md), [Home Directory Reorganization](./storage/home-directory-reorganization.md)
- **Android** — [Windows Ghost Spectre → Debian Dual Boot + Waydroid](./android/windows-ghost-spectre-debian-dualboot-waydroid.md)
- **Gaming** — [Prism Launcher — Minecraft 1.7.10 Session](./gaming/prism_launcher_1.7.10_session.md)
- **Learning** — [Folder Organization: What I Learned](./learning/folder-organization-learning.md), [Android Calculator Mod](./learning/android-calculator-mod.md)
- **Reference** — [Shortcut Cheat Sheet](./reference/shortcuts-cheatsheet.md)
- - **Windows** — [Windows Ricing Draft](./windows/windows-ricing-draft.md) (Rainmeter skin config: [MiniStat.ini](./windows/MiniStat.ini))

## Templates

The [`Examples/`](./Examples) folder has skeleton templates you can copy when writing a new note in this repo's style:
- [Calculator/redesign-style project](./Examples/CalculatorExample.md)
- [System tuning experiment](./Examples/DebianZRAMExample.md)
- [Mistakes/lessons log](./Examples/NotesMistakesExample.md)

## About me

I run Debian on a low-spec laptop (Intel Celeron, 2GB RAM) and enjoy making old hardware work well again, rather than replacing it. Most of these projects started as a simple annoyance (a password prompt, an LED that would not stay on) and turned into a deeper investigation of how Linux actually works underneath.

I use AI tools during development, but every fix here was tested, debugged, and understood by me — including the cases where the AI's first suggestion was wrong.

## License

Code is licensed under [MIT](LICENSE-CODE).
Notes and documentation are licensed under [CC BY-NC 4.0](LICENSE-DOCS.md).
