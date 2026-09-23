# XFWM4 Desktop Zoom — Fix

## The problem
On Debian XFCE, the desktop suddenly appeared zoomed in, as if a magnifier feature had been triggered.

Checking XFWM4's configuration revealed two relevant keys:
```text
/general/zoom_desktop
/general/zoom_pointer
```

`zoom_desktop` controls XFWM4's built-in Desktop Zoom feature.
`zoom_pointer` controls whether the zoom effect follows the pointer position.

## Investigation
Command used to search for related settings:
```bash
xfconf-query -c xfwm4 -lv | grep -Ei 'zoom|magnif'
```

This confirmed both keys exist:
```text
/general/zoom_desktop
/general/zoom_pointer
```

Note: the correct XFWM4 channel name is:
```text
xfwm4
```
not:
```text
xfcewm4
```

## Suspected trigger
XFWM4's Desktop Zoom can be triggered with:
```text
Alt + Scroll Up   -> Zoom in
Alt + Scroll Down -> Zoom out
```

On a laptop touchpad, an accidental scroll/gesture while a modifier key like `Alt` happens to be held down can trigger the zoom unintentionally.

## Checking the current status
```bash
xfconf-query -c xfwm4 -p /general/zoom_desktop
```

If the result is:
```text
true
```
Desktop Zoom is enabled.

If:
```text
false
```
it's disabled.

## Preventing accidental zoom
Disable Desktop Zoom:
```bash
xfconf-query -c xfwm4 -p /general/zoom_desktop -s false
```

Then verify:
```bash
xfconf-query -c xfwm4 -p /general/zoom_desktop
```
Target result:
```text
false
```

No need to change `/general/zoom_pointer` — that key only affects pointer behavior while zoom is already active.

## If the desktop still looks zoomed
Restart XFWM4:
```bash
xfwm4 --replace &
```

## Conclusion
The issue was not a display resolution change. XFWM4 has a built-in Desktop Zoom feature that can be triggered accidentally by a modifier + scroll combination — especially easy to trigger on a touchpad.

To avoid this happening again, the safest configuration is:
```bash
xfconf-query -c xfwm4 -p /general/zoom_desktop -s false
```
