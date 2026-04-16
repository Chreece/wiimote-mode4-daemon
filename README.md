# 🎮 Wiimote Mode 4 (DolphinBar) → Virtual Gamepad (Linux)

Use **Wiimotes on Linux** with the **Mayflash DolphinBar in Mode 4** by converting raw HID input into a virtual gamepad using `uinput`.

This project is a small Linux daemon that bridges the gap between:

- **Mayflash DolphinBar Mode 4** raw Wiimote HID input
- **RetroArch** and other software that expect a normal Linux controller

## Features

- Works with **Mayflash DolphinBar (Mode 4)**
- Converts Wiimote input into a **virtual gamepad**
- Exposes a controller named `Wiimote Mode4 Virtual Pad`
- Supports **RetroArch**
- Handles **connect / disconnect detection**
- Supports multiple Wiimote report modes used by Dolphin / libretro cores
- No Bluetooth pairing on the Linux host required
- Low latency
- Includes a **RetroArch autoconfig file** in the repository

## How it works

The daemon:

1. Reads Wiimote traffic from `/dev/hidraw*`
2. Detects valid Wiimote report traffic coming from the DolphinBar
3. Translates Wiimote button states into Linux input events
4. Creates a virtual controller through `uinput`
5. Removes the virtual controller again when the Wiimote is no longer present

## Requirements

- Linux
- Python 3
- `python3-evdev`
- `uinput` kernel module
- Mayflash DolphinBar set to **Mode 4**

## Install dependencies

```bash
sudo apt install python3-evdev
```

## Enable `uinput`

```bash
sudo modprobe uinput
```

To load it automatically on boot:

```bash
echo uinput | sudo tee /etc/modules-load.d/uinput.conf
```

## Installation

### 1. Install the script

```bash
sudo cp wiimote-mode4-daemon /usr/local/bin/
sudo chmod +x /usr/local/bin/wiimote-mode4-daemon
```

### 2. Create the systemd service

```bash
sudo tee /etc/systemd/system/wiimote-mode4-daemon.service >/dev/null <<'EOF'
[Unit]
Description=Wiimote Mode4 hidraw to virtual gamepad bridge
After=network.target

[Service]
ExecStart=/usr/local/bin/wiimote-mode4-daemon
Restart=always
RestartSec=2

[Install]
WantedBy=multi-user.target
EOF
```

### 3. Enable and start it

```bash
sudo systemctl daemon-reload
sudo systemctl enable wiimote-mode4-daemon
sudo systemctl start wiimote-mode4-daemon
```

### 4. Install the included RetroArch autoconfig file

Copy the autoconfig file from this repository into your RetroArch autoconfig directory:

```bash
mkdir -p ~/.config/retroarch/autoconfig
cp "Wiimote Mode4 Virtual Pad.cfg" ~/.config/retroarch/autoconfig/
```

If your repository uses a different folder name, adjust the source path accordingly.

## Usage

1. Set the DolphinBar to **Mode 4**
2. Turn on a Wiimote
3. Press a button on the Wiimote
4. A virtual controller should appear on Linux

Check whether the virtual controller exists:

```bash
grep -c 'Wiimote Mode4 Virtual Pad' /proc/bus/input/devices
```

## RetroArch

This repository includes a ready-to-use RetroArch autoconfig file for:

```text
Wiimote Mode4 Virtual Pad
```

After copying the file into RetroArch’s autoconfig directory, RetroArch should load the mapping automatically when the virtual controller appears.

## Troubleshooting

### No controller appears

Check that the DolphinBar is visible to Linux:

```bash
lsusb | grep 057e:0306
```

If nothing appears, the DolphinBar is not attached to the host.

### Multiple virtual controllers appear

Make sure you only have **one daemon instance** running:

```bash
pgrep -af wiimote-mode4-daemon
```

If needed, clean it up and restart:

```bash
sudo systemctl stop wiimote-mode4-daemon
sudo pkill -f wiimote-mode4-daemon
sudo systemctl start wiimote-mode4-daemon
```

### Random disconnects

Check the logs:

```bash
journalctl -u wiimote-mode4-daemon -n 50
```

If you are logging to a file, check that too:

```bash
tail -n 50 /tmp/wiimote-mode4-daemon.log
```

### No input inside RetroArch

Check that the virtual device exists:

```bash
grep 'Wiimote Mode4 Virtual Pad' /proc/bus/input/devices
```

Also confirm the included autoconfig file was copied to:

```text
~/.config/retroarch/autoconfig/
```

### Check which process is using the hidraw devices

```bash
sudo fuser -v /dev/hidraw*
```

## Notes

- This project is for **DolphinBar Mode 4**
- It maps **Wiimote buttons** to a virtual controller
- It does **not** currently provide motion / IR support
- Software that directly grabs the same `hidraw` device may interfere with it

## Known limitations

- Requires the Mayflash DolphinBar in Mode 4
- No motion support yet
- No IR pointer support yet
- No rumble / LED sync yet
- Multi-Wiimote handling may still need improvement depending on setup

## Future ideas

- Motion support
- IR support
- Rumble support
- LED sync
- Better multi-Wiimote support
- Cleaner per-emulator profiles

## 🤖 AI Assistance

This project was developed with the help of **ChatGPT**.

The implementation, debugging, and reverse-engineering of the DolphinBar Mode 4 behavior were done through iterative testing and AI-assisted development.

All code and behavior have been validated and tested in a real Linux environment.

## Credits

This project was built by reverse-engineering the behavior of:

- the **Mayflash DolphinBar**
- **Wiimote HID report modes**
- Linux `hidraw`
- Linux `uinput`Then rebind controls inside RetroArch.
