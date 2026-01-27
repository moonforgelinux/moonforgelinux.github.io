---
title: 'meta-moonforge-raspberrypi'
type: 'docs'
---
This layer adds support for the Raspberry Pi 5 and 4.

## What it does

* Extends the base image with recipes, distro and local configurations to build images for the Raspberry Pi.

## How to use it

To use this layer, include the following kas file:

```yml
header:
  version: 16
  includes:
    - kas/include/layer/meta-moonforge-raspberrypi.yml

local_conf_header:
  30_meta-moonforge-raspberrypi: |
    WKS_FILE = "moonforge-image-base-raspberrypi.wks.in"

machine: raspberrypi5
```

## Testing

The built image can be flashed with `bmaptool`:

```sh
$ bmaptool copy \
> build/tmp/deploy/images/raspberrypi5/moonforge-image-base-raspberrypi5-0.wic.bz2 \
> /dev/sdX
```

Where `/dev/sdX` is the device of the SD card.
