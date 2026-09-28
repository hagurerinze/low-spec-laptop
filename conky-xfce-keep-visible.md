# Conky — Keep Visible on XFCE

## Problem

A Conky configuration using:

```lua
own_window_type = 'desktop'
```

may disappear or become covered after clicking the desktop or interacting with windows in XFCE.

## Fix

Use `dock` as the Conky window type and add window hints that keep it below normal application windows while making it sticky and excluded from the taskbar/pager.

Replace:

```lua
own_window_type = 'desktop',
```

with:

```lua
own_window_type = 'dock',
own_window_hints = 'undecorated,below,sticky,skip_taskbar,skip_pager',
```

## Recommended Configuration

```lua
conky.config = {
    alignment = 'top_right',
    gap_x = 20,
    gap_y = 20,
    minimum_width = 220,

    own_window = true,
    own_window_type = 'dock',
    own_window_class = 'Conky',

    own_window_argb_visual = true,
    own_window_argb_value = 0,
    own_window_transparent = true,

    own_window_hints = 'undecorated,below,sticky,skip_taskbar,skip_pager',

    double_buffer = true,
    use_xft = true,
    font = 'DejaVu Sans:size=10',

    update_interval = 1,
    cpu_avg_samples = 2,
    net_avg_samples = 2,

    draw_shades = false,
    draw_outline = false,
    draw_borders = false,

    default_color = 'white',
}

conky.text = [[
${font DejaVu Sans:bold:size=12}${nodename}${font}

OS      : ${sysname} ${kernel}
Uptime  : ${uptime}

CPU     : ${cpu}% ${cpubar 6}
RAM     : $mem / $memmax
${membar 6}

Disk    : ${fs_used /} / ${fs_size /}
${fs_bar 6 /}

Time    : ${time %H:%M:%S}
Date    : ${time %A, %d %B %Y}
]];
```

## Restart Conky

After saving the configuration:

```bash
killall conky
conky -c ~/.config/conky/conky.conf &
```

Adjust the path if the configuration is stored elsewhere.

## Why This Works

`own_window_type = 'desktop'` integrates Conky with the desktop layer. Depending on the XFCE window manager behavior, desktop actions can cause Conky to be hidden or covered.

`own_window_type = 'dock'` makes Conky a separate dock-type window.

The window hints provide the following behavior:

- `undecorated` — no window decorations.
- `below` — stays below normal application windows.
- `sticky` — remains visible across workspaces.
- `skip_taskbar` — does not appear in the taskbar.
- `skip_pager` — does not appear in the workspace pager.

This configuration is intended for a desktop information overlay that should remain visible without interfering with normal application windows.
