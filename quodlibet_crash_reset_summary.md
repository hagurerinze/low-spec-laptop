# Quod Libet Crash After Moving Audio Files

## Problem

Quod Libet on Debian started crashing after the user's audio files were moved.

The initial hypothesis is that Quod Libet's cached library database still contains references to the old audio-file locations, or that its library database became corrupted.

The recommended approach is to reset the Quod Libet library database first, without deleting the actual audio files.

## Step 1 — Close Quod Libet

```bash
pkill -x quodlibet
```

## Step 2 — Check the configuration directory

Run:

```bash
ls -ld ~/.config/quodlibet ~/.quodlibet 2>/dev/null
```

Possible configuration locations:

- `~/.config/quodlibet`
- `~/.quodlibet`

## Step 3 — Reset only the cached song database

If `~/.config/quodlibet` exists:

```bash
mv ~/.config/quodlibet/songs ~/.config/quodlibet/songs.backup
```

If `~/.quodlibet` is the active directory:

```bash
mv ~/.quodlibet/songs ~/.quodlibet/songs.backup
```

Then start Quod Libet:

```bash
quodlibet
```

Quod Libet should rebuild its library database. The old database is retained as `songs.backup`.

This is the preferred first reset because it is less destructive than resetting the entire configuration.

## Step 4 — Full configuration reset if the crash continues

Close Quod Libet:

```bash
pkill -x quodlibet
```

For the newer configuration path:

```bash
mv ~/.config/quodlibet ~/.config/quodlibet.backup
```

Then:

```bash
quodlibet
```

If the older configuration path is being used:

```bash
pkill -x quodlibet
mv ~/.quodlibet ~/.quodlibet.backup
quodlibet
```

The `.backup` directory is kept so the previous configuration can be recovered if needed.

## Step 5 — Generate a debug log

If Quod Libet still crashes:

```bash
quodlibet --debug 2>&1 | tee ~/quodlibet-debug.log
```

The resulting file is:

```text
~/quodlibet-debug.log
```

This log can help determine whether the crash is caused by:

- stale/corrupted Quod Libet library data,
- references to the old audio paths,
- GStreamer or another audio backend,
- or another Quod Libet runtime/startup error.

## Recommended troubleshooting order

1. Close Quod Libet.
2. Check which configuration directory exists.
3. Back up/reset only the `songs` database.
4. Start Quod Libet and test it.
5. If it still crashes, back up/reset the complete Quod Libet configuration.
6. If it still crashes, run the debug command.
7. Inspect `~/quodlibet-debug.log`.

## Important

The `mv` commands above do not delete the actual music files. They only rename Quod Libet's database/configuration files or directories.

Keep the `.backup` files until Quod Libet is confirmed to work normally.
