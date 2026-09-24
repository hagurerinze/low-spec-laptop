# Windows Ricing: Draft

Laptop: Dell, 4GB RAM, Celeron N2xxx, Windows 10 (Ghost Spectre), idle RAM around 1GB.

## Status

- Active: StartAllBack (paid), accent color and dark mode already set
- Doing on my own: cursors
- Postponed: system icons, live wallpaper, .msstyles themes
- Rainmeter: **installed** (standard install), using the default skins for now
- MiniStat: **postponed**, to be edited and polished later

---

## 1. Rainmeter

### Installed

- Standard install from rainmeter.net
- Default skins are in use for now. Check Rainmeter's RAM in Task Manager (Details tab), and unload any default skins that are not needed via Manage → Skins → Unload.

### MiniStat (later)

Skin file: `MiniStat.ini` (clock, date, CPU, RAM). Not in use yet, kept for later editing.

Install when ready:

1. Copy it to `Documents\Rainmeter\Skins\MiniStat\MiniStat.ini`
2. Rainmeter → Manage → Refresh all → select MiniStat → Load

### Locked grid (nothing overlaps)

Skin size is 260 x 168. Left padding 16, right edge 244, content width 228.

| Zone | Y | Content |
|---|---|---|
| 0 | 0-168 | Background |
| 1 | 10-52 | Clock |
| 2 | 58-72 | Date |
| - | 84 | Divider line |
| 3 | 94-120 | CPU: label, value, bar (Y 112) |
| 4 | 128-152 | RAM: label, value, bar (Y 146) |

Each zone keeps at least 6 px of space from the next one. When restyling (fonts, colors, sizes), keep following this grid. To add a meter, put it in a new zone below Y 152 and increase the background height.

### Locking the position

1. Right-click the skin → Position → **On desktop** (default, does not cover windows)
2. Drag it to the top-right corner, away from desktop icons and the taskbar
3. Once it fits: right-click → Position → turn off **Draggable**
4. Optional: **Click through**, so clicks on the desktop are not blocked by the skin

### Avoiding collisions with other tools

- StartAllBack only handles the taskbar and Start menu, so it does not compete with the skin for desktop space.
- The other tools in this draft (TranslucentTB, ExplorerPatcher, Windhawk, OldNewExplorer) are not installed yet, so Rainmeter is locked down on its own first.
- Live wallpaper later: check that the skin still shows on top of it.

---

## 2. Tool drafts (postponed)

None of these are installed yet. RAM usage is unknown, so measure it yourself in Task Manager.

### TranslucentTB

- Function: transparent/blur/acrylic taskbar with per-state rules (desktop, maximized window, Start open)
- Conflict: with the taskbar transparency option in StartAllBack
- Only use it if dynamic behavior is needed. Otherwise skip it.

### ExplorerPatcher

- Function: patches for the taskbar, Start, context menu, Alt+Tab, and File Explorer
- Conflict: large overlap with StartAllBack, do not install both together
- Most likely will not be used.

### Windhawk

- Function: a collection of small mods for the shell and Explorer
- Conflict: only if picking mods that duplicate StartAllBack features
- Rule: install one mod at a time and check Task Manager each time.

### OldNewExplorer

- Function: changes the look of File Explorer (for example ribbon style, details pane)
- Conflict: StartAllBack also has its own File Explorer options, so check whether it is already enough
- Note: an old tool, so compatibility with the Windows 10 build used by Ghost Spectre needs testing first. Create a restore point before installing.

---

## 3. WSL1 / WSL2 (draft)

Does not touch the desktop layer, so it does not collide with the Rainmeter skin.

### Order of attempts

1. **Check virtualization**: Task Manager → Performance → CPU → "Virtualization". If Disabled, try enabling it in the Dell BIOS. This decides whether WSL2 is possible.
2. **WSL1 first** (does not need virtualization):
   `dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`
   If it fails, the components were probably removed from the Ghost Spectre build. Try again using an original Windows 10 ISO of the same build as the source. The chances are low if the components were already deleted.
3. **WSL2** only if virtualization is available. Extra feature: `VirtualMachinePlatform`. Limit its RAM via `%USERPROFILE%\.wslconfig`:

   ```
   [wsl2]
   memory=1GB
   processors=2
   ```

### Alternatives if WSL cannot be restored

- **MSYS2**: Unix-style shell and packages, the most flexible
- **Git Bash**: the lightest, enough for git, ssh, and basic tools
- **Cygwin**: full Unix-like environment, heavier

Choose WSL if a full Debian userland with apt is needed. Choose MSYS2 or Git Bash if a shell and basic tools are enough.

---

## 4. Uploading to GitHub

### Destination

The planned repo: **low-spec-laptop**. Its description already covers Celeron N2xxx with 4GB, so this Dell fits there. Create a new `windows/` folder to keep it separate from the Debian notes.

### Folder structure

```
low-spec-laptop/
└── windows/
    ├── ricing-draft.md
    └── rainmeter/
        └── MiniStat/
            └── MiniStat.ini
```

Rename `windows-ricing-draft.md` to `ricing-draft.md` when copying.

### License

- `ricing-draft.md` (notes) → **CC-BY-NC-4.0**
- `MiniStat.ini` (code/skin config) → **MIT**

This follows the dual-license decision for the repo. For clarity, add a comment line at the top of `MiniStat.ini`:

```
; License: MIT
```

### Commits

Split into two commits to keep the history clean:

```
git add windows/ricing-draft.md
git commit -m "docs(windows): add ricing draft for Ghost Spectre (Rainmeter, WSL, tool drafts)"

git add windows/rainmeter/MiniStat/MiniStat.ini
git commit -m "feat(rainmeter): add MiniStat skin (clock, date, CPU, RAM)"

git push
```

When MiniStat is edited later, use a new commit, for example:
`style(rainmeter): restyle MiniStat (fonts, colors)`
