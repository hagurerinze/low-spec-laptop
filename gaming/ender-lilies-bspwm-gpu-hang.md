# Ender Lilies: GPU hang on bspwm (works on XFCE)

Fix for `VK_ERROR_DEVICE_LOST` when running Ender Lilies with Wine + DXVK on bspwm.

## My setup

- Laptop: 2GB RAM, Celeron N3xxx, Intel HD Graphics 500 (Apollo Lake)
- OS: Debian 13
- Wine: Debian Wine 10.0
- DXVK: 2.5.3
- Window managers: XFCE (works) and bspwm (crashed)

## The problem

On XFCE the game runs fine. On bspwm the game closes or freezes, even after `wineserver -k`.

The DXVK log shows:

```
err:   DxvkSubmissionQueue: Command submission failed: VK_ERROR_DEVICE_LOST
```

`sudo dmesg | tail -30` shows a real GPU hang:

```
i915 0000:00:02.0: [drm] Resetting rcs0 for preemption time out
i915 0000:00:02.0: [drm] EnderLiliesStea[...] context reset due to GPU hang
i915 0000:00:02.0: [drm] GPU HANG: ecode 9:1:8ed9fff2, in EnderLiliesStea
```

So it is not a Wine problem. The Intel GPU froze and the kernel reset it.

## Cause (my best guess)

The GPU is too busy. The game uses the GPU a lot, and picom, conky, and tint2 also use it. When the GPU is late, the kernel thinks it is frozen (`preemption time out`) and resets it.

Test results:

| Test | Result |
|---|---|
| Wine virtual desktop (960x540) | Still crashed |
| Set 960x540 in `GameUserSettings.ini` | Still crashed |
| Turn off DXVK pipeline libraries | Still crashed |
| Run the game alone (picom, conky, tint2 closed) + `MESA_VK_WSI_PRESENT_MODE=fifo` | Works |
| Game with picom, conky, tint2 on (without the fix) | Crashed |
| Raise `preempt_timeout_ms` to 2000 | Works, even with all three on |

## The fix

### 1. Give the GPU more time before reset

Check that the file exists:

```
ls /sys/class/drm/card0/engine/rcs0/preempt_timeout_ms
```

Set it to 2000 (not saved after reboot):

```
echo 2000 | sudo tee /sys/class/drm/card0/engine/rcs0/preempt_timeout_ms
```

Make it permanent:

```
echo 'w /sys/class/drm/card0/engine/rcs0/preempt_timeout_ms - - - - 2000' | sudo tee /etc/tmpfiles.d/i915-preempt.conf
sudo systemd-tmpfiles --create /etc/tmpfiles.d/i915-preempt.conf
cat /sys/class/drm/card0/engine/rcs0/preempt_timeout_ms
```

It should print `2000`.

Note: your GPU may be `card1` and not `card0`. Check with `ls /sys/class/drm/`.

### 2. Use vsync (fifo) for extra safety

```
export MESA_VK_WSI_PRESENT_MODE=fifo
```

By default the game uses `PRESENT_MODE_IMMEDIATE`, so it uses the GPU all the time. With `fifo` (normal vsync), the game waits for the screen refresh, and the GPU gets free time. The game is limited to about 60 FPS, which is fine for Ender Lilies.

## Launcher script

Always start the game with window options. Without them, the game starts in fullscreen at your screen size (1366x717), which is heavy for this GPU.

Set `WINEPREFIX` **before** `wineserver -k`. If not, it kills the wrong prefix.

```bash
#!/bin/bash
export WINEPREFIX=~/.wine-ender
wineserver -k
sleep 1
export MESA_VK_WSI_PRESENT_MODE=fifo   # remove this line on XFCE if you want
export DXVK_CONFIG_FILE=~/dxvk.conf
export DXVK_FILTER_DEVICE_NAME="Intel"
export DXVK_HUD=fps
cd ~/'ENDER LILIES Quietus of the Knights'
wine EnderLilies.exe -windowed -ResX=960 -ResY=540 -dx11
```

What the game options do:

- `-windowed`: use a window, not fullscreen
- `-ResX=960 -ResY=540`: window size
- `-dx11`: use DirectX 11 (DXVK)

## Notes

- Without a window option, the game asks for fullscreen and bspwm accepts it. A bspwm rule does not change a fullscreen window.
- If you use the game window in bspwm and it gets tiled, add a floating rule to `bspwmrc`:
  ```
  bspc rule -a "*:*:Ender*" state=floating
  ```
  Use `xprop WM_CLASS` to find the real class name.
- `wineserver -k` does nothing if no Wine server is running, so it is safe to run every time.
