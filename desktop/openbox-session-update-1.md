# Openbox Custom Session: Update 1

This is an update note for [openbox-custom-session.md](openbox-custom-session.md). The first note is not changed. This file only has the new things.

## What is new

- Keybinds added to Openbox
- Brightness keys (needs `brightnessctl`)
- Second screenshot key
- Tips for editing `rc.xml` safely
- A way to test `rc.xml` without logging out (Xephyr)

## Extra install

```sh
sudo apt install brightnessctl
sudo usermod -aG video $USER
```

Log out and log in again. After that, `brightnessctl` works without sudo.

## Keybinds

Keybinds are in `~/.config/openbox/rc.xml`, inside the `<keyboard>` block. Make a backup first:

```sh
cp ~/.config/openbox/rc.xml ~/.config/openbox/rc.xml.bak
```

I only add new keys. The default keys stay as they are (for example `W-e` still opens Konqueror, and `Print` still uses scrot).

In Openbox: `W` = Super (Windows key), `A` = Alt, `C` = Ctrl, `S` = Shift.

| Key | What it does |
|---|---|
| `W-Return` | Open terminal (xfce4-terminal) |
| `W-b` | Open Brave |
| `W-space` | Open the Openbox menu |
| `W-q` | Close window |
| `W-Up` | Maximize / restore |
| `W-Down` | Minimize |
| Volume keys | Volume up, down, mute (pactl) |
| Brightness keys | Brightness up, down (brightnessctl) |
| `Shift+Print` | Screenshot with xfce4-screenshooter |

Default keys that I did not change: `Print` and `Alt+Print` (scrot), `W-d` (show desktop), `W-e` (Konqueror), `W-F1` to `W-F4`, `W-S-arrow` keys.

Before you add a key, check that it is not used already:

```sh
grep -n 'keybind key="W-' ~/.config/openbox/rc.xml
grep -nE "XF86|Print" ~/.config/openbox/rc.xml
```

The block to add (only the content, not the `<keyboard>` tags). Put it before the line `</keyboard>`:

```xml
<!-- Custom: apps -->
<keybind key="W-Return">
  <action name="Execute"><command>xfce4-terminal</command></action>
</keybind>
<keybind key="W-b">
  <action name="Execute"><command>brave-browser</command></action>
</keybind>
<keybind key="W-space">
  <action name="ShowMenu"><menu>root-menu</menu></action>
</keybind>
<!-- Custom: window -->
<keybind key="W-q">
  <action name="Close"/>
</keybind>
<keybind key="W-Up">
  <action name="ToggleMaximize"/>
</keybind>
<keybind key="W-Down">
  <action name="Iconify"/>
</keybind>
<!-- Custom: volume -->
<keybind key="XF86AudioRaiseVolume">
  <action name="Execute"><command>pactl set-sink-volume @DEFAULT_SINK@ +5%</command></action>
</keybind>
<keybind key="XF86AudioLowerVolume">
  <action name="Execute"><command>pactl set-sink-volume @DEFAULT_SINK@ -5%</command></action>
</keybind>
<keybind key="XF86AudioMute">
  <action name="Execute"><command>pactl set-sink-mute @DEFAULT_SINK@ toggle</command></action>
</keybind>
<!-- Custom: brightness -->
<keybind key="XF86MonBrightnessUp">
  <action name="Execute"><command>brightnessctl set +10%</command></action>
</keybind>
<keybind key="XF86MonBrightnessDown">
  <action name="Execute"><command>brightnessctl set 10%-</command></action>
</keybind>
<!-- Custom: screenshot -->
<keybind key="S-Print">
  <action name="Execute"><command>xfce4-screenshooter</command></action>
</keybind>
```

If `pactl` is not available, use `amixer set Master 5%+`, `amixer set Master 5%-` and `amixer set Master toggle`.

## Tips for editing rc.xml

- Always make a backup first.
- There is only one `<applications>` block and one `<keyboard>` block. Do not paste a second one. Put the new content inside the existing block.
- Check the file before you reload:

```sh
xmllint --noout ~/.config/openbox/rc.xml && echo OK
```

- If it does not say `OK`, restore the backup.
- Reload Openbox (only inside an Openbox session):

```sh
openbox --reconfigure
```

- Do not paste long text into the terminal with a heredoc. It can break in the middle. Use `nano` to make the file, and check it with `xmllint` before you use it.

## Test without logging out (Xephyr)

You can test `rc.xml` from XFCE with a small X window:

```sh
sudo apt install xserver-xephyr
Xephyr :2 -screen 1024x600 &
DISPLAY=:2 openbox
```

- Click inside the Xephyr window first.
- Some keys do not reach Xephyr: Super keys, volume and brightness keys, and `Shift+Print`. Test these in the real Custom Session.
- Close it with `pkill Xephyr`.
- If the mouse is stuck inside Xephyr, press Ctrl+Shift.

If a key does not work, run `xev -event keyboard`, press the key, and check the key name.

## Status

- Keybinds are added. I still need to test all of them in the real Custom Session.
- The small tint2 flash is still there (see the first note). It is only cosmetic.

## To do

- Test all keybinds in the real Custom Session
- Fix or reduce the tint2 flash
- Red italic weekday text ("Thursday") on the clock
- Move the wallpaper to `~/Pictures`
- Maybe dock autohide later
