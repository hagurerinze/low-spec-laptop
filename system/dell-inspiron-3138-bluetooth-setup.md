# Bluetooth Setup — Dell Inspiron 3138 (Debian 13)

## 1. System Information

| Component | Information |
|---|---|
| Laptop | Dell Inspiron 3138 |
| OS | Debian GNU/Linux 13 (Trixie) |
| Desktop | XFCE |
| Bluetooth Adapter | Qualcomm Atheros AR9462 |
| USB ID | `0cf3:0036` |
| Bluetooth Stack | BlueZ |
| Audio Server | PulseAudio |

## 2. Check Bluetooth Hardware

Check the USB devices:

```bash
lsusb
```

Detected adapter:

```text
ID 0cf3:0036 Qualcomm Atheros Communications AR9462 Bluetooth
```

Check the wireless controller:

```bash
lspci -nnk
```

Detected network controller:

```text
Qualcomm Atheros QCA9565 / AR9565 [168c:0036]
Kernel driver in use: ath9k
```

Check the Bluetooth controller:

```bash
bluetoothctl list
```

The controller was detected:

```text
Controller 9C:D2:1E:93:36:BA Hagure [default]
```

## 3. Install Bluetooth Packages

Install BlueZ and Blueman:

```bash
sudo apt update
sudo apt install -y bluetooth bluez blueman
```

The installation also included Bluetooth audio support:

```text
pulseaudio-module-bluetooth
```

## 4. Enable Bluetooth Service

Enable and start the Bluetooth service:

```bash
sudo systemctl enable --now bluetooth
```

Restart the service:

```bash
sudo systemctl restart bluetooth
```

Optional: install `rfkill` to manage wireless device blocks.

```bash
sudo apt install rfkill
```

Then:

```bash
sudo rfkill unblock bluetooth
```

## 5. Scan for Bluetooth Devices

Open the Bluetooth command-line tool:

```bash
bluetoothctl
```

Run these commands:

```text
power on
agent on
default-agent
scan on
```

The controller successfully started discovery and detected nearby devices.

Example:

```text
[NEW] Device 4D:91:46:F2:C8:8E
[NEW] Device 5B:42:31:7D:2F:E0
```

Stop scanning:

```text
scan off
```

Exit:

```text
exit
```

## 6. Blueman GUI

Open the graphical Bluetooth manager:

```bash
blueman-manager
```

Blueman can be used to manage nearby devices, pairing, and connections.

## 7. Fix Bluetooth Audio Profile

### Problem

Blueman displayed these warnings:

```text
blueman.bluez.errors.DBusFailedError:
br-connection-profile-unavailable
```

And:

```text
No audio endpoints registered to bluetoothd.
Pulseaudio Bluetooth module, bluez-alsa, PipeWire
or other audio support missing.
```

### Check the audio server

```bash
pactl info | grep -E 'Server Name|Server String'
```

Result:

```text
Server String: /run/user/1000/pulse/native
Server Name: pulseaudio
```

Check audio services:

```bash
systemctl --user status pipewire pipewire-pulse pulseaudio --no-pager
```

Result:

- PulseAudio: active and running.
- PipeWire: not installed.

Check loaded Bluetooth modules:

```bash
pactl list modules short | grep -i bluetooth
```

The command returned no output. No Bluetooth module was loaded.

### Solution

Load the PulseAudio Bluetooth discovery module:

```bash
pactl load-module module-bluetooth-discover
```

Then check the loaded modules again:

```bash
pactl list modules short | grep -i bluetooth
```

After loading the module, reopen Blueman:

```bash
blueman-manager
```

**Result:** Bluetooth audio support was working after the module was loaded.

No PipeWire installation was needed.

## 8. Current Status

| Feature | Status |
|---|---|
| Bluetooth hardware detection | Working |
| Bluetooth service | Working |
| Device scanning | Working |
| Blueman GUI | Working |
| PulseAudio | Working |
| Bluetooth audio discovery module | Loaded successfully |
| Bluetooth audio test | Passed |
| OBEX file transfer | Pending further work |

## 9. Notes and Future Work

### OBEX Support — Draft

OBEX support is not yet confirmed as fixed.

The OBEX configuration and fix may be different from the setup used on the Acer Aspire ES1-432 (N3K).

Further investigation is needed before applying the same solution.

**Status:** Pending comparison with the Acer Aspire configuration.

---

*Document status: Initial Bluetooth setup and audio troubleshooting completed. OBEX work remains open.*
