# bspwm: feh bug fix and three conky layout

Date: 2026-10-23
Laptop: Debian, 2GB RAM, Celeron N3xxx, bspwm

## 1. The feh bug in bspwmrc

### Problem

I edited the last line of `bspwmrc` (the `feh` wallpaper line) in Mousepad.
After that, the config did not work right.

### Cause

A stray letter `e` was in front of the first line:

```sh
e#!/bin/sh
```

The first line must be exactly:

```sh
#!/bin/sh
```

The `feh` line itself was already correct:

```sh
feh --bg-fill "/home/hagure/Pictures/Etc/Neon Sakura Shrine Girl.png" &
```

### Good habits for the feh line

- Use the full path, not `~`.
- Put quotes around the path if the file name has spaces.
- Put `&` at the end, so `bspwmrc` does not wait for `feh`.
- Linux file names are case-sensitive.

### Quick checks

```sh
# does the file exist?
ls -l "/home/hagure/Pictures/Etc/Neon Sakura Shrine Girl.png"

# test feh by hand
feh --bg-fill "/home/hagure/Pictures/Etc/Neon Sakura Shrine Girl.png"

# bspwmrc must be executable
chmod +x ~/.config/bspwm/bspwmrc

# reload bspwm
bspc wm -r
```

## 2. Three conky layout

Files are in `~/.config/conky/`.

| File | Position | Shows | Font |
|------|----------|-------|------|
| `center.conf` | middle_middle | name, CPU, RAM, ROOT bars, network speed, kernel | JetBrains Mono |
| `right.conf` | top_right | big clock, date, battery, uptime | Inter |
| `left.conf` | top_left | swap, Wi-Fi, load, process count, top 4 RAM programs | JetBrains Mono |

All three use a transparent window (`own_window_type = 'desktop'`, ARGB, value 0).

### start.sh

```sh
#!/bin/sh

killall conky 2>/dev/null
sleep 1

conky -c ~/.config/conky/right.conf &
sleep 0.5
conky -c ~/.config/conky/center.conf &
sleep 0.5
conky -c ~/.config/conky/left.conf &
```

It is started from `bspwmrc`:

```sh
~/.config/conky/start.sh &
```

To test without reloading bspwm, run `~/.config/conky/start.sh` in a terminal.

### center.conf

```lua
conky.config = {
    alignment = 'middle_middle',

    gap_x = 0,
    gap_y = 0,

    minimum_width = 220,
    minimum_height = 170,

    update_interval = 2,

    own_window = true,
    own_window_type = 'desktop',
    own_window_transparent = true,
    own_window_argb_visual = true,
    own_window_argb_value = 0,
    own_window_hints = 'undecorated,below,sticky,skip_taskbar,skip_pager',

    double_buffer = true,

    draw_borders = false,
    draw_outline = false,
    draw_shades = false,

    use_xft = true,
    xftalpha = 0.95,

    font = 'JetBrains Mono:size=10',

    default_color = 'ECECEC',
    color1 = 'F6C1CF',
    color2 = 'AAAAAA',
    color3 = '89B4FA',

    cpu_avg_samples = 2,
    net_avg_samples = 2,

    no_buffers = true,
};

conky.text = [[
${alignc}${font JetBrains Mono:bold:size=15}${color1}Hagure${font}

${voffset 8}

${color2}CPU${alignr}${color}${cpu}%
${color1}${cpubar 4,140}

${voffset 5}

${color2}RAM${alignr}${color}${memperc}%
${color1}${membar 4,140}

${voffset 5}

${color2}ROOT${alignr}${color}${fs_used_perc /}%
${color1}${fs_bar 4,140 /}

${voffset 8}

${color3}↑${color} ${upspeedf wlp3s0} KiB/s
${color3}↓${color} ${downspeedf wlp3s0} KiB/s

${voffset 10}

${alignc}${color2}${kernel}
]];
```

### right.conf

```lua
conky.config = {
    -- Position
    alignment = 'top_right',
    gap_x = 36,
    gap_y = 36,
    maximum_width = 240,

    -- Font
    use_xft = true,
    override_utf8_locale = true,
    font = 'Inter:size=11',

    -- Text color
    default_color = '#F2F2F2',

    -- Readability
    draw_shades = true,
    default_shade_color = '#000000',
    default_shade_alpha = 90,

    -- Window
    own_window = true,
    own_window_type = 'desktop',
    own_window_transparent = true,
    own_window_argb_visual = true,
    own_window_argb_value = 0,
    own_window_hints = 'undecorated,below,sticky,skip_taskbar,skip_pager',

    -- Performance (laptop-safe)
    double_buffer = true,
    update_interval = 1.5,
    cpu_avg_samples = 2,
}

conky.text = [[
${voffset 6}
${alignr}${font Inter:size=46}${time %H:%M}${font}
${alignr}${font Inter:size=11}${time %a · %d %b %Y}${font}

${voffset 12}
${alignr}${hr 120}

${voffset 14}
${alignr}${font Inter:size=11}Battery ${battery_percent BAT0}%${font}
${alignr}${font Inter:size=11}Uptime ${uptime_short}${font}
]]
```

### left.conf (new)

```lua
conky.config = {
    alignment = 'top_left',
    gap_x = 36,
    gap_y = 36,
    maximum_width = 240,

    use_xft = true,
    override_utf8_locale = true,
    font = 'JetBrains Mono:size=10',

    default_color = 'ECECEC',
    color1 = 'F6C1CF',
    color2 = 'AAAAAA',
    color3 = '89B4FA',

    draw_shades = true,
    default_shade_color = '#000000',
    default_shade_alpha = 90,

    own_window = true,
    own_window_type = 'desktop',
    own_window_transparent = true,
    own_window_argb_visual = true,
    own_window_argb_value = 0,
    own_window_hints = 'undecorated,below,sticky,skip_taskbar,skip_pager',

    double_buffer = true,
    update_interval = 2,
    cpu_avg_samples = 2,
    top_name_width = 14,
};

conky.text = [[
${font JetBrains Mono:bold:size=13}${color1}System${font}

${voffset 6}
${color2}SWAP${alignr}${color}${swapperc}%
${color1}${swapbar 4,140}

${voffset 6}
${color2}WIFI${alignr}${color}${wireless_essid wlp3s0}
${color2}LOAD${alignr}${color}${loadavg}
${color2}PROCS${alignr}${color}${running_processes}/${processes}

${voffset 10}
${color3}Top RAM${color}
${color2}${top_mem name 1}${alignr}${color}${top_mem mem_res 1}
${color2}${top_mem name 2}${alignr}${color}${top_mem mem_res 2}
${color2}${top_mem name 3}${alignr}${color}${top_mem mem_res 3}
${color2}${top_mem name 4}${alignr}${color}${top_mem mem_res 4}
]];
```

### Notes

- `wlp3s0` is my Wi-Fi interface name. Change it if yours is different (check with `ip link`).
- Swap is useful to watch on 2GB RAM.
- The left and right conkys use the same 36px gap, so the corners look even.
- Ideas for later: calendar, disk info, now-playing music.
