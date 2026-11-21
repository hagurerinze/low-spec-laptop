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

    -- Readability (key improvement)
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
