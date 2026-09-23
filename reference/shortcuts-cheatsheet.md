# Shortcut Cheat Sheet

## bspwm (your actual configured bindings)

| Shortcut | Action |
|---|---|
| `Super + Return` | Open terminal (xterm) |
| `Super + Q` | Close focused window |
| `Super + M` | Toggle tiled / monocle layout |
| `Super + H / J / K / L` | Move focus (west / south / north / east) |
| `Super + Shift + H / J / K / L` | Swap window (west / south / north / east) |
| `Super + 1–4` | Switch to workspace I–IV |
| `Super + Shift + 1–4` | Move focused window to workspace I–IV |
| `Super + Shift + E` | Quit bspwm session (back to LightDM) |

**Note:** bspwm has no "minimize" — it's a tiling WM, not a traditional desktop.
Use workspace switching instead, or the tint2 panel buttons at the bottom
once workspace indicators are enabled.

**Clipboard (xterm, once configured):**
| Shortcut | Action |
|---|---|
| Click + drag | Select text |
| `Ctrl + Shift + C` | Copy selection |
| `Ctrl + Shift + V` | Paste |

*(No true "select all" exists in xterm — selection is visual/spatial, not
buffer-based. Click-drag from the top to the bottom of the visible screen
selects everything currently shown.)*

---

## XFCE (standard defaults — check Settings → Keyboard to confirm/customize)

| Shortcut | Action |
|---|---|
| `Alt + Tab` | Switch between open windows |
| `Alt + F4` | Close focused window |
| `Ctrl + Alt + Left/Right` | Switch workspace |
| `Ctrl + Alt + D` | Show desktop |
| `Super` | Open Whisker Menu (if bound) |
| `Print Screen` | Take a screenshot |
| `Alt + F7` | Move window (then click to drop) |
| `Alt + F8` | Resize window (then click to confirm) |

*XFCE keybindings are more flexible/customizable than bspwm's — check
**Settings Manager → Keyboard → Application Shortcuts** on your system to see
your actual current bindings, since these may differ if you've changed
anything before.*
