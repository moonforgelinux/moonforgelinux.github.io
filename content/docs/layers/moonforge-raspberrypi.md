---
title: 'meta-moonforge-raspberrypi'
linkTitle: 'Raspberry Pi'
type: 'docs'
weight: 25
description: >
  Adds board support for the Raspberry Pi 4 and Raspberry Pi 5.
---

`meta-moonforge-raspberrypi` provides the BSP (Board Support Package) configuration
needed to build a Moonforge image for the Raspberry Pi 4 and Raspberry Pi 5. 
Given the wide availability and low cost of these boards, this is the 
most common hardware target used when getting started with Moonforge,.

## What it does

- Extends the base image with the recipes, distro settings, and `local.conf`
  configurations required for Raspberry Pi targets.
- Provides a WKS (Wic Kickstart) file that describes the partition layout for an SD
  card: a FAT boot partition, two ext4 rootfs partitions (A and B, for RAUC A/B
  updates), and an ext4 data partition.
- Configures the bootloader and kernel for Raspberry Pi 4 and 5.

## Why use it

The Raspberry Pi is an excellent prototyping and validation platform. Its broad
community support, large selection of peripherals (cameras, touchscreens, HATs), and
availability in production quantities make it a practical target for both development
and low-to-medium volume product deployments.

## Dependencies

- [`meta-moonforge-distro`](../moonforge-distro/), required by all layers.

## How to use it

```yaml
header:
  version: 16
  includes:
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-distro.yml
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-raspberrypi.yml

local_conf_header:
  20_meta-moonforge-distro: |
    OVERLAYFS_ETC_DEVICE = "/dev/mmcblk0p3"
    IMAGE_DATA_MIN_SIZE = "4096M"
  30_meta-moonforge-raspberrypi: |
    WKS_FILE = "moonforge-image-base-raspberrypi.wks.in"

machine: raspberrypi5
```

Set `machine` to `raspberrypi5` for the Pi 5 or `raspberrypi4-64` for the Pi 4
(64-bit mode).

### Configuration variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OVERLAYFS_ETC_DEVICE` | Yes | Persistent data partition,  `/dev/mmcblk0p3` for SD card, `/dev/mmcblk0p4` when using RAUC updates |
| `WKS_FILE` | Yes | WKS partition layout file for Raspberry Pi |
| `IMAGE_DATA_MIN_SIZE` | No | Minimum data partition size in KiB (default: `4096M`) |
| `machine` | Yes | `raspberrypi5` or `raspberrypi4-64` |

## Flashing the SD card

Use `bmaptool` for fast, verified flashing:

```
$ sudo bmaptool copy \
    build/tmp/deploy/images/raspberrypi5/moonforge-image-base-raspberrypi5.rootfs.wic.bz2 \
    /dev/sdX
```

Replace `/dev/sdX` with the device node of your SD card. On most Linux desktops this
is `/dev/sdb` or `/dev/sdc`. You can use `lsblk` to confirm before writing.

> **Warning:** Writing to the wrong device will destroy data. Double-check the device
> node before running `bmaptool`.

`bmaptool` reads the `.bmap` file that Yocto generates alongside the `.wic.bz2` image
to skip empty blocks, making the flash operation significantly faster than `dd`.

## Logging in

Connect a USB serial adapter to the Raspberry Pi's UART pins (GPIO 14/15) and open a
terminal at 115200 baud, or connect a keyboard and HDMI display. Log in as `root` with
no password on debug builds.

## Next steps

Once the base image is running on your Raspberry Pi, add feature layers to the kas
configuration to extend it:

- [`meta-moonforge-docker`](../moonforge-docker/) to run containerised applications.
- [`meta-moonforge-rauc`](../moonforge-rauc/) + [`meta-moonforge-rauc-update`](../moonforge-rauc-update/) to enable OTA updates.
- [`meta-moonforge-graphics`](../moonforge-graphics/) + [`meta-moonforge-wpe`](../moonforge-wpe/) to drive a display with a web-based kiosk UI.

See: [meta-moonforge-raspberrypi on GitHub](https://github.com/moonforgelinux/meta-moonforge/blob/main/kas/include/layer/meta-moonforge-raspberrypi.yml)
