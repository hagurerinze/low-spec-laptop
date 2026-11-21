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
