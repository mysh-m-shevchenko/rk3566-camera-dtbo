# RK3566 Camera Device Tree Overlays

Device tree overlays for camera sensors and HDMI-to-CSI bridges on RK3566-based boards (Radxa Zero 3W, etc.).

## Supported Devices

### Camera Sensors

| Overlay | Sensor | Resolution | Description |
|---------|--------|------------|-------------|
| `rk3566-camera-imx219` | Sony IMX219 | 8MP | Raspberry Pi Camera Module v2 |
| `rk3566-camera-imx290` | Sony IMX290/IMX327 | 2MP | Low-light/starlight sensor |
| `rk3566-camera-imx415` | Sony IMX415 | 8MP/4K | 4K UHD sensor |
| `rk3566-camera-imx477` | Sony IMX477 | 12.3MP | Raspberry Pi HQ Camera |
| `rk3566-camera-imx708` | Sony IMX708 | 12MP | Raspberry Pi Camera Module v3 |
| `rk3566-camera-ov5640` | OmniVision OV5640 | 5MP | Autofocus sensor |
| `rk3566-camera-ov5647` | OmniVision OV5647 | 5MP | Raspberry Pi Camera Module v1.3 |
| `rk3566-camera-ov8858` | OmniVision OV8858 | 8MP | Mobile/embedded sensor |
| `rk3566-camera-ov9281` | OmniVision OV9281 | 1MP | Global shutter (mono) |
| `rk3566-camera-ov13850` | OmniVision OV13850 | 13MP | High-resolution sensor |
| `rk3566-camera-gc2053` | GalaxyCore GC2053 | 2MP | Security camera sensor |
| `rk3566-camera-gc2093` | GalaxyCore GC2093 | 2MP | HDR security sensor |

### HDMI/Video Bridges

| Overlay | Chip | Description |
|---------|------|-------------|
| `rk3566-hdmi-csi-tc358743` | Toshiba TC358743XBG | HDMI 1.4 to MIPI CSI-2 (up to 1080p30 in 2-lane mode) |
| `rk3566-hdmi-csi-tc358746` | Toshiba TC358746AXBG | Parallel video to MIPI CSI-2 |

## Hardware Configuration

All overlays use:
- **I2C Bus**: I2C2 (pins on 40-pin header)
- **CSI Interface**: CSI2 DPHY0 in full mode (2 lanes)
- **Reset GPIO**: GPIO3_C6

**Note**: Only one camera overlay can be active at a time as they all use the same CSI2 DPHY0 interface.

## Installation

```bash
sudo dpkg -i rk3566-camera-dtbo_1.0.0_arm64.deb
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

## Sensor Categories

### Raspberry Pi Compatible
- **IMX219** - RPi Camera v2 (8MP)
- **OV5647** - RPi Camera v1.3 (5MP)
- **IMX477** - RPi HQ Camera (12.3MP)
- **IMX708** - RPi Camera v3 (12MP)

### Low-Light / Security
- **IMX290/IMX327** - Starlight/low-light sensor (2MP, same driver)
- **GC2053** - Security camera sensor (2MP)
- **GC2093** - HDR security sensor (2MP)

### High Resolution
- **OV13850** - 13MP sensor
- **IMX415** - 4K UHD sensor (8MP)
- **OV8858** - 8MP sensor

### Special Purpose
- **OV5640** - 5MP with autofocus
- **OV9281** - Global shutter mono (1MP, machine vision)

### HDMI Capture
- **TC358743** - HDMI to CSI-2 bridge (1080p30 in 2-lane)
- **TC358746** - Parallel to CSI-2 bridge

## Example: HDMI Capture Setup

1. Install the package:
   ```bash
   sudo dpkg -i rk3566-camera-dtbo_1.0.0_arm64.deb
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
- **Reset**: GPIO3_C6
- **Power**: 3.3V and GND
- **CSI**: 15-pin FPC to board CSI connector

### Camera Sensors

Standard Raspberry Pi camera ribbon cable to CSI connector.

## Kernel Driver Requirements

Some sensors may require kernel configuration:

| Sensor | Kernel Config |
|--------|---------------|
| IMX219 | `CONFIG_VIDEO_IMX219=y` |
| IMX290/327 | `CONFIG_VIDEO_IMX290=y` |
| IMX415 | `CONFIG_VIDEO_IMX415=y` |
| IMX477 | `CONFIG_VIDEO_IMX477=y` |
| IMX708 | `CONFIG_VIDEO_IMX708=y` |
| OV5640 | `CONFIG_VIDEO_OV5640=y` |
| OV5647 | `CONFIG_VIDEO_OV5647=y` |
| OV8858 | `CONFIG_VIDEO_OV8858=y` |
| OV9281 | `CONFIG_VIDEO_OV9281=y` |
| OV13850 | `CONFIG_VIDEO_OV13850=y` |
| GC2053 | `CONFIG_VIDEO_GC2053=y` |
| GC2093 | `CONFIG_VIDEO_GC2093=y` |
| TC358743 | `CONFIG_VIDEO_TC358743=y` |
| TC358746 | `CONFIG_VIDEO_TC358746=y` |

## Troubleshooting

### No device detected

1. Check I2C connection:
   ```bash
   sudo i2cdetect -y 2
   ```

2. Verify overlay is loaded:
   ```bash
   dmesg | grep -i "camera\|tc358\|imx\|ov56\|gc20"
   ```

3. Check media pipeline:
   ```bash
   media-ctl -p
   ```

### Wrong resolution or format

Adjust link frequency in the overlay source (`.dts` file) and rebuild.

### ISP/rkaiq issues

Some sensors (especially IMX708) require proper ISP tuning files (IQ files) for correct color rendering.

## Building from Source

```bash
cd /path/to/rk3566-camera-dtbo
dpkg-buildpackage -us -uc -b -aarm64
```

## Sources

- [Radxa Wiki](https://wiki.radxa.com/Device-tree-overlays)
- [Firefly Wiki - Camera](https://wiki.t-firefly.com/en/ROC-RK3566-PC/driver_camera.html)
- [Linux Kernel DT Bindings](https://www.kernel.org/doc/Documentation/devicetree/bindings/media/i2c/)
- [Rockchip Linux Kernel](https://github.com/rockchip-linux/kernel)

## License

Copyright (c) 2024 MYSh LLC

Device tree overlays are licensed under GPL-2.0 (kernel compatible).
