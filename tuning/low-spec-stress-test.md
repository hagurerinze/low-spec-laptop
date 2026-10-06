# Stress Test on a Tuned 2 GB RAM Celeron N3xxx Laptop

A short note about a real-life test on my old laptop.

## Device

- CPU: Intel Celeron N3xxx
- RAM: 2 GB
- OS: Debian (heavily tuned: CPU, RAM, zram, swap, iGPU)
- Very old device, but tuned as much as possible

## What Happened

I was playing **Ender Lilies** (Wine + DXVK). By accident, **YT Music in Brave was opened twice**. So the laptop had a heavy game and two heavy browser tabs at the same time.

## Result

- The system **did not freeze**.
- I could still **close the apps by hand**.
- The game had **noticeable stuttering**. This is normal for this hardware.
- After I closed everything, the laptop **went back to normal**.

## After That

- I closed Ender Lilies, then **watched anime**. No problem.
- After that I **pushed my daily .md notes** to GitHub.
- **No sign of slowdown** after all these heavy tasks.

## Summary

Heavy task after heavy task, and the system stayed stable. It feels very stable, even though the device is very old.

The tuning works. The most important result: when memory is under pressure, the system **stays alive and under my control** instead of freezing.
