# RK3566 Camera Device Tree Overlays

Device tree overlays for camera sensors and HDMI-to-CSI bridges on RK3566-based boards (Radxa Zero 3W, etc.).

## Supported Devices

### Camera Sensors

| Overlay | Sensor | Resolution | Description |
|---------|--------|------------|-------------|
| `rk3566-camera-imx219` | Sony IMX219 | 8MP | Raspberry Pi Camera Module v2 |
| `rk3566-camera-ov5647` | OmniVision OV5647 | 5MP | Raspberry Pi Camera Module v1.3 |
| `rk3566-camera-imx477` | Sony IMX477 | 12.3MP | Raspberry Pi HQ Camera |
| `rk3566-camera-gc2053` | GalaxyCore GC2053 | 2MP | Common in security cameras |

### HDMI/Video Bridges

| Overlay | Chip | Description |
|---------|------|-------------|
| `rk3566-hdmi-csi-tc358743` | Toshiba TC358743XBG | HDMI 1.4 to MIPI CSI-2 (up to 1080p30 in 2-lane mode) |
| `rk3566-hdmi-csi-tc358746` | Toshiba TC358746AXBG | Parallel video to MIPI CSI-2 |

## Hardware Configuration

All overlays use:
- **I2C Bus**: I2C2 (pins on 40-pin header)
- **CSI Interface**: CSI2 DPHY0 in full mode (2 lanes)
- **Reset GPIO**: GPIO3_C6 (directly controllable)

**Note**: Only one camera overlay can be active at a time as they all use the same CSI2 DPHY0 interface.

## Installation

```bash
sudo dpkg -i rk3566-camera-dtbo_1.0.0_all.deb
```

## Usage

### List available overlays

```bash
camera-overlay list
```

### Show enabled overlays

```bash
camera-overlay status
```

### Enable a camera overlay

```bash
sudo camera-overlay enable rk3566-camera-imx219
```

### Disable a camera overlay

```bash
sudo camera-overlay disable rk3566-camera-imx219
```

### Show overlay information

```bash
camera-overlay info rk3566-hdmi-csi-tc358743
```

## Configuration Files

The utility modifies:
- `/boot/extlinux/extlinux.conf` - Primary boot configuration (fdtoverlays line)
- `/boot/uEnv.txt` - Alternative boot configuration (overlays line)

After enabling/disabling overlays, **reboot is required** for changes to take effect.

## Example: HDMI Capture Setup

1. Install the package:
   ```bash
   sudo dpkg -i rk3566-camera-dtbo_1.0.0_all.deb
   ```

2. Enable TC358743 overlay:
   ```bash
   sudo camera-overlay enable rk3566-hdmi-csi-tc358743
   sudo reboot
   ```

3. After reboot, verify the device:
   ```bash
   v4l2-ctl --list-devices
   media-ctl -p
   ```

4. Capture video:
   ```bash
   gst-launch-1.0 v4l2src device=/dev/video0 ! videoconvert ! autovideosink
   ```

## Hardware Wiring

### TC358743 (HDMI Input)

Connect TC358743 module to Zero 3W:
- **I2C**: SDA to GPIO3_B5 (Pin 3), SCL to GPIO3_B6 (Pin 5)
- **Reset**: GPIO3_C6 (directly use pin)
- **Power**: 3.3V and GND
- **CSI**: 15-pin FPC to board CSI connector

### Camera Sensors

Standard Raspberry Pi camera ribbon cable to CSI connector.

## Troubleshooting

### No device detected

1. Check I2C connection:
   ```bash
   sudo i2cdetect -y 2
   ```

2. Verify overlay is loaded:
   ```bash
   dmesg | grep -i "camera\|tc358\|imx\|ov56"
   ```

3. Check media pipeline:
   ```bash
   media-ctl -p
   ```

### Wrong resolution or format

Adjust link frequency in the overlay source (`.dts` file) and rebuild.

## Building from Source

```bash
cd /path/to/rk3566-camera-dtbo
dpkg-buildpackage -us -uc -b
```

## License

Copyright (c) 2024 MYSh LLC

Device tree overlays are licensed under GPL-2.0 (kernel compatible).
