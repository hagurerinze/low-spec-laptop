# Light Openbox Custom Session (Draft)

A small and light desktop session for LightDM. It is made for a low-spec laptop (2GB RAM). XFCE stays as the daily session. This one is for when I want to use less RAM.

Goal: **light and usable**.

## What is used

| Part | Tool |
|---|---|
| Window manager | Openbox |
| Wallpaper | feh |
| Big clock on desktop | conky |
| Panel (top right) | tint2 |
| Dock (bottom center) | tint2 |

No compositor (no picom). Conky uses pseudo-transparent, and it works fine.

## Install

```sh
sudo apt install openbox tint2 conky-all feh fonts-montserrat
```

## Files

### Session script

`/usr/local/bin/custom-session` (run `sudo chmod +x` on it)

```sh
#!/bin/sh
feh --bg-fill /path/to/wallpaper.png
sleep 1
conky -c ~/.config/conky/clock.conf &
tint2 -c ~/.config/tint2/upper.tint2rc &
tint2 -c ~/.config/tint2/dock.tint2rc &
exec openbox-session
```

### LightDM entry

`/usr/share/xsessions/custom.desktop`

```ini
[Desktop Entry]
Name=Custom Session
Comment=Openbox + tint2 + conky
Exec=/usr/local/bin/custom-session
Type=Application
```

### Conky clock

`~/.config/conky/clock.conf`

```lua
conky.config = {
  own_window = true,
  own_window_type = 'desktop',
  own_window_transparent = true,
  own_window_hints = 'undecorated,below,sticky,skip_taskbar,skip_pager',
  alignment = 'top_left',
  gap_x = 130, gap_y = 200,
  update_interval = 10,
  double_buffer = true,
  use_xft = true,
  default_color = 'white',
};

conky.text = [[
${font Montserrat:bold:size=110}${time %H.%M}${font}
${voffset 10}${font Montserrat:size=14}${time %d %B %Y}${font}
]]
```

Note: do not use the `own_window_argb_*` lines without a compositor. The background turns black.

### tint2 panel and dock

Important lines in both files:

```ini
panel_layer = bottom
strut_policy = none
autohide = 0
```

- `panel_layer = bottom` and `strut_policy = none` let windows cover the panels.
- `autohide = 0` means the panel is always shown.
- Top panel: `panel_items = SC` (S = systray, C = clock).
- Dock: `panel_items = L` (L = launcher).

### Openbox rule

In `~/.config/openbox/rc.xml`, inside `<applications>`:

```xml
<application class="Tint2" type="dock">
  <layer>below</layer>
  <focus>no</focus>
</application>
```

Check the file, then reload:

```sh
xmllint --noout ~/.config/openbox/rc.xml && echo OK
openbox --reconfigure
```

## Status

Works:
- Session shows in LightDM and starts
- Clock, panel and dock are in the right place
- Maximized windows cover the panels
- Clock numbers do not pile up

Small issue:
- If a window is not maximized and there is a gap, clicking a panel makes it flash on top for a moment, then it goes back down. It is only cosmetic.

## To do

- Red italic weekday text ("Thursday") on the clock
- Move wallpaper to `~/Pictures`
- Maybe dock autohide later
