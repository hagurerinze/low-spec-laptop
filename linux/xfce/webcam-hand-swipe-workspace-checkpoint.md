# Webcam Hand-Swipe Workspace Control — Checkpoint

**Status:** In progress; paused before the next test  
**Machine:** Acer Aspire ES1-432 (Laptop 1)  
**OS / desktop:** Debian GNU/Linux 13 (trixie), Xfce 4.20.2, X11  
**CPU / RAM:** Intel Celeron N3350 @ 1.10 GHz, 2 cores; 2 GB RAM  
**Goal:** Use the built-in webcam to detect a left/right hand swipe and eventually switch Xfce workspaces. This project is Xfce-only; bspwm support is out of scope.

## 1. Current hardware and software checks

The built-in webcam is available at `/dev/video0`. `/dev/video1` did not work in earlier tests. A basic OpenCV capture test succeeded:

```text
OpenCV sistem: 4.10.0
Webcam terbuka: True
Frame berhasil: True
```

The CPU flags do **not** include `avx`. The relevant CPU is:

```text
Intel(R) Celeron(R) CPU N3350 @ 1.10GHz
```

System packages already installed:

- `python3-opencv` — Debian OpenCV 4.10.0
- `v4l-utils`
- `wmctrl`
- `xdotool`

The Xfce session has four workspaces. `wmctrl -d` previously showed Workspace 1 through Workspace 4.

## 2. What has been tried

### Initial frame-difference prototype

`~/gesture-workspace/test-swipe.py` used OpenCV frame differences to infer horizontal movement. It detected movement, but it could react to the face/body as well as the hand. This is not suitable as the final implementation.

### MediaPipe attempt

A Python virtual environment exists at:

```text
~/gesture-workspace/venv
```

MediaPipe and its hand-landmarker model were installed/downloaded:

```text
MediaPipe: 1.1.0
OpenCV in venv: 5.0.0
Model: ~/gesture-workspace/hand_landmarker.task
Model size: about 7.5 MiB
```

The script `~/gesture-workspace/hand-swipe.py` attempted to use MediaPipe Tasks `HandLandmarker`. Running it failed with:

```text
FATAL ERROR: This binary was compiled with avx enabled, but this feature is not available on this processor (go/sigill-fail-fast).
Illegal instruction
```

**Conclusion:** the MediaPipe native binary/runtime used in the virtual environment is incompatible with this CPU because AVX is unavailable. This is not evidence of a webcam or model-file failure. Do not keep rerunning this binary on this laptop. The script and model can be kept as historical experiment files, but this route is currently blocked.

## 3. Verified working baseline

This command was run successfully using Debian's system Python and OpenCV:

```bash
python3 - <<'PY'
import cv2
print("OpenCV sistem:", cv2.__version__)
cap = cv2.VideoCapture("/dev/video0", cv2.CAP_V4L2)
print("Webcam terbuka:", cap.isOpened())
if cap.isOpened():
    ok, frame = cap.read()
    print("Frame berhasil:", ok)
cap.release()
PY
```

Result:

```text
OpenCV sistem: 4.10.0
Webcam terbuka: True
Frame berhasil: True
```

This is the known-good baseline for future work. Use system `python3` and Debian OpenCV for the next prototype, not the MediaPipe virtual environment.

## 4. Proposed next step — not yet run

The next proposed experiment is a simple OpenCV motion detector that watches only a defined region near the bottom of the camera image. Restricting the region may reduce false detections from the face, but it does **not** identify a hand semantically. Lighting changes, background movement, and other objects in the region can still trigger it.

The proposed script will be created at:

```text
~/gesture-workspace/hand-swipe-cv.py
```

It has **not been confirmed as run or tested yet**. The user paused before taking this step and requested this checkpoint.

When resuming:

1. Create/run the region-based OpenCV prototype.
2. Test left-to-right and right-to-left hand movement inside the green rectangle.
3. Check whether face movement still triggers detections.
4. Adjust the region/thresholds only after observing the result.
5. Only after detection is reasonably stable, connect detected directions to Xfce workspace switching using `wmctrl`.
6. Add a cooldown/debounce so one swipe does not trigger multiple workspace changes.
7. Keep workspace switching disabled during early camera tests.

## 5. Design constraints and safety notes

- Target is the Acer Aspire ES1-432 with 2 GB RAM and an Intel Celeron N3350.
- Prefer low-resource Debian system packages and avoid unnecessary heavyweight runtimes.
- The webcam itself and system OpenCV have passed a basic capture test.
- Do not treat frame-difference motion as true hand recognition.
- Do not alter Xfce workspace configuration or other desktop settings until the detector has been tested.
- The eventual action should switch only among the existing four Xfce workspaces.
- This project does not need bspwm integration.

## 6. Current checkpoint summary

**Working:** `/dev/video0`, Debian system Python, system OpenCV 4.10.0, four Xfce workspaces.

**Blocked:** MediaPipe runtime installed through pip crashes due to unavailable AVX instructions on the CPU.

**Next action:** Test a region-based OpenCV motion prototype. No workspace switching has been connected yet.
