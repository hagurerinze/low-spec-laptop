# Prism Launcher --- Minecraft 1.7.10 Project

## Status

**Project status:** Draft / locked after failed launch attempt

Target: - Minecraft **1.7.10** - **Offline** play; no Microsoft account
intended - Linux native on Debian 13 (Trixie) - Hardware target: laptop
with **2 GB RAM + Celeron** - Intended optimization path: **Forge +
OptiFine**, followed by FPS optimization if needed

## What Has Been Completed

Prism Launcher was installed successfully:

``` text
Prism Launcher 11.0.2
```

System Java 21 was already installed:

``` text
OpenJDK 21.0.12
```

Because Minecraft 1.7.10 is an old Minecraft version, a separate Java 8
runtime was installed instead of replacing the system Java:

``` text
Temurin 8
OpenJDK 1.8.0_502
64-Bit Server VM
```

Java 8 path:

``` text
/usr/lib/jvm/temurin-8-jre-amd64/bin/java
```

Prism was able to open its main window.

## Failed Attempt

The initial Prism launch produced several filesystem/log warnings and
eventually:

``` text
Segmentation fault
```

The log also showed that Prism could not initially find:

``` text
libopenal.so
```

and several network requests were aborted or failed.

The exact log showed Prism Launcher 11.0.2 running on Linux x86_64 and
successfully reaching the main window before the crash.

Important: the observed log itself records a **segmentation fault**. The
conclusion that the failure was caused specifically by insufficient RAM
was not established by this log alone.

Separately, the project constraint is that the machine has only **2 GB
RAM**, so RAM pressure remains an important concern for Minecraft
1.7.10, especially once Forge, OptiFine, and additional optimization
mods are introduced.

## Java Situation

Debian 13 does not provide the normal `openjdk-8-jre` package:

``` text
Package openjdk-8-jre is not available
Candidate: (none)
```

Therefore Java 8 was installed separately using Temurin.

Current intended Java arrangement:

``` text
System / modern Minecraft
    → Java 21

Minecraft 1.7.10
    → Temurin Java 8
```

Do not replace the system Java 21 just for Minecraft 1.7.10.

## Intended Minecraft Setup

The planned final test is not simply vanilla.

Target structure:

``` text
Minecraft 1.7.10
    ↓
Forge 1.7.10
    ↓
OptiFine 1.7.10
    ↓
Additional compatible FPS optimization
```

The launcher itself is not expected to provide a meaningful FPS
advantage when every launcher uses the same Minecraft version, Java
runtime, loader, mods, and settings.

Prism is being retained because of its overall Linux/modding reputation,
instance management, Java management, and configuration flexibility.

## RAM Constraint

Machine RAM:

``` text
2 GB
```

Initial conservative target for Minecraft:

``` text
Minimum RAM: 512 MiB
Maximum RAM: 768 MiB
```

Do not immediately allocate the entire available RAM to Minecraft
because Debian XFCE and background processes also require memory.

The actual optimal allocation should be tested rather than assumed.

## Next Experiments

The next work should be kept as a **separate follow-up .md/project**
rather than mixing it into this session.

Suggested follow-up sequence:

### Experiment 1 --- Prism + Java 8 + Minecraft 1.7.10

Create a 1.7.10 instance and explicitly select:

``` text
/usr/lib/jvm/temurin-8-jre-amd64/bin/java
```

Test whether the base instance launches.

### Experiment 2 --- Forge 1.7.10

Create the intended Forge 1.7.10 instance.

Test launch before adding OptiFine.

### Experiment 3 --- OptiFine 1.7.10

Add the appropriate OptiFine 1.7.10 build.

Test FPS and stability.

### Experiment 4 --- FPS Optimization

Only after Forge + OptiFine is confirmed working, investigate compatible
optimization options such as:

``` text
OptiFine settings
render distance
simulation/entity settings
particle settings
animation settings
memory allocation
JVM behavior
```

Any additional mods should be checked for actual 1.7.10 compatibility
before installation.

## Current Checkpoint

The project currently stops here:

``` text
Prism Launcher 11.0.2       ✓ installed
Java 21                      ✓ installed
Temurin Java 8               ✓ installed
Java 8 executable             ✓ verified
Prism main window             ✓ opened
Minecraft 1.7.10              → next experiment
Forge                          → not yet tested
OptiFine                      → not yet tested
FPS optimization              → not yet tested
Offline account                → not yet configured
```

## Important

This session should be treated as the **Prism Launcher setup /
failed-attempt checkpoint**.

Further Minecraft 1.7.10 experiments should be documented in another
`.md` so the failed Prism setup and the later Forge/OptiFine/FPS
experiments remain separated.
