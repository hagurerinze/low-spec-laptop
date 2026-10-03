# Debian Webcam Troubleshooting

## Context

The webcam was tested on Debian using multiple camera applications.

The goal was to determine whether the camera itself was working or whether the problem was specific to the webcam application.

## Initial Test: Cheese

Running:

```bash
cheese
```

produced:

```text
(org.gnome.Cheese:2357): cheese-CRITICAL **: 16:53:42.326: GValue type gint x GstValueList, cannot be handled for resolution
Segmentation fault
```

Cheese therefore crashed during camera/resolution handling.

## Cross-Testing

The camera was then tested with other applications.

### guvcview

```bash
guvcview
```

Result:

- Camera opened successfully.
- Live camera output was available.

### ffplay

```bash
ffplay /dev/video0
```

Result:

- Camera opened successfully.
- Live camera output was available.

These tests established that the webcam hardware and the Linux video device were functional.

## Diagnosis

Because both `guvcview` and `ffplay` could access the camera while Cheese crashed, the problem was isolated to Cheese rather than the webcam itself.

The Cheese error:

```text
GValue type gint x GstValueList, cannot be handled for resolution
```

suggests a problem with Cheese/GStreamer handling of the camera's advertised resolution or format.

There was therefore no need to reinstall or replace the webcam driver based on these tests.

## Removing Cheese

Cheese was removed from the system:

```bash
sudo apt purge cheese
sudo apt autoremove --purge
```

User-level Cheese configuration and cache can also be removed:

```bash
rm -rf ~/.config/cheese
rm -rf ~/.local/share/cheese
rm -rf ~/.cache/cheese
```

Optional package-cache cleanup:

```bash
sudo apt autoclean
```

Verify that Cheese is no longer installed:

```bash
which cheese
```

If no path is returned, the `cheese` executable is no longer available.

## Useful Webcam Diagnostics

Check video devices:

```bash
ls /dev/video*
```

Normally a working webcam exposes a device such as:

```text
/dev/video0
```

Check V4L2 devices:

```bash
v4l2-ctl --list-devices
```

Install `v4l-utils` if necessary:

```bash
sudo apt install v4l-utils
```

Check supported formats and resolutions:

```bash
v4l2-ctl --list-formats-ext
```

Check whether the common UVC webcam driver is loaded:

```bash
lsmod | grep uvcvideo
```

If required, load it manually:

```bash
sudo modprobe uvcvideo
```

## Conclusion

The webcam was confirmed to work under Debian.

Working:

- `guvcview`
- `ffplay`
- V4L2 camera device

Failing:

- `cheese`

The failure was therefore treated as a Cheese/GStreamer compatibility issue rather than a hardware or kernel webcam failure.

## Repository Placement

Recommended location:

```text
low-spec-laptop/
└── hardware/
    └── camera-webcam-troubleshooting.md
```

This topic belongs under `hardware/` because the investigation covers webcam detection, V4L2, the kernel driver, supported formats, and application-level camera access.

## Suggested Commit Messages

All of these are under 50 characters:

```text
Document webcam troubleshooting
```

```text
Document Cheese webcam crash
```

```text
Document Debian webcam testing
```

```text
Add webcam troubleshooting notes
```

Recommended:

```text
Document Cheese webcam crash
```

This is specific to the actual incident while remaining concise.
