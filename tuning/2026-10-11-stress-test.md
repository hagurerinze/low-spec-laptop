# Stress Test Notes: 2GB RAM Laptop (Celeron N3xxx, HDD)

These are my notes from a hard stress test on my old laptop.
I want to be honest. I write what I know, what I am not sure about, and what I did wrong.
This is not about Windows being bad. It is only my own experience on my own laptops.

**This test is my main baseline (main reference) for the next tests.**

## My laptop

- CPU: Intel Celeron N3xxx
- RAM: 2GB (about 1.8 GB usable)
- Storage: HDD
- OS: Debian with XFCE
- zram: 1.3G, zstd, priority 100
- HDD swap: `/dev/sda2`, 4G, priority -2

## What was running

This test was not planned. I did not expect the laptop to survive it.

- Photopea, YouTube Music, and Claude. All three were Brave PWAs.
- I switched between Brave windows many times.
- Bluetooth was on (I used TWS earbuds).
- I turned the live wallpaper on and off a few times.
- My WhatsApp bot was running.
- I also played Ender Lilies on this day, but I forgot the exact time.

The laptop barely stuttered. The body of the laptop did not feel hot.

## What I saw in Task Manager (not exact)

I checked Task Manager before I closed YouTube Music. I do not remember the exact numbers:

- Physical RAM: about 1.3 / 1.8 GB
- Swap: about 1.3 to 1.7 GB out of 5.3 GB
- zram peak: I think about 1.5 / 1.8, but I forgot

These numbers are from my memory. Please treat them as a guess.

## Real numbers (taken after I closed YouTube Music)

So these are not the numbers at the heaviest moment.

### zramctl

| Name | Size |
|---|---|
| DISKSIZE | 1.3G |
| DATA (before compress) | 898.9M |
| COMPR (after compress) | 264M |
| TOTAL (real RAM used) | 270.8M |

### mm_stat

- Data: 901 MB
- Compressed: 265 MB
- Used: 272 MB
- **Peak used: 344 MB** (this is the peak real RAM that zram used since boot)
- Compression ratio: about 3.4 to 1

### swapon --show

| Device | Size | Used | Priority |
|---|---|---|---|
| /dev/sda2 (HDD) | 4G | 71.2M | -2 |
| /dev/zram0 | 1.3G | 955.9M | 100 |

### free -h

- RAM used: 1.5Gi, free: 96Mi, available: 300Mi
- Swap used: 1.0Gi of 5.3Gi

### vmstat

- `so` (swap out) was 0 in almost all rows. No new writing to swap at that time.
- `si` (swap in) was 443 and 132 in two rows.
- `wa` (IO wait) was 0 to 1 percent. The HDD was not the problem.
- The first row of vmstat is the average since boot, so I ignore its `wa 54`.

### Temperature (sensors)

- Package: 45°C
- Core 0: 42°C
- Core 2: 44°C
- ACPI: 42°C

This was also after YouTube Music was closed. So it is a temperature after heavy use, not at the peak.

## What I learned about zram

- zram does not take 300 MB first. It starts near 0 and grows when more data goes in.
- 1.3G is only the limit of the original data. It is not reserved RAM.
- When zram is full at 1.3G, it uses only about 300 to 400 MB of real RAM (with a 3.4 to 1 ratio).
- The data still uses RAM. It is just much smaller. This is real saving, not an illusion.
- The cost is CPU time for compress and decompress.
- If the data is hard to compress, zram will use more RAM.

## A correction (my mistake and the AI's mistake)

At first I thought the HDD swap was not used at all.
The real result is that the HDD was used a little (71.2M).
So the correct sentence is: **"The HDD was almost not used"**, not "not used at all".

Also, Linux does not save the peak of total swap or the peak per device.
It only shows the current state. Only the zram real RAM peak (`mem_used_max`) is saved.
So the peak of HDD swap was not recorded.

## My claims (these are possibilities, not measured)

- With YouTube Music running, the temperature is probably not far from now. Maybe 50°C at most. This is only my guess. I did not measure it.
- Earlier, the swap on the HDD may have reached some amount between 0 and 0.4G. This is only an estimate.

## My experience with Windows on this laptop

- This laptop used Windows 10 Home Single Language, and later Windows 10 Pro.
- On both, the laptop got hot easily, even when I was not doing much.
- This laptop got the HDD "1160" problem while I used Windows. At that time the laptop had been quite hot.
- I also used the laptop on a bed for a few days. The battery went down after that. This may also be a cause. Heat from the bed and from Windows can be mixed. I cannot prove one single cause.
- I did not see any I/O error.
- Back then I did not know SentinelDisk, so I never checked the disk health. The starting condition of the disk was not recorded.

My opinion: I am not saying Windows is bad. I think old laptops last longer if they move away from a Windows-centered setup. This is from one laptop only. It is not a rule for all laptops.

## My other laptops

- N2K, 4GB RAM
- Intel i3 gen 3, 2GB RAM

These two laptops are a bit hotter. I have never used them on Windows, so the comparison is fair (same Linux side).

## Fair notes

- Modern laptops have faster CPUs and are allowed to go up to 80-90°C when working. So a high temperature there is normal.
- My laptop is cool partly because the Celeron N3xxx is a low-power chip (about 6W). So I am not saying my laptop is more efficient than a modern laptop.
- My point is only this: for an old laptop that can still be pushed hard like this, the OS makes a big difference.

## Next test

The next test may be sudden, so I may not be ready. But I want to try these:

- Check temperature while the apps are running:

```bash
watch -n 2 sensors
```

- Log the swap every 5 seconds to find the HDD swap peak:

```bash
while true; do echo "$(date +%T) $(swapon --noheadings --show=NAME,USED | tr '\n' ' ')" >> ~/swaplog.txt; sleep 5; done
```

- Check zram peak before reboot (it is lost after reboot):

```bash
cat /sys/block/zram0/mm_stat
```

- I will write the RAM and CPU numbers too.
- Take a screenshot at the heaviest moment.
