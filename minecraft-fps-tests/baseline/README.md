# Minecraft FPS Test - a

This is the first test (baseline). Later tests will be in `changelog-1`, `changelog-2`, and so on.

## Setup

- Launcher: TLauncher (fresh install)
- OS: Debian Linux
- Minecraft version: 1.7.10 (profile "Hagure 1.7.10")
- Hardware: Intel Celeron N3xxx (tuned a lot), 2 GB RAM, HDD (1160 reallocated sectors)
- Shaders: Builder's Modded Shaders V2.4.1
- Mods:
  - OptiFine HD U E7
  - BetterFps 1.0.1
  - FpsReducer 1.10.3
  - MenuFPSUnlocker 1.1.0
  - fpsplus
  - UniMixins 0.3.1
  - TLSkinCape 1.4 (for skin and cape, not for FPS)

## What I did

I deleted `.minecraft` and `.tlauncher`, then installed TLauncher again.
I did not run any other cleanup commands, because I wanted to test the performance first.

## Result

"Low" means all video settings are at the lowest.
"Max" means all video settings are at the highest.

| Setting          | Old setup (from my memory) | Now            |
|------------------|----------------------------|----------------|
| Max              | 50-70 fps                  | 75-100 fps     |
| Max + shaders    | about 20 fps               | 25-30 fps      |
| Low              | 100-200 fps                | not tested yet |
| Low + shaders    | about 30 fps               | not tested yet |

Memory allocated at world creation: about 200 MB (about the same as before).

## Notes

- The old numbers are from my memory. They can be wrong.
- The old setup had more FPS mods than now.
- Results can be different on other PCs, other OS, and other setups.
- These numbers are from a new world, right after creating it.
- Minecraft 1.7.10 is old, so it is much lighter than new versions. Do not compare these numbers with new versions.
- No long-term test yet.
- Low and low + shaders will be tested later.
