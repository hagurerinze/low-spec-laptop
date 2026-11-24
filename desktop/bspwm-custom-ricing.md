# Android / ChromeOS-Inspired Desktop with bspwm

**Project:** Dell Inspiron 3138  
**Operating System:** Debian GNU/Linux 13 (Trixie)  
**Display Server:** X11  
**Window Manager:** bspwm  
**Documentation period:** 2–5 November 2026  
**Status:** Prototype development

---

## 1. Project Overview

The goal is to transform the Debian XFCE desktop into a desktop environment inspired by:

- ChromeOS
- ChromeOS Flex
- FydeOS
- Android launcher interfaces
- Chromebook desktop experience

The project uses bspwm as the window manager and a custom GTK3 application as the desktop shell.

The intention is not to replace Debian or build a new operating system. The objective is to create a custom desktop interface on top of the existing Debian installation.

### 1.1 Target Interface

The desktop is divided into three main components.

**Top panel**

- Android/ChromeOS-inspired status bar.
- Workspace indicators.
- Date and time.
- Launcher placeholder.

**Left panel — Home Screen**

- Wallpaper.
- Clock and date.
- Widgets.
- Brave search bar.
- Favorite applications.

**Right panel — App Drawer**

- Application search.
- Installed application grid.
- Always visible alongside the home screen.

The left and right panels are intended to appear simultaneously when returning to the desktop.

### 1.2 Window Behavior

Normal applications should eventually use fullscreen mode.

The intended behavior:

1. The desktop shell acts as the home screen.
2. Opening an application displays the application above the shell.
3. Returning to the desktop reveals the home screen and app drawer again.

Fullscreen application integration is deliberately deferred until the shell is sufficiently developed.

### 1.3 Important Design Decisions

- The app drawer is not a popup.
- The `[≡]` button on Polybar is currently only a placeholder.
- Rofi is not intended to be the primary app drawer because it behaves like a popup.
- A custom GTK3 shell is preferred.
- The existing Polybar remains the top panel.
- The project must be developed incrementally, with each stage tested before continuing.

---

## 2. Hardware and Initial Environment

Target machine: Dell Inspiron 3138.

| Component | Specification |
|---|---|
| CPU | Intel Celeron N2815 |
| CPU frequency | 1.86 GHz |
| CPU cores | 2 |
| RAM | 4 GB |
| Storage | WDC WD5000LPVX 500 GB HDD |
| Display | 1366 × 768 |
| Operating system | Debian 13 Trixie |
| Desktop before project | XFCE |
| Display server | X11 |
| Display manager | LightDM |

Initial environment:

- Debian XFCE was already installed.
- Conky was available.
- bspwm, sxhkd, and Polybar were not initially installed.
- Rofi was not installed.
- The system was using X11.

The project targets the Dell Inspiron 3138, not the Acer Aspire ES1-432.

---

## 3. Stage 1 — Installing bspwm and sxhkd

### 3.1 Installation

```bash
sudo apt update
sudo apt install bspwm sxhkd
```

Installed components:

- bspwm 0.9.10
- sxhkd 0.6.2

Session file:

```text
/usr/share/xsessions/bspwm.desktop
```

### 3.2 bspwm Configuration

Configuration file:

```text
~/.config/bspwm/bspwmrc
```

Initial configuration:

```sh
#!/bin/sh

# Start keyboard shortcuts
pgrep -x sxhkd >/dev/null || sxhkd &

# Window settings
bspc config border_width 2
bspc config window_gap 8
bspc config split_ratio 0.50
bspc config borderless_monocle true
bspc config gapless_monocle true

# Focus settings
bspc config focus_follows_pointer true
bspc config pointer_follows_focus false

# Colors
bspc config normal_border_color "#444444"
bspc config focused_border_color "#5294e2"
bspc config active_border_color "#777777"

# Desktop settings
bspc monitor -d 1 2 3 4
```

Make the configuration executable:

```bash
chmod +x ~/.config/bspwm/bspwmrc
```

### 3.3 sxhkd Configuration

Configuration file:

```text
~/.config/sxhkd/sxhkdrc
```

Initial shortcuts:

```sh
# Terminal
super + Return
    xfce4-terminal

# Close focused window
super + shift + q
    bspc node -c

# Quit bspwm
super + shift + Escape
    bspc quit

# Reload bspwm configuration
super + shift + r
    bspc wm -r

# Focus windows
super + {h,j,k,l}
    bspc node -f {west,south,north,east}

# Move windows
super + shift + {h,j,k,l}
    bspc node -s {west,south,north,east}

# Switch desktops
super + {1-4}
    bspc desktop -f '^{1-4}'
```

Reload sxhkd:

```bash
pkill -x sxhkd
sxhkd &
```

### 3.4 Verification

The following behavior was tested successfully:

- Login to bspwm through LightDM.
- Open a terminal using `Super + Enter`.
- Switch workspaces using `Super + 1` through `Super + 4`.
- Navigate between tiled windows.
- Close a focused window.

The bspwm session was confirmed as X11.

---

## 4. Stage 2 — Wallpaper

The wallpaper is stored at:

```text
/home/hagure/Pictures/Etc/upscalemedia-transformed(1).jpeg
```

The `feh` application was available at:

```text
/usr/bin/feh
```

Apply wallpaper:

```bash
feh --bg-fill "/home/hagure/Pictures/Etc/upscalemedia-transformed(1).jpeg"
```

The wallpaper was successfully displayed on bspwm workspaces.

At this stage, the wallpaper command had not yet been integrated into the final startup configuration.

---

## 5. Stage 3 — Polybar

### 5.1 Installation and Verification

Polybar was installed and verified.

Version:

```text
polybar 3.7.2
```

Available features included:

- ALSA
- curl
- i3
- MPD
- Network
- PulseAudio
- XKB

### 5.2 Configuration

File:

```text
~/.config/polybar/config.ini
```

Configuration:

```ini
[bar/main]
width = 100%
height = 30
offset-x = 0
offset-y = 0

fixed-center = true
bottom = false

background = #202124
foreground = #e8eaed

line-size = 0
border-size = 0
padding-left = 1
padding-right = 1

module-margin = 1

font-0 = monospace:size=10

modules-left = launcher bspwm
modules-center =
modules-right = date

wm-restack = bspwm
override-redirect = false

[module/bspwm]
type = internal/bspwm

label-focused = %name%
label-focused-background = #3c4043
label-focused-padding = 2

label-occupied = %name%
label-occupied-padding = 2

label-empty = %name%
label-empty-foreground = #777777
label-empty-padding = 2

[module/date]
type = internal/date
interval = 1

date = %a, %d %b
time = %H:%M

label = %date%  %time%

[module/launcher]
type = custom/text
content = [≡]
content-foreground = #e8eaed
content-padding = 1
```

### 5.3 Manual Test

```bash
polybar -c ~/.config/polybar/config.ini main
```

Polybar successfully displayed:

- Launcher placeholder.
- Workspace indicators.
- Date and time.

### 5.4 Startup Integration

A backup was created:

```bash
cp ~/.config/bspwm/bspwmrc ~/.config/bspwm/bspwmrc.backup-panel
```

The following lines were appended to `bspwmrc`:

```sh
# Start Polybar
pkill -x polybar 2>/dev/null || true
polybar -c "$HOME/.config/polybar/config.ini" main &
```

Reload:

```bash
bspc wm -r
```

Polybar was confirmed running after the reload.

**Design decision:** `[≡]` remains a placeholder. It is not the app drawer launcher.

---

## 6. Stage 4 — GTK3 Availability

The project requires a custom GTK interface.

First, GTK development tooling was checked:

```bash
pkg-config --modversion gtk+-3.0
```

Result:

```text
pkg-config: command not found
```

This initially appeared to indicate missing GTK development support.

However, the Python GTK binding was tested:

```bash
python3 -c 'import gi; gi.require_version("Gtk", "3.0"); from gi.repository import Gtk; print(Gtk.get_major_version(), Gtk.get_minor_version())'
```

Result:

```text
3 24
```

Conclusion:

GTK3 was already usable through Python GObject Introspection.

Installing `pkg-config` was not necessary for the prototype.

---

## 7. Stage 5 — Creating the GTK Shell Prototype

Project directory:

```bash
mkdir -p ~/bspwm-shell
cd ~/bspwm-shell
```

Main file:

```text
~/bspwm-shell/shell.py
```

### 7.1 Initial Layout

The prototype contains two main GTK containers.

**Left container**

- HOME SCREEN heading.
- Clock/date/widgets placeholder.
- Brave search placeholder.
- Favorite applications placeholder.

**Right container**

- APP DRAWER heading.
- Application search placeholder.
- Application grid placeholder.

The initial interface was static. Search bars and application entries were not yet functional.

### 7.2 Initial CSS

The original CSS used solid backgrounds:

```python
CSS = b"""
window {
    background-color: #17191f;
}

.left-panel {
    background-color: #252a35;
}

.right-panel {
    background-color: #191c23;
}

.heading {
    color: #ffffff;
    font-size: 24px;
    font-weight: bold;
}

.description {
    color: #aeb6c2;
    font-size: 14px;
}

.search {
    background-color: #353b47;
    color: #ffffff;
    border-radius: 24px;
    padding: 12px;
    font-size: 16px;
}
"""
```

### 7.3 Initial GTK Window

The initial window was a normal GTK window:

```python
window = Gtk.Window(title="Bspwm Shell Prototype")
window.set_default_size(1200, 700)
window.set_position(Gtk.WindowPosition.CENTER)
window.connect("destroy", Gtk.main_quit)
```

The GTK window was displayed using:

```bash
python3 ~/bspwm-shell/shell.py
```

### 7.4 Initial Problem: bspwm Tiled the Shell

The prototype appeared as a normal tiled window.

This was expected behavior because bspwm manages normal application windows by default.

The shell needed to behave like a desktop background rather than a normal application window.

---

## 8. Stage 6 — Testing Floating and Fullscreen

### 8.1 Identifying the Window

The first attempt used the currently focused window:

```bash
bspc node "$(bspc query -N -n focused)" -t floating
```

Then:

```bash
bspc node "$(bspc query -N -n focused)" -t fullscreen
```

### Mistake: Focus Was Not the Shell

The terminal could remain focused after launching the GTK prototype.

Therefore, commands targeting the focused window could affect the terminal instead of the prototype.

The correction was to identify the GTK window explicitly.

### 8.2 Window Identification

```bash
wmctrl -l
```

The prototype was identified as:

```text
0x02400003  0 Hagure Bspwm Shell Prototype
```

Change only that window to floating:

```bash
bspc node 0x02400003 -t floating
```

Verify:

```bash
bspc query -N -n 0x02400003
```

```bash
bspc query -T -n 0x02400003 | python3 -c 'import json,sys; print(json.load(sys.stdin)["client"]["state"])'
```

Result:

```text
0x02400003
floating
```

### 8.3 Fullscreen Test

The prototype was then tested in fullscreen:

```bash
bspc node 0x02400003 -t fullscreen
```

Result:

- GTK shell occupied the screen.
- Terminal and other windows were covered.
- The fullscreen behavior worked.

However, this was only a test.

**Important distinction:**

Fullscreen does not make a window a desktop background. It simply covers other windows.

### 8.4 Closing the Fullscreen Prototype

`Alt + F4` did not work as expected.

The window was closed directly using its ID:

```bash
wmctrl -ic 0x02400003
```

This successfully closed the prototype.

---

## 9. Stage 7 — Testing Window Rules

### 9.1 Identifying WM_CLASS

The GTK prototype was launched again.

```bash
wmctrl -l
```

Window:

```text
0x02400003  0 Hagure Bspwm Shell Prototype
```

Check its class:

```bash
xprop -id 0x02400003 WM_CLASS
```

Result:

```text
WM_CLASS(STRING) = "shell.py", "Shell.py"
```

### 9.2 Temporary bspwm Rule

A temporary rule was applied:

```bash
bspc rule -a Shell.py state=floating layer=below
```

Purpose:

- `state=floating`: prevent normal tiling.
- `layer=below`: place the shell below normal windows.

This was a runtime test, not a permanent configuration in `bspwmrc`.

### 9.3 Workspace Confusion

The terminal and Brave were not immediately visible because they were on another workspace.

The user initially could not determine whether the shell was behind them or whether it was simply on another workspace.

Switch to workspace 1:

```text
Super + 1
```

Check workspace nodes:

```bash
bspc query -N -d 1
```

Check all windows:

```bash
wmctrl -l
```

The prototype was confirmed to exist on workspace 1 alongside terminal and Brave.

### 9.4 Verify Window State

```bash
bspc query -T -n 0x02400003 | python3 -c 'import json,sys; d=json.load(sys.stdin); print("state:",d["client"]["state"]); print("layer:",d["client"]["layer"]); print("desktop:",d["desktop"]["name"])'
```

Result:

```text
state: floating
layer: below
Traceback (most recent call last):
...
KeyError: 'desktop'
```

**Mistake:** the command assumed that the JSON response contained a top-level `desktop` object.

It did not.

The window state and layer were successfully retrieved before the Python exception occurred.

The actual JSON structure was inspected using:

```bash
bspc query -T -n 0x02400003 | python3 -m json.tool
```

Relevant output:

```json
{
    "id": 37748739,
    "splitType": "vertical",
    "splitRatio": 0.5,
    "vacant": true,
    "hidden": false,
    "sticky": false,
    "client": {
        "className": "Shell.py",
        "instanceName": "shell.py",
        "state": "floating",
        "layer": "below",
        "shown": true,
        "floatingRectangle": {
            "x": 83,
            "y": 34,
            "width": 1200,
            "height": 700
        }
    }
}
```

This confirmed:

- The shell was floating.
- The shell was on the below layer.
- The shell was shown.
- The shell was not yet a full-screen desktop background.

---

## 10. Stage 8 — GTK Desktop Window Type

The next attempt was to make GTK identify itself as a desktop window.

The previous normal GTK window declaration was changed to:

```python
window = Gtk.Window(title="Bspwm Shell Prototype")
window.set_default_size(1366, 768)
window.set_position(Gtk.WindowPosition.CENTER)
window.set_type_hint(Gdk.WindowTypeHint.DESKTOP)
window.connect("destroy", Gtk.main_quit)
```

The prototype was relaunched.

Observed result:

- The shell appeared as a desktop-like background.
- It no longer behaved like the earlier fullscreen test.
- It could remain behind normal windows.
- The shell was also observed appearing on another workspace.

This was a major step toward the intended desktop shell.

However, the final workspace behavior still needs more testing.

---

## 11. Stage 9 — First Transparency Attempt

The shell was still visually gray and opaque.

The original CSS used solid hexadecimal colors.

The CSS was changed to use RGBA:

```css
window {
    background-color: rgba(23, 25, 31, 0.75);
}

.left-panel {
    background-color: rgba(37, 42, 53, 0.75);
}

.right-panel {
    background-color: rgba(25, 28, 35, 0.75);
}
```

The shell still appeared mostly solid.

### 11.1 Checking the Compositor

The following command was used:

```bash
pgrep -a -f 'picom|compton|xcompmgr|xfwm4'
```

No process was returned.

Another check:

```bash
xprop -root _NET_WM_CM_S0
```

Result:

```text
_NET_WM_CM_S0: not found.
```

At that point, no compositor was detected through the process check.

### 11.2 Installing Picom

```bash
sudo apt install picom
```

Picom was launched manually:

```bash
picom --config /dev/null --backend xrender &
```

Process verification:

```bash
pgrep -a picom
```

Result:

```text
42848 picom --config /dev/null --backend xrender
```

The root property check still returned:

```text
_NET_WM_CM_S0: not found.
```

This particular `xprop` check was inconclusive. It should not be treated as proof that Picom was inactive, because the process check confirmed that Picom was running.

### 11.3 Second Transparency Test

The GTK prototype was relaunched while Picom was running.

Result:

The shell remained solid.

This indicated that the CSS alpha values alone were not sufficient.

---

## 12. Stage 10 — GTK RGBA Visual

The next correction was to make the GTK window use an RGBA visual.

### 12.1 Incorrect Attempt

The following code was initially added before the CSS provider:

```python
screen = Gdk.Screen.get_default()
visual = screen.get_rgba_visual()

if visual is not None:
    Gtk.Window.set_default_visual(visual)
```

Running the prototype produced:

```text
AttributeError: type object 'Window' has no attribute 'set_default_visual'
```

**Mistake:** `Gtk.Window.set_default_visual()` is not available in the GTK3 Python binding used here.

### 12.2 Corrected Implementation

The incorrect block was removed.

The RGBA visual was applied to the actual window instance instead:

```python
window = Gtk.Window(title="Bspwm Shell Prototype")

screen = Gdk.Screen.get_default()
visual = screen.get_rgba_visual()

if visual is not None:
    window.set_visual(visual)

window.set_default_size(1366, 768)
```

This resolved the previous Python exception.

### 12.3 Result

After restarting the prototype:

- Transparency became visible.
- The wallpaper could be seen through the panels.
- The panels still looked too thick.

This confirmed that the RGBA visual was an important missing part of the transparency setup.

---

## 13. Stage 11 — Reducing Panel Opacity

The initial RGBA value was:

```text
0.75
```

It was reduced to:

```text
0.40
```

The user requested a screenshot method that did not require a terminal.

### 13.1 Screenshot Shortcut

The available screenshot application was checked:

```bash
command -v flameshot xfce4-screenshooter scrot
```

Result:

```text
/usr/bin/xfce4-screenshooter
```

Because the current session was bspwm, the XFCE screenshot keyboard shortcut could not be assumed to exist.

A shortcut was therefore added manually.

Edit:

```bash
nano ~/.config/sxhkd/sxhkdrc
```

Append:

```sh
# Fullscreen screenshot
Print
    xfce4-screenshooter -f
```

Reload:

```bash
pkill -x sxhkd
sxhkd &
```

Now `Print` opens the screenshot GUI.

### 13.2 Screenshot Assessment

The screenshot showed:

- The wallpaper was visible.
- The left and right panels were still visually heavy.
- The interface looked more like two large translucent panels than a native Android home screen.
- The search bars were still solid.

The solid search bars were intentionally retained because they improve text readability.

### 13.3 Further Opacity Adjustment

The outer window background was changed to fully transparent:

```css
window {
    background-color: transparent;
}
```

The panel backgrounds were changed to:

```css
.left-panel {
    background-color: rgba(37, 42, 53, 0.18);
}

.right-panel {
    background-color: rgba(25, 28, 35, 0.18);
}
```

The `.search` CSS was left unchanged.

This made the wallpaper more visible and reduced the heavy appearance.

---

## 14. Stage 12 — Spacing and Positioning

The previous screenshot showed that the shell was too close to the Polybar and screen edges.

The layout was adjusted.

Original:

```python
main = Gtk.Box(orientation=Gtk.Orientation.HORIZONTAL, spacing=0)
window.add(main)
```

Changed to:

```python
main = Gtk.Box(orientation=Gtk.Orientation.HORIZONTAL, spacing=8)
main.set_border_width(8)
window.add(main)
```

Effects:

- `spacing=8` adds space between the left and right panels.
- `set_border_width(8)` adds an 8-pixel margin inside the shell window.

The prototype was restarted.

### 14.1 Latest Visual Assessment

The latest screenshot showed:

- Polybar remains at the top.
- GTK shell has visible margins.
- The shell no longer touches the screen edges.
- The center separation is clearer.
- The wallpaper is more visible through the panels.
- The overall layout is closer to the intended home screen.

The user considered this version better.

---

## 15. Current Configuration Summary

### 15.1 Current GTK Window Properties

The prototype uses:

```python
window = Gtk.Window(title="Bspwm Shell Prototype")

screen = Gdk.Screen.get_default()
visual = screen.get_rgba_visual()

if visual is not None:
    window.set_visual(visual)

window.set_default_size(1366, 768)
window.set_position(Gtk.WindowPosition.CENTER)
window.set_type_hint(Gdk.WindowTypeHint.DESKTOP)
window.connect("destroy", Gtk.main_quit)
```

### 15.2 Current CSS Direction

```python
CSS = b"""
window {
    background-color: transparent;
}

.left-panel {
    background-color: rgba(37, 42, 53, 0.18);
}

.right-panel {
    background-color: rgba(25, 28, 35, 0.18);
}
"""
```

Other CSS classes remain in the file:

- `.heading`
- `.description`
- `.search`

The search bar remains solid.

### 15.3 Current Layout Direction

```python
main = Gtk.Box(orientation=Gtk.Orientation.HORIZONTAL, spacing=8)
main.set_border_width(8)
window.add(main)
```

### 15.4 Current Runtime Components

| Component | Status |
|---|---|
| bspwm | Installed and working |
| sxhkd | Installed and working |
| Polybar | Installed and running |
| feh | Available |
| GTK3 | Available through Python GI |
| GTK shell | Prototype working |
| Picom | Installed and manually tested |
| Desktop window type | Tested |
| RGBA visual | Working |
| Screenshot shortcut | Added to sxhkd |

---

## 16. Mistakes and Lessons Learned

This section records failed attempts and corrections rather than hiding them.

| Mistake / Problem | Cause | Correction |
|---|---|---|
| GTK shell appeared tiled | Normal GTK windows are managed by bspwm | Explicitly set floating state during testing |
| Focused-window command could target terminal | Terminal remained focused | Identify the GTK window using `wmctrl -l` |
| Fullscreen covered all other windows | Fullscreen is not desktop/background behavior | Test desktop window type instead |
| `Alt + F4` did not close the prototype | Fullscreen/focus behavior made the shortcut unreliable | Close using `wmctrl -ic WINDOW_ID` |
| Terminal and Brave seemed inaccessible | They were on another workspace | Switch workspace and inspect windows |
| JSON query raised `KeyError: 'desktop'` | The assumed JSON structure was incorrect | Inspect the actual JSON using `python3 -m json.tool` |
| RGBA CSS did not initially work | GTK window did not yet use RGBA visual; compositor was also absent initially | Test Picom and apply `Gdk.Screen.get_rgba_visual()` to the window |
| `Gtk.Window.set_default_visual()` raised `AttributeError` | The method was unavailable in the GTK3 binding | Use `window.set_visual(visual)` |
| Transparency still looked thick | Alpha value was too high for the intended appearance | Reduce panel alpha to `0.18` and make the outer window transparent |
| Screenshot shortcut was unavailable | XFCE shortcuts were not automatically configured in bspwm | Register `Print` in sxhkd |

### Important Technical Lessons

1. A floating window is not automatically a desktop window.
2. A fullscreen window is not equivalent to a desktop background.
3. bspwm commands should target explicit window IDs when focus is uncertain.
4. A window's `WM_CLASS` is useful for creating targeted bspwm rules.
5. GTK RGBA CSS requires an appropriate RGBA visual.
6. Real transparency under X11 requires compositing support.
7. `xprop -root _NET_WM_CM_S0` should not be used alone as proof that Picom is inactive.
8. Workspace placement must be checked separately from window stacking.
9. Visual changes should be tested one at a time.

---

## 17. Current Problems and Deferred Work

### 17.1 Visual Problems

- The home screen still needs a more Android-like visual hierarchy.
- The clock and date are placeholders.
- Widgets are placeholders.
- Favorite applications are placeholders.
- The application grid is a placeholder.
- Search bars are not functional yet.
- The left and right panels still need further UI refinement.

### 17.2 Window Management

The GTK shell has been tested as a desktop window.

Further verification is still needed for:

- Consistent visibility across all workspaces.
- Correct stacking above the wallpaper and below normal applications.
- Reliable return to the home screen.
- Normal application fullscreen behavior.

### 17.3 Startup Integration

The current project has Polybar startup integration.

The GTK shell still needs a carefully tested permanent startup procedure.

Picom was launched manually during testing:

```bash
picom --config /dev/null --backend xrender &
```

Its startup behavior has not yet been finalized.

### 17.4 Deferred Features

The following features should not be implemented prematurely:

- Fullscreen application rules.
- App drawer search functionality.
- Application launching from the grid.
- Brave search integration.
- Pull-down notification panel.
- Launcher button functionality.
- Advanced widgets.
- Final startup integration.

---

## 18. Next Development Checkpoint

The latest confirmed checkpoint is the visual layout with:

- Polybar at the top.
- GTK desktop shell.
- Left home screen.
- Right app drawer.
- RGBA transparency.
- Panel opacity set to `0.18`.
- Eight-pixel spacing between panels.
- Eight-pixel internal margin.

The next planned component is a real-time clock and date on the home screen.

This should be implemented independently without changing:

- Current panel geometry.
- Current transparency.
- Polybar configuration.
- bspwm global settings.

After the clock is verified, continue with widgets and other home-screen components.

---

## 19. Development Rules

To avoid repeating earlier problems:

1. Make one change at a time.
2. Create a backup before modifying important configuration files.
3. Verify each change before proceeding.
4. Stop when an error occurs.
5. Do not apply commands to the currently focused window if its identity is uncertain.
6. Use explicit window IDs when testing individual windows.
7. Keep temporary experiments separate from permanent configuration.
8. Do not prematurely implement fullscreen application behavior.
9. Preserve the intended left/right home-screen architecture.
10. Keep the project lightweight enough for the Dell Inspiron 3138.

**Current project state:** GTK desktop shell prototype is functional as a translucent, two-panel desktop background. The interface is still under development and is not yet a complete Android/ChromeOS-like desktop environment.
