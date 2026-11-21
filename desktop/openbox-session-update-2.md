# Openbox Custom Session: Update 2

This is an update note for [openbox-custom-session.md](openbox-custom-session.md) and [openbox-session-update-1.md](openbox-session-update-1.md). The old notes are not changed. This file only has the new things.

## What is new

- Top panel (tint2) now has a dark rounded background
- Top panel is smaller and fits the clock
- Big conky clock has new colors and an outline
- Weekday name (for example "Friday") is added to the clock
- Brightness keys work

## Why the dark background on the top panel

My wallpaper is bright (white and pink). Tray icons (keyboard layout, wifi) have light colors, so they almost disappear. A dark, transparent background makes the icons and the white clock easy to see.

## Top panel (tint2)

File: `~/.config/tint2/upper.tint2rc`

```ini
rounded = 6
border_width = 0
background_color = #000000 55
border_color = #000000 0

panel_items = SC
panel_position = top right horizontal
panel_size = 175 44
panel_padding = 12 0 8
panel_margin = 6 6
panel_background_id = 1
panel_layer = bottom
strut_policy = none
autohide = 0
wm_menu = 1

systray_padding = 0 4 8
systray_icon_size = 18
systray_background_id = 0

time1_format = %H:%M:%S
time1_font = Montserrat Semi-Bold 11
time2_format = %d %B %Y
time2_font = Montserrat 7
clock_font_color = #ffffff 100
clock_padding = 8 0
clock_background_id = 0
```

The block at the top (`rounded`, `border_width`, `background_color`, `border_color`) is background number 1. That is why `panel_background_id = 1`.

How to change it:

| Setting | What it does |
|---|---|
| `panel_size = 175 44` | Width and height of the panel |
| `panel_margin = 6 6` | Distance from the right edge and from the top. Bigger first number moves the panel to the left |
| `panel_padding = 12 0 8` | Space between the panel edge and the content |
| `rounded = 6` | How round the corners are. Smaller number = less round |
| `background_color = #000000 55` | Black with 55 percent opacity. Bigger number = darker |

Notes:

- If the clock text is cut by the rounded corner, make the panel wider (`panel_size`) or make `rounded` smaller.
- `panel_padding` and `clock_padding` are the same on the left and right. You cannot make only the right side smaller. Change the panel width instead.
- The panel is on the right side, so when you make it wider, it grows to the left.

Restart tint2:

```sh
pkill tint2
tint2 -c ~/.config/tint2/upper.tint2rc &
tint2 -c ~/.config/tint2/dock.tint2rc &
```

## Big clock (conky)

File: `~/.config/conky/clock.conf`

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
  draw_outline = true,
  default_outline_color = '#7a0c24',
};

conky.text = [[
${font Montserrat:bold:size=110}${time %H.%M}${font}
${voffset -85}${goto 90}${color #ff2d55}${font Montserrat:italic:bold:size=40}${time %A}${font}${color}
${voffset 40}${goto 75}${font Montserrat:bold:size=16}${time %d %B %Y}${font}
]]
```

What I changed:

- White numbers with a dark red outline (`draw_outline` and `default_outline_color`). This is easy to read on the bright wallpaper.
- The weekday is a pink italic text. It is drawn on top of the big numbers, with a negative `voffset`.
- The date is bold and a bit bigger, so it is easier to read.

How to move things:

| Setting | What it does |
|---|---|
| `voffset` | Up and down. More negative = higher |
| `goto` | Left and right, from the left edge of the conky window. Bigger number = more to the right |
| `size=40` | Size of the weekday text |
| `gap_x`, `gap_y` | Position of the whole clock on the screen |
| `default_color` and `#ff2d55` | Colors of the numbers and the weekday |

If you move the weekday, check the date line too (`voffset 40`). The date line follows the weekday, so its space can change.

Restart conky:

```sh
pkill conky; conky -c ~/.config/conky/clock.conf &
```

## Keys

- Brightness keys work (with `brightnessctl`).
- Volume keys are not tested yet.

Media keys send these names: `XF86AudioLowerVolume`, `XF86AudioRaiseVolume`, `XF86AudioMute`, `XF86MonBrightnessDown`, `XF86MonBrightnessUp`. On some laptops you must press Fn together with the key. To check the key name, run `xev -event keyboard` and press the key.

## Status

Works:

- Top panel with dark rounded background and a readable clock
- Big clock with weekday, date and outline
- Brightness keys

Not done yet:

- Volume keys (three keys, to test later)
- The small tint2 flash when a panel is clicked through a gap
- Move the wallpaper to `~/Pictures`
- Add apps to the dock
