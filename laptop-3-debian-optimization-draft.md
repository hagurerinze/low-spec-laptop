# Laptop 3 — Low-Spec Debian Optimization

## Status

Draft — ongoing.

This document records the hardware and software optimization work performed on Laptop 3 without replacing hardware. The investigation is organized by subsystem so that each change can be measured before moving to the next one.

The current machine is an Acer Aspire E1-470 running Debian 13 Trixie with XFCE.

---

## 1. System Overview

| Component | Current configuration |
|---|---|
| Device | Acer Aspire E1-470 |
| OS | Debian 13 Trixie |
| Desktop | XFCE |
| CPU | Intel Core i3-3217U |
| CPU generation | Ivy Bridge |
| Cores / Threads | 2 / 4 |
| Base / maximum clock | 1.80 GHz / 1.80 GHz |
| TDP | 17 W |
| GPU | Intel HD Graphics 4000 |
| GPU device | `8086:0166` |
| RAM | 2 GB DDR3-1600 |
| Installed RAM modules | 1 SODIMM |
| Storage | WDC WD5000LPVX-22V0TT0 |
| Storage type | 5400 RPM SATA HDD |
| Capacity | 500 GB |
| Kernel during investigation | `6.12.107+deb13-amd64` |

The goal is not to make the machine behave like modern hardware. The goal is to remove avoidable software/configuration overhead and identify the practical limits of the existing hardware.

---

# 2. CPU Investigation

## 2.1 CPU Characteristics

The system uses an Intel Core i3-3217U:

```text
Intel Core i3-3217U
Ivy Bridge
2 cores / 4 threads
1.80 GHz maximum
17 W TDP
```

The CPU is relatively old but still capable of handling a lightweight Linux desktop and ordinary applications when memory and storage pressure are controlled.

---

## 2.2 CPU Frequency Driver

The active CPU frequency driver was:

```text
intel_cpufreq
```

The available governors were:

```text
performance
schedutil
```

The observed frequency policy was approximately:

```text
Minimum: 800 MHz
Maximum: 1.80 GHz
```

The CPU therefore has no meaningful turbo range above 1.80 GHz that should be treated as additional performance headroom.

---

## 2.3 Governor Benchmark

The initial benchmark produced:

```text
schedutil:
~1807 bogo ops/s

performance:
~1960 bogo ops/s
```

This appeared to indicate an improvement of approximately 8.5%.

However, the test was repeated under more controlled conditions.

Repeated results:

```text
schedutil:
~1927 bogo ops/s

performance:
~1930 bogo ops/s
```

Observed Mperf values were approximately:

```text
schedutil:
~1787–1793 MHz

performance:
~1794–1795 MHz
```

CPU C0 residency was approximately:

```text
schedutil:
~88–91%

performance:
~98–100%
```

### Interpretation

The original ~8.5% difference was not reproducible.

The controlled comparison showed only approximately:

```text
0.17%
```

difference between the two governors.

Therefore:

> `performance` does not provide a meaningful sustained CPU performance increase on this system under the tested workload.

Both governors allow the CPU to reach approximately its 1.8 GHz maximum.

The `performance` governor can still be considered for AC-only operation, but it should not be described as a significant performance optimization based on this benchmark.

The system was restored to:

```text
schedutil
```

because it provides dynamic frequency behavior without a measurable sustained performance disadvantage.

---

## 2.4 CPU Stress Test

The following stress test completed successfully:

```bash
stress-ng --cpu 4 --timeout 30s --metrics-brief
```

Observed temperatures were approximately:

```text
Idle/light load:
~59–60°C

CPU stress:
~65–67°C
```

No instability was observed during the test.

### CPU conclusion

The CPU is not currently showing a clear software configuration bottleneck.

The main CPU-side findings are:

- `intel_cpufreq` works correctly.
- The CPU reaches its expected maximum frequency.
- `schedutil` and `performance` produced nearly identical sustained benchmark performance.
- Stress testing completed successfully.
- Temperatures remained reasonable during the short test.

No further CPU governor modification is currently justified by the measurements.

---

# 3. Memory and ZRAM Investigation

## 3.1 Initial Memory Situation

Before the current zram configuration, the system showed approximately:

```text
RAM:
1.7 GiB total
1.5 GiB used
159 MiB free
273 MiB available

HDD swap:
~730 MiB used
```

With only 2 GB physical RAM, memory pressure is an important part of the overall system behavior.

---

## 3.2 Kernel Support

The kernel configuration showed:

```text
CONFIG_ZSWAP=y
CONFIG_ZRAM=m
```

Available zram compression backends included:

```text
LZ4
LZ4HC
ZSTD
DEFLATE
```

ZSWAP was not enabled by default.

---

## 3.3 ZRAM Configuration

The system was configured to use ZSTD compression with zram.

Runtime setup:

```bash
sudo modprobe zram
echo zstd | sudo tee /sys/block/zram0/comp_algorithm
echo 1536M | sudo tee /sys/block/zram0/disksize
sudo mkswap /dev/zram0
sudo swapon -p 100 /dev/zram0
```

The selected size is approximately 75% of the machine's physical RAM:

```text
Physical RAM: ~2 GB
ZRAM:         1.5 GB
Ratio:        ~75%
```

The HDD swap remains available as a lower-priority fallback.

---

## 3.4 Persistent ZRAM Configuration

A systemd service was created:

```ini
[Unit]
Description=ZSTD zram swap
After=systemd-modules-load.service
Before=swap.target

[Service]
Type=oneshot
ExecStart=/sbin/modprobe zram
ExecStart=/bin/sh -c 'echo zstd > /sys/block/zram0/comp_algorithm'
ExecStart=/bin/sh -c 'echo 1536M > /sys/block/zram0/disksize'
ExecStart=/sbin/mkswap /dev/zram0
ExecStart=/sbin/swapon -p 100 /dev/zram0
ExecStop=/sbin/swapoff /dev/zram0
ExecStop=/bin/sh -c 'echo 1 > /sys/block/zram0/reset'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

---

## 3.5 Post-Reboot Result

After reboot:

```text
NAME       TYPE      SIZE USED PRIO
/dev/sda2  partition 4G   0B   -2
/dev/zram0 partition 1.5G 415M 100
```

`zramctl` showed:

```text
/dev/zram0
algorithm: zstd
disk size: 1.5G
DATA:      ~398.8M
COMPR:     ~99.8M
TOTAL:     ~108.8M
STREAMS:   4
```

This corresponds to approximately a 4:1 compression ratio for the observed compressed data.

Memory state after reboot:

```text
RAM:
1.7 GiB total
1.3 GiB used
160 MiB free
~467 MiB available

Swap:
5.5 GiB total
~414 MiB used
```

The important observation is that the HDD swap remained unused while zram handled the active compressed swap workload.

---

## 3.6 RAM Usage Audit

A process-level memory audit showed Firefox dominating user-space RAM usage.

Representative processes:

```text
Firefox Isolated Web Content    ~473 MB RSS
firefox-esr                     ~188 MB RSS
Firefox Privileged Content       ~87 MB RSS
Firefox WebExtensions            ~57 MB RSS
```

Other major desktop components were substantially smaller:

```text
Xorg                    ~33 MB
xfwm4                   ~29 MB
xfce4-panel             ~29 MB
xfdesktop               ~27 MB
xfce4-terminal          ~23 MB
NetworkManager applet   ~23 MB
```

The Firefox processes together consumed a large portion of physical memory.

This means the desktop itself is not the only significant memory consumer; browser workload is a major factor.

---

## 3.7 GPU Memory Reservation

Kernel messages showed an Intel graphics memory reservation of approximately:

```text
64 MiB
```

The kernel reported:

```text
Reserving Intel graphics memory
[mem 0x7aa00000-0x7e9fffff]
```

The system also reported approximately:

```text
Memory available:
1616572K / 1976500K
```

The GPU reservation alone is not sufficient to explain the overall memory pressure.

Other memory consumers include:

- kernel/firmware reservations
- XFCE and desktop services
- applications
- shared memory
- filesystem cache
- browser processes

---

## 3.8 Memory Pressure

`systemd-journald` reported:

```text
Under memory pressure, flushing caches
```

This occurred during the investigation.

The observation supports the conclusion that the 2 GB physical RAM configuration is a significant system constraint.

---

## 3.9 Memory Conclusion

The current memory strategy is:

```text
Physical RAM
    ↓
ZSTD ZRAM 1.5 GB
    ↓
HDD swap 4 GB fallback
```

The current 75% zram configuration is retained as the working configuration.

A larger zram allocation, such as approximately 85% of physical RAM, remains a possible future experiment but has not been adopted as the baseline.

The desktop environment itself also remains a potential optimization target because the machine has only 2 GB RAM.

---

# 4. GPU Investigation

## 4.1 GPU Identification

The system uses:

```text
Intel HD Graphics 4000
Ivy Bridge
PCI ID: 8086:0166
```

The kernel identified the GPU as:

```text
IVYBRIDGE
display version 7.00
```

The i915 driver initialized successfully:

```text
Initialized i915 1.6.0
```

---

## 4.2 Kernel / DRM Status

Relevant kernel messages showed:

```text
i915 0000:00:02.0: [drm] Found IVYBRIDGE
[drm] Initialized i915 1.6.0
fbcon: i915drmfb
```

There were firmware-related messages such as:

```text
EFI: Firmware Bug: Invalid EFI memory map entries
ACPI: Firmware Bug: BIOS _OSI(Linux) query ignored
```

but no obvious i915/DRM failure was observed.

The GPU driver was functioning normally.

---

## 4.3 i915 Parameters

After reading the parameters with appropriate privileges, the relevant values included:

```text
enable_fbc            = -1
enable_ips            = Y
enable_psr            = -1
enable_psr2_sel_fetch = Y
enable_dc             = -1
enable_dpcd_backlight = -1
enable_dmc_wl         = N
enable_guc            = -1
enable_hangcheck      = Y
enable_sagv           = Y
reset                 = 3
mitigations           = auto
```

No broad i915 tuning was applied.

In particular, settings such as GuC, PSR, power management, and GT clocks were not forcibly changed without evidence that they were causing a problem.

---

# 5. Framebuffer Compression (FBC)

## 5.1 Initial State

Initially:

```text
enable_fbc = -1
```

The i915 FBC debug status showed:

```text
FBC disabled: disabled per module param or by default
```

---

## 5.2 FBC Test

A temporary GRUB boot parameter was tested:

```text
i915.enable_fbc=1
```

The test boot showed:

```text
enable_fbc = 1
```

FBC became available/enabled during the test.

No visual glitches were observed during normal browser use.

---

## 5.3 Runtime FBC Behavior

Later, the debug status showed:

```text
FBC disabled: framebuffer not fenced

[PLANE:32:primary A]: FBC possible
[PLANE:48:primary B]: plane not visible
[PLANE:64:primary C]: plane not visible
```

XFCE compositor state was checked.

The compositor was temporarily disabled and FBC status was checked again. The status remained:

```text
framebuffer not fenced
```

The compositor was restored afterward.

Therefore, the XFCE compositor was not identified as the primary reason for the runtime FBC limitation.

---

## 5.4 Permanent FBC Configuration

Because the forced FBC configuration produced no visible stability problems, the parameter was made persistent through GRUB:

```text
i915.enable_fbc=1
```

The system was subsequently rebooted.

Important distinction:

> `i915.enable_fbc=1` forces FBC to be enabled as a driver configuration option, but runtime FBC can still be unavailable for a particular framebuffer when fencing or other runtime conditions prevent it.

The configuration is therefore retained, but runtime debug output should not be interpreted as a guarantee that every frame is compressed.

---

# 6. GPU Frequency Scaling

The GPU frequency interface showed:

```text
Current: ~350 MHz
Minimum: 350 MHz
Maximum: 1050 MHz
```

At idle, the GPU remained around:

```text
350 MHz
```

During a `glxgears` workload, the GPU frequency increased.

This confirms that dynamic GPU frequency scaling is functioning.

No fixed GPU clock was applied.

Forcing the GPU to a higher clock would increase power/thermal load without evidence of a corresponding real-world benefit.

---

# 7. OpenGL Acceleration

`glxinfo -B` reported:

```text
direct rendering: Yes
Vendor: Intel
Device: Mesa Intel(R) HD Graphics 4000 (IVB GT2)
Version: Mesa 25.0.7
Accelerated: yes
Video memory: 1536MB
Unified memory: yes
```

Renderer:

```text
Mesa Intel(R) HD Graphics 4000 (IVB GT2)
```

Supported OpenGL levels included:

```text
OpenGL core profile 4.2
OpenGL compatibility profile 4.2
OpenGL ES 3.0
```

This confirms hardware-accelerated rendering.

The reported 1536 MB video memory should not be interpreted as dedicated physical VRAM. The HD 4000 uses shared/unified system memory.

Most importantly:

```text
Accelerated: yes
```

and the renderer is the Intel GPU rather than software rendering such as `llvmpipe`.

---

# 8. VA-API / Hardware Video Acceleration

The system initially did not have `vainfo` installed.

Relevant VA packages after installation included:

```text
i965-va-driver:amd64              2.4.1+dfsg1-2
intel-media-va-driver:amd64       25.2.3+dfsg1-1
libva-drm2                         2.22.0-3
libva-wayland2                     2.22.0-3
libva-x11-2                        2.22.0-3
libva2                              2.22.0-3
mesa-va-drivers                    25.0.7-2+deb13u1
```

The DRM devices were:

```text
/dev/dri/card0
/dev/dri/renderD128
```

Using the legacy Ivy Bridge VA driver:

```bash
LIBVA_DRIVER_NAME=i965 vainfo
```

returned:

```text
VA-API version: 1.22
Driver version:
Intel i965 driver for Intel(R) Ivybridge Mobile - 2.4.1
```

Supported profiles included hardware video acceleration for:

```text
MPEG-2
H.264
VC-1
JPEG
Video processing
```

H.264 decode/encode profiles were also exposed.

Therefore, VA-API hardware acceleration is working.

---

# 9. GPU Conclusion

The GPU investigation found:

- i915 is correctly loaded.
- Direct rendering is enabled.
- Mesa hardware acceleration is active.
- HD 4000 frequency scaling works.
- VA-API works through the i965 driver.
- No major i915/DRM failure was found.
- FBC was tested and forced on through GRUB.
- No visual instability was observed.
- No evidence currently justifies forcing additional GPU parameters.

The GPU software configuration is therefore considered sufficiently functional for the current optimization stage.

---

# 10. HDD Investigation

## 10.1 Drive Identification

The system uses:

```text
Model:
WDC WD5000LPVX-22V0TT0

Family:
Western Digital Blue Mobile

Capacity:
500 GB

Rotation:
5400 RPM

Interface:
SATA 3.0, 6.0 Gb/s
```

Linux reports the disk as:

```text
/dev/sda
```

Partition layout:

```text
/dev/sda1   512M     EFI
/dev/sda2   4G       swap
/dev/sda3   60G      /
/dev/sda4   401.3G  /home
```

---

# 11. HDD SMART Health

SMART overall health:

```text
PASSED
```

Relevant attributes:

```text
Reallocated sectors:       0
Current pending sectors:   0
Offline uncorrectable:     0
UDMA CRC errors:           0
Error log:                 No Errors Logged
Temperature:               36°C
Power-on hours:            6385
Start/stop count:          39567
Load cycle count:          145906
G-sense errors:            430
```

The available SMART data does not show an active bad-sector or interface-error problem.

The drive is old and has substantial accumulated mechanical activity, but there is no current SMART evidence of imminent failure from the tested attributes.

---

# 12. HDD I/O Configuration

Current scheduler:

```text
none [mq-deadline]
```

The active scheduler is:

```text
mq-deadline
```

Current queue settings:

```text
read_ahead_kb = 128
nr_requests   = 64
```

The drive is correctly identified as rotational storage:

```text
ROTA = 1
```

No queue parameter has been changed yet.

---

# 13. Sequential HDD Benchmark

Using `hdparm`:

```bash
sudo hdparm -Tt /dev/sda
```

Observed buffered disk reads:

```text
102.39 MB/s
103.21 MB/s
106.57 MB/s
106.66 MB/s
```

Average:

```text
~104.7 MB/s
```

The cached read figures were approximately 3.5–4.8 GB/s.

Those cached figures primarily represent memory/cache behavior and should not be treated as physical HDD throughput.

The approximately 105 MB/s buffered read result is the useful sequential-read measurement.

---

# 14. Initial Random Read Benchmark

The first random-read test created a 512 MiB file using:

```bash
fallocate -l 512M ~/hdd-fio-test
```

The subsequent `fio` benchmark reported:

```text
~200k IOPS
~782 MiB/s
~4.1 µs average latency
~3.75% disk utilization
```

These values were rejected as a representative HDD benchmark.

The numbers are not physically plausible for a 5400 RPM mechanical HDD and did not show the expected mechanical-disk behavior.

The test was therefore repeated after explicitly writing actual data to the test file.

---

# 15. Valid Random Read Benchmark

The test file was populated with:

```bash
dd if=/dev/zero of=~/hdd-fio-test \
    bs=1M count=512 \
    status=progress \
    conv=fsync
```

The observed write result was:

```text
512 MiB
~83.9 MB/s
```

The populated file was then tested with:

```bash
fio --name=hdd-randread \
    --filename="$HOME/hdd-fio-test" \
    --size=512M \
    --rw=randread \
    --bs=4k \
    --ioengine=libaio \
    --direct=1 \
    --iodepth=1 \
    --runtime=30 \
    --time_based \
    --group_reporting
```

Results:

```text
Random 4K read:
~120 IOPS

Bandwidth:
~483 KiB/s

Average latency:
~8.24 ms

Disk utilization:
~97.54%

Queue depth:
1
```

Latency distribution:

```text
50th percentile:    ~7.90 ms
90th percentile:    ~12.13 ms
95th percentile:    ~12.91 ms
99th percentile:    ~32.38 ms
99.9th percentile:  ~79.17 ms
Maximum:            ~93.0 ms
```

The benchmark completed without I/O errors.

The test file was removed afterward:

```bash
rm ~/hdd-fio-test
```

---

# 16. HDD Benchmark Summary

Current measured storage behavior:

| Workload | Result |
|---|---:|
| Sequential read | ~104.7 MB/s |
| Sequential write | ~83.9 MB/s |
| Random 4K read | ~120 IOPS |
| Random 4K bandwidth | ~483 KiB/s |
| Random 4K average latency | ~8.24 ms |
| Random 4K disk utilization | ~97.54% |

The random-read result is consistent with the expected limitations of a 5400 RPM mechanical HDD.

The important distinction is:

```text
Sequential I/O
    relatively high throughput

Random 4K I/O
    low IOPS
    millisecond latency
    very high disk utilization
```

This explains why desktop responsiveness can remain limited even when sequential disk throughput looks reasonable.

---

# 17. HDD Assessment

The current evidence does not show an obvious HDD software configuration failure.

The major limitation is mechanical random-access latency.

The HDD is capable of approximately 100 MB/s sequential transfer, but small random operations require mechanical seeking and rotational latency.

This is especially relevant for:

- application startup
- package installation
- filesystem metadata operations
- browser cache activity
- loading many small files
- multitasking while memory pressure causes additional I/O
- desktop workloads involving frequent small reads/writes

The existing `mq-deadline` scheduler is therefore being retained as the baseline until an A/B test provides evidence for a change.

---

# 18. Current Optimization State

The current configuration can be summarized as:

```text
CPU
├─ Driver: intel_cpufreq
├─ Governor: schedutil
└─ Range: 800 MHz – 1.80 GHz

Memory
├─ Physical RAM: ~2 GB
├─ ZRAM: 1.5 GB
├─ Compression: ZSTD
├─ ZRAM priority: 100
└─ HDD swap priority: -2

GPU
├─ Intel HD Graphics 4000
├─ i915
├─ Mesa hardware acceleration: yes
├─ VA-API: working
├─ Dynamic GPU frequency: working
└─ FBC parameter: i915.enable_fbc=1

HDD
├─ WDC WD5000LPVX
├─ 5400 RPM
├─ Scheduler: mq-deadline
├─ read_ahead_kb: 128
└─ nr_requests: 64
```

---

# 19. Findings So Far

## CPU

The CPU is functioning close to its expected limit.

Changing from `schedutil` to `performance` did not produce a meaningful sustained improvement in the controlled benchmark.

Current status:

```text
No further CPU governor change justified.
```

## Memory

2 GB RAM is a major constraint.

ZSTD zram provides a useful compressed-memory layer and keeps the HDD swap as a lower-priority fallback.

Current status:

```text
ZSTD zram 1.5 GB retained.
```

## Desktop

XFCE itself is not the only memory consumer, but the machine has enough memory pressure that a lighter desktop environment remains a logical future optimization target.

Firefox can consume several hundred MiB by itself, making application workload an important part of memory pressure.

Current status:

```text
Desktop optimization not yet performed.
```

## GPU

The HD 4000 has functional hardware acceleration.

Current status:

```text
i915: working
OpenGL: hardware accelerated
VA-API: working
Dynamic frequency scaling: working
FBC: forced parameter enabled
```

No additional aggressive i915 tuning is currently justified.

## HDD

The HDD is healthy according to the available SMART data and performs normally for a 5400 RPM laptop HDD in sequential workloads.

Its main limitation is random I/O latency.

Current status:

```text
No HDD queue changes yet.
```

---

# 20. Optimization Philosophy

The investigation uses a measurement-first approach.

```text
Observe
   ↓
Measure
   ↓
Identify the bottleneck
   ↓
Change one variable
   ↓
Measure again
   ↓
Keep the change only if it produces a useful result
```

The purpose is not to maximize synthetic benchmark numbers.

The purpose is to improve actual usability on a low-spec machine while avoiding unnecessary tuning that increases complexity, heat, instability, or power consumption without measurable benefit.

---

# 21. Next Steps

The next HDD experiment should be an A/B comparison between:

```text
Current:
mq-deadline
```

and:

```text
Alternative:
none
```

The same random-read workload should be used for both tests.

The comparison should focus on:

```text
IOPS
average latency
90th / 95th / 99th percentile latency
disk utilization
```

The scheduler should only be changed permanently if the measured difference is meaningful for the intended workload.

Other possible future stages:

1. Desktop environment comparison.
2. HDD scheduler A/B test.
3. `read_ahead_kb` experiment if a workload-specific reason exists.
4. `nr_requests` experiment only if evidence supports it.
5. Further memory-pressure testing.
6. Real-world application startup measurements.
7. End-to-end comparison after all selected changes.

---

# 22. Overall Draft Conclusion

Laptop 3 is constrained primarily by its combination of:

```text
2 GB RAM
+
5400 RPM mechanical HDD
+
older integrated GPU
```

The CPU itself is not currently the dominant problem.

The most important findings so far are:

- CPU governor changes do not provide a meaningful sustained performance increase.
- ZSTD zram is useful for the 2 GB memory configuration.
- Firefox can consume a substantial fraction of available RAM.
- Intel HD 4000 hardware acceleration is functioning.
- VA-API hardware video acceleration is functioning.
- GPU frequency scaling is functioning.
- FBC was successfully tested and enabled through the kernel parameter.
- The HDD has no obvious SMART health failure indicators.
- Sequential HDD throughput is approximately 105 MB/s read and 84 MB/s write.
- Random 4K HDD performance is approximately 120 IOPS with ~8.24 ms average latency.
- The HDD's mechanical random-access behavior is a major source of storage-side latency.
- No HDD scheduler or queue parameter has yet been changed without measurement.

The project remains focused on extracting practical performance improvements from the existing hardware rather than masking hardware limitations with unsupported or arbitrary kernel tuning.
