# Debian GRUB Cerydra Theme Setup

## Final State

GRUB theme:
`/usr/share/grub/themes/Cerydra/theme.txt`

GRUB background:
`/usr/share/grub/themes/Cerydra/background.png`

Configured in `/etc/default/grub`:

```ini
GRUB_THEME="/usr/share/grub/themes/Cerydra/theme.txt"
GRUB_BACKGROUND="/usr/share/grub/themes/Cerydra/background.png"
```

Custom GRUB entries:
- Shutdown
- Reboot

## Install Cerydra

```bash
cd ~/Downloads/StarRailGrubThemes

sudo cp -a /etc/default/grub /etc/default/grub.before-cerydra

sudo mkdir -p /usr/share/grub/themes

sudo rm -rf /usr/share/grub/themes/Cerydra
sudo cp -a assets/themes/Cerydra /usr/share/grub/themes/Cerydra

sudo sed -i   's|^#\?GRUB_THEME=.*|GRUB_THEME="/usr/share/grub/themes/Cerydra/theme.txt"|'   /etc/default/grub

grep -q '^GRUB_THEME=' /etc/default/grub ||   echo 'GRUB_THEME="/usr/share/grub/themes/Cerydra/theme.txt"' |   sudo tee -a /etc/default/grub >/dev/null
```

## Replace Debian GRUB Background

Debian was still loading:

`/usr/share/images/desktop-base/desktop-grub.png`

because `/etc/grub.d/05_debian_theme` uses the desktop-base wallpaper when `GRUB_BACKGROUND` is not explicitly set.

Set Cerydra:

```bash
sudo sed -i   's|^#\?GRUB_BACKGROUND=.*|GRUB_BACKGROUND="/usr/share/grub/themes/Cerydra/background.png"|'   /etc/default/grub

grep -q '^GRUB_BACKGROUND=' /etc/default/grub || echo 'GRUB_BACKGROUND="/usr/share/grub/themes/Cerydra/background.png"' | sudo tee -a /etc/default/grub >/dev/null
```

Backup:

```bash
sudo cp -a /etc/default/grub /etc/default/grub.before-cerydra-background
```

Regenerate:

```bash
sudo update-grub
```

Expected:

```text
Found theme: /usr/share/grub/themes/Cerydra/theme.txt
Found background image: /usr/share/grub/themes/Cerydra/background.png
```

Verify:

```bash
sudo grep -n 'background_image' /boot/grub/grub.cfg | head
```

Expected:

```text
117:if background_image /usr/share/grub/themes/Cerydra/background.png; then
```

## Add Shutdown and Reboot

Backup:

```bash
sudo cp -a   /etc/grub.d/40_custom   /etc/grub.d/40_custom.before-power-menu
```

Configure:

```bash
sudo tee /etc/grub.d/40_custom >/dev/null <<'EOF'
#!/bin/sh
exec tail -n +3 $0

menuentry 'Shutdown' {
    halt
}

menuentry 'Reboot' {
    reboot
}
EOF

sudo chmod 755 /etc/grub.d/40_custom
```

Regenerate:

```bash
sudo update-grub
```

Verify:

```bash
sudo grep -nE "menuentry 'Shutdown'|menuentry 'Reboot'"   /boot/grub/grub.cfg
```

Expected:

```text
menuentry 'Shutdown' {
menuentry 'Reboot' {
```

## Verification

```bash
grep '^GRUB_THEME=' /etc/default/grub
grep '^GRUB_BACKGROUND=' /etc/default/grub

sudo grep -n 'Cerydra' /boot/grub/grub.cfg | head
sudo grep -n 'background_image' /boot/grub/grub.cfg | head
sudo grep -nE "menuentry 'Shutdown'|menuentry 'Reboot'"   /boot/grub/grub.cfg
```

## Backups

```text
/etc/default/grub.before-cerydra
/etc/default/grub.before-cerydra-background
/etc/grub.d/40_custom.before-power-menu
/etc/lightdm/lightdm.conf.before-show-users
```

## LightDM Avatar Attempt

AccountsService avatar was configured at:

```text
/var/lib/AccountsService/icons/hagure.jpg
```

with:

```text
Icon=/var/lib/AccountsService/icons/hagure.jpg
```

The image was valid JPEG 1280x1280 and readable by LightDM, but Slick Greeter still did not display the avatar.

User list works.

LightDM avatar remains unused/not displayed.

## Notes

The important GRUB background fix was setting:

```ini
GRUB_BACKGROUND="/usr/share/grub/themes/Cerydra/background.png"
```

This avoids modifying Debian's `/etc/grub.d/05_debian_theme`.

Cerydra is now used for both the GRUB theme and the GRUB background.
