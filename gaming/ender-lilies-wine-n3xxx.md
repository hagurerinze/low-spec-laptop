# Running Ender Lilies on a 2GB Celeron N3xxx (Wine + DXVK)

Notes from getting *ENDER LILIES: Quietus of the Knights* (Windows build) running on a low-spec Debian laptop. Includes the mistakes made along the way, because they are the useful part.

## Setup

- CPU: Intel Celeron N3xxx, RAM: 2GB
- OS: Debian 13 (trixie) with XFCE
- GPU: Intel HD Graphics 500 (Apollo Lake), Mesa 25.0.7, Vulkan works
- Wine: 10.0 (Debian repack), prefix at `~/.wine-ender`
- Game folder: `~/ENDER LILIES Quietus of the Knights`
- Game engine: Unreal Engine 4 (folders `Engine` and `EnderLilies`, process `EnderLiliesSteam-Win64-Shipping.exe`)

## Final result

- Plays well and is stable. Reached the horse checkpoint without crashes.
- Resolution kept at 960x540 (never raised).
- FPS: roughly 20s to 30 fps in normal play, under 20 fps during boss fights.
- In-game gamma and resolution scale (exact menu name not remembered) are both set to 75. It does not look bad and stays playable.
- Final working combination: DXVK 2.5.3, graphics pipeline library disabled, Intel iGPU forced, DXVK HUD on.

## What was done

### 1. Prefix and dependencies

```bash
sudo apt install cabextract vulkan-tools mesa-vulkan-drivers
wget https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
chmod +x winetricks
sudo mv winetricks /usr/local/bin/

export WINEPREFIX=~/.wine-ender
export WINEARCH=win64
wineboot -i
winetricks -q vcrun2019 dxvk
```

Check that Vulkan sees the iGPU with `vulkaninfo --summary`. The device list also shows `llvmpipe` (CPU renderer), which must not be used.

### 2. DXVK version

`winetricks dxvk` installed DXVK 3.1.1, which gave `VK_ERROR_DEVICE_LOST`. DXVK 2.5.3 was copied over it manually:

```bash
export WINEPREFIX=~/.wine-ender
cd /tmp
wget https://github.com/doitsujin/dxvk/releases/download/v2.5.3/dxvk-2.5.3.tar.gz
tar xf dxvk-2.5.3.tar.gz
cp dxvk-2.5.3/x64/*.dll $WINEPREFIX/drive_c/windows/system32/
cp dxvk-2.5.3/x32/*.dll $WINEPREFIX/drive_c/windows/syswow64/
```

Do not run `winetricks dxvk` again, it would overwrite 2.5.3 with the newer version.

### 3. DXVK config

```bash
echo "dxvk.enableGraphicsPipelineLibrary = False" > ~/dxvk.conf
```

### 4. Launcher script `~/ender.sh`

```bash
#!/bin/bash
export WINEPREFIX=~/.wine-ender
export DXVK_CONFIG_FILE=~/dxvk.conf
export DXVK_FILTER_DEVICE_NAME="Intel"
export DXVK_HUD=fps
cd ~/'ENDER LILIES Quietus of the Knights'
wine EnderLilies.exe -windowed -ResX=960 -ResY=540 -dx11
```

`DXVK_HUD=fps` is kept on purpose, see the mistakes section.

### 5. Menu and desktop entry

`~/.local/share/applications/enderlilies.desktop` (copied to `~/Desktop/` as well, marked executable):

```ini
[Desktop Entry]
Type=Application
Name=Ender Lilies
Comment=Ender Lilies: Quietus of the Knights (Wine)
Exec=/home/hagure/ender.sh
Path=/home/hagure
Icon=/home/hagure/.local/share/icons/enderlilies.png
Terminal=false
Categories=Game;
```

The icon was extracted from the Shipping exe:

```bash
wrestool -x -t 14 ~/'ENDER LILIES Quietus of the Knights'/EnderLilies/Binaries/Win64/EnderLiliesSteam-Win64-Shipping.exe -o ~/.local/share/icons/enderlilies.ico
icotool -x --index=1 -o ~/.local/share/icons/enderlilies.png ~/.local/share/icons/enderlilies.ico
```

The leftover folder `~/.local/share/icons/ender-extract` was kept for now (not deleted).

## Mistakes and lessons

1. **Wrong engine assumed.** The game was first assumed to be Unity. It is Unreal Engine 4, which is heavier. Check the folder layout (`Engine/`) before guessing.
2. **Wine Staging was planned, but it was never installed.** `wine --version` prints `wine-10.0 (Debian 10.0~repack-6)`, which is the plain Debian package, not WineHQ Staging. The game runs fine on it, so Staging is not required here. Check with `wine --version` before assuming which build is in use.
3. **`winetricks` was not installed.** The first run only created the prefix and printed `command not found`. The `err:ole`, `rpcss` and `setupapi` lines during first prefix creation are normal noise.
4. **DXVK 3.1.1 was too new for this iGPU.** It crashed with `VK_ERROR_DEVICE_LOST`. DXVK 2.5.3 works.
5. **WineD3D (fallback) has a visual bug.** It ran, but the background was drawn over the character (`GL_INVALID_ENUM in glTexBufferRange`). Not usable.
6. **Wrong theory: pipeline library as the cause.** Disabling `dxvk.enableGraphicsPipelineLibrary` was tried first as the fix. It did not stop the crash by itself.
7. **Launcher differed from the run that worked.** The first `ender.sh` left out `DXVK_HUD=fps` and crashed with `DEVICE_LOST`. Running with `DXVK_HUD=fps` worked again. The likely reason is that the HUD changes render timing slightly and avoids a hang in the Intel driver. This is a workaround, not a proven fix, so if `DEVICE_LOST` appears again: `wineserver -k` and relaunch.
8. **Two changes at once.** Raising the resolution to 1280x720 and changing the launcher in the same step made the cause unclear. Change one thing per test.
9. **Icon extraction hiccups.** The icons folder was not created first, and `wrestool` printed `mismatch of size` warnings for `EnderLilies.exe`. Extracting from the Shipping exe gave a valid `.ico`. The other extracted file (about 345 KB) is broken and can be ignored.

## Troubleshooting quick list

- Kill stuck Wine processes: `WINEPREFIX=~/.wine-ender wineserver -k`
- Check that DXVK is used: look for `DXVK: v2.5.3` in the log. Check that the config is read: `Graphics pipeline libraries not supported`.
- Filter the log: `~/ender.sh 2>&1 | grep -E "err:|DXVK:|DEVICE_LOST"`
- Watch memory while playing: `watch -n2 free -h`
- Shader compilation stutter is worst in the first minutes, and the DXVK state cache makes it better over time.

## Draft: trying Wine Staging (untested)

Status: **idea only, not tested yet.** The game already runs on plain Debian Wine 10.0 (`wine --version` prints `wine-10.0 (Debian 10.0~repack-6)`), so this is only to compare.

Wine Staging is regular Wine plus experimental patches that are not upstream yet. It aims at compatibility, not speed, so no FPS gain is guaranteed and some games get regressions. On this laptop the main limits are the Intel HD 500 iGPU, 2GB RAM and the DXVK version, none of which Staging changes.

### Plan

1. Back up the working prefix first:

```bash
   cp -r ~/.wine-ender ~/.wine-ender.bak
```

2. Install Staging from the WineHQ repo (check the current steps on the WineHQ Debian download page before running, the repo files can change):

```bash
   sudo dpkg --add-architecture i386
   sudo mkdir -pm755 /etc/apt/keyrings
   sudo wget -O /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key
   sudo wget -NP /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/debian/dists/trixie/winehq-trixie.sources
   sudo apt update
   sudo apt install --install-recommends winehq-staging
```

3. Confirm the version, then test with the same launcher (`~/ender.sh`) and the same resolution (960x540):

```bash
   wine --version
```

4. Compare FPS (DXVK HUD) in the same area and in the same boss fight as before: roughly 20s to 30 fps normal, under 20 fps in boss fights with plain Wine.

### Notes and risks

- Installing from the WineHQ repo replaces the Debian Wine package, and the prefix may get updated by the new version. That is why the backup comes first.
- If it crashes or is not faster, restore the prefix backup and go back to the Debian package.
- Keep DXVK at 2.5.3, do not run `winetricks dxvk` again.
- Result: _to be filled in after testing._

If the game crashes on bspwm, see [GPU hang fix](ender-lilies-bspwm-gpu-hang.md).

See also: [Ender Lilies: GPU hang on bspwm](./ender-lilies-bspwm-gpu-hang.md), which explains the `VK_ERROR_DEVICE_LOST` problem in more detail.
