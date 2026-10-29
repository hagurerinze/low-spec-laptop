# Hagure-DL --- Initial Documentation

Status: Initial checkpoint; to be updated after the next round of testing.

## Purpose

A Bash toolbox for downloading video and audio with `yt-dlp` on Debian 13 XFCE. The primary target for the video profiles is the Dell Inspiron 3138, equipped with an Intel Celeron N2815 and older Intel HD Graphics. The focus is ease of use and playback compatibility, not simply the highest available resolution.

## Environment at This Checkpoint

- OS: Debian GNU/Linux 13 (trixie)
- Upstream `yt-dlp` binary: `/usr/local/bin/yt-dlp`
- Tested version: `2026.10.26`
- FFmpeg: `7.1.5`
- Toolbox output directory: `~/Videos/Downloader`
- Toolbox script: `~/bin/hagure-dl`

Check the active versions and executable locations:

```bash
command -v yt-dlp
yt-dlp --version
ffmpeg -version
```

## Initial Test Results

### Automatic Download

Command:

```bash
yt-dlp "VIDEO_URL"
```

The download succeeded using `yt-dlp`'s default format selection. The output format is not guaranteed to be MP4; the result may be WebM, and `yt-dlp` may select a high-resolution format depending on what is available.

### Verbose Mode

Command:

```bash
yt-dlp -v "VIDEO_URL"
```

This does not use a different download profile. The `-v` option enables diagnostic logging, while format selection still uses the default behavior when `-f` is not specified.

### Format Selection with `b`

Command:

```bash
yt-dlp -f "b" "VIDEO_URL"
```

During testing, this command failed with the message `Requested format is not available`. This means the `b` format selector did not match a format available from that extraction; it does not prove that all downloads fail.

Check the available formats first:

```bash
yt-dlp -F "VIDEO_URL"
```

### JavaScript Runtime Warning

Earlier logs showed a warning that no supported JavaScript runtime was available. The download still succeeded for the tested URL, but some formats may be unavailable. If extraction problems occur again, check the official yt-dlp EJS documentation before installing an additional runtime.

## Planned Profiles

| Menu | Profile | Purpose |
|---|---|---|
| 1 | H.264/AVC (`avc1`), up to 720p | Main profile for compatibility with the Dell |
| 2 | H.264/AVC, up to 480p | Lower resolution to reduce workload and file size |
| 3 | Best available quality | Prioritizes quality rather than compatibility |
| 4 | M4A audio | Audio only |
| 5 | MP4 video, if available | Selects an available MP4 format |
| 6 | List formats | Inspect resolution, codec, and container |
| 7 | H.264 playlist, up to 720p | Download a playlist |

Note: Resolution and codec can affect decoding workload, but smooth playback also depends on bitrate, frame rate, audio/video codecs, and system condition. H.264 at 720p is a reasonable starting profile for the Dell, not a guarantee that every video will play without lag.

## Selectors Currently Used in the Toolbox Design

H.264 720p profile:

```bash
-f 'bv*[vcodec^=avc1][height<=720]+ba[ext=m4a]/b[ext=mp4][height<=720]' \
--merge-output-format mp4
```

H.264 480p profile:

```bash
-f 'bv*[vcodec^=avc1][height<=480]+ba[ext=m4a]/b[ext=mp4][height<=480]' \
--merge-output-format mp4
```

M4A audio:

```bash
-f 'ba[ext=m4a]/ba' -x --audio-format m4a
```

Important: The current video selectors have a limited MP4 fallback. If a video does not provide a matching format combination, the menu may fail. Use `yt-dlp -F "VIDEO_URL"` to inspect the available formats before changing the selector. `--merge-output-format mp4` specifies the output container when merging is required; it does not convert the video codec to H.264.

## File Locations

Files downloaded through the toolbox are saved to:

```text
~/Videos/Downloader
```

Therefore, `ls -lh ~/*.mp4` checks only MP4 files directly inside the home directory and will not find files in the toolbox output directory. Check toolbox downloads with:

```bash
ls -lh ~/Videos/Downloader
```

To check video and audio files downloaded directly into the home directory:

```bash
find ~ -maxdepth 1 -type f \( -iname '*.mp4' -o -iname '*.webm' -o -iname '*.mkv' -o -iname '*.m4a' \) -printf '%f\t%k KB\n'
```

## Next Testing Steps

1. Run the toolbox and test menu 1 with the same URL.
2. If it fails, run `yt-dlp -F "VIDEO_URL"` and record the available `avc1` formats, resolutions, and audio formats.
3. Confirm whether the video is actually merged into an MP4 container and whether the resulting video codec remains H.264.
4. Test menu 4 to verify that the output is M4A audio.
5. Improve the selectors to provide more reliable fallbacks without silently choosing a codec that may be too demanding for the Dell.

## References

- yt-dlp documentation: https://github.com/yt-dlp/yt-dlp
- Format selection guide: https://github.com/yt-dlp/yt-dlp#format-selection
- JavaScript challenge/EJS guide: https://github.com/yt-dlp/yt-dlp/wiki/EJS
