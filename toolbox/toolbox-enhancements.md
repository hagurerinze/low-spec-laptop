# Toolbox Script — Enhancement Round Documentation

This covers the second round of work on `~/toolbox.sh`, after the initial version (see `toolbox-script.md`) was already working. Includes a real mistake made along the way and how it was handled.

## Additions made this round
- Figlet "Hagure" banner (`big` font) with a cyan→blue gradient, one-time line-by-line reveal animation on first launch only (not repeated on every menu redraw, to stay fast)
- Live uptime + memory status line
- Sectioned menu: Daily / Emergency Fixes / System, with per-option color coding
- New options: Organize `.md` files, Test internet connection, Re-enable Scroll Lock LED, Restart Bluetooth (Emergency), Restart WiFi (Emergency), Power Options submenu (Suspend/Hibernate/Hybrid Sleep/Reboot/Shutdown/Switch User/Logout), Open New Terminal, ADB Check, Task Manager submenu (btop/top/GUI), GParted
- Menu loops back after each action instead of exiting

## Mistake made — "Organize .md files" (first version)
First version of the `.md` organizer used `find ~ -iname "*.md"` with **no filtering or overwrite protection**, then ran it. This swept up **110 files total**, including every `.md` license/legal notice bundled inside Minecraft's Java runtime, two separate TLauncher JRE installs, and Brave's wallet folder — none of which were the user's own files.

Because `mv` doesn't warn before overwriting, and many of these bundled license files shared the same filename across different folders (e.g. `mesa3d.md` existed in 3+ places), **62 of the 110 moved files were silently overwritten** in the destination folder, leaving only 48 unique files. All 62 lost files were duplicate open-source license/legal notices (Mesa3D, ICU, zlib-style texts, etc.) — not anything the user created. The user's own 5 real documents (toolbox-script.md, smart-hdd-check.md, scrolllock-led-fix.md, bluetooth-obex-fix.md, holochips_test.md) all had unique filenames and were not affected.

**Resolution:** rather than attempt to reconstruct ~62 duplicate license files back into their original scattered runtime folders (low value, easily re-obtained from the software's own source if ever needed), the user's real documents were left in place and the fix moved to preventing this from happening again.

**Fix — rebuilt the feature with:**
- A `zenity --checklist` picker showing every found `.md` file individually, so the user selects exactly which ones to move (plus a manual "[ SELECT ALL ]" entry) instead of an unfiltered blanket move
- Overwrite protection: before moving, the script checks if the destination filename already exists, and if so appends `-0`, `-1`, etc. instead of silently overwriting

## Other bugs hit and fixed this round

### Colors printed as literal text (`\e[32m`)
Bash only interprets `\e` as an actual escape sequence with `$'...'` string syntax, not plain `"..."`. Fixed by defining all color variables with `$'\e[...m'`.

### Menu looked repetitive on blank Enter
Using bash's built-in `select` menu reprints the entire menu (banner included) if the user presses Enter without a valid number. Replaced with a manual `read`-based loop that only reprints "Invalid option, try again." on bad input, not the whole screen.

### GParted password prompt got skipped
`sudo gparted &` (backgrounded) let the script race ahead to "Press Enter to continue" before the password prompt could actually be typed into. Fixed by running it in the foreground (no `&`) with a yes/no confirm first — same pattern as System Update, which already worked correctly.

### Box border misalignment when centering
Initial attempt centered each menu line independently based on that line's own visible character length. Two things broke this:
1. Emoji (🎮 ⚠️ ⚙️) render as double-width in the terminal but count as a single character in bash, throwing off length calculations.
2. Centering each line separately means different lines get different left-padding — so a box's left/right border characters no longer lined up vertically, especially visible when the terminal window was maximized.

**Fix:** switched to computing **one single margin** for the whole screen (based on terminal width and banner width), applied identically to every printed line via a `pline()` helper. This guarantees every line shares the same left edge, so nothing can drift out of alignment relative to anything else. Also dropped the right-side vertical border character entirely (kept only left border + horizontal rules) since a right border requires exact width tracking per line to look clean, which was the actual source of fragility — removing it removed the whole class of bug rather than patching it repeatedly.

## Key lesson from this round
Any destructive/bulk file operation (like moving all `.md` files) should default to **selection + confirmation**, not blanket "move everything found" — especially in a home folder where installed applications routinely bundle files with the same names/extensions as user documents.

## Result
Toolbox now has 15 organized, color-coded, confirmed-safe options, a centered animated banner that stays aligned at any terminal width, and file operations that ask before overwriting or moving anything in bulk.
