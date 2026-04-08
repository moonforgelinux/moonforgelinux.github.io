---
title: 'Available Layers'
linkTitle: 'Layers'
weight: 20
description: >
  A guide to the available Moonforge layers and how to compose them into an OS image.
---

Moonforge layers are the building blocks of your OS image. Each layer encapsulates a
single, well-defined feature. Layers are designed to be combined freely, you include
only what your product needs, and the result is an image that contains nothing more.

## How layers work

Every layer ships a kas YAML fragment alongside its BitBake recipes. The fragment
declares:

- Which external repositories the layer depends on.
- Which BitBake layers within those repositories must be activated.
- Default `local.conf` values the layer requires (overridable in your own fragment).

When you include a layer's kas fragment in your configuration, all of its dependencies
are resolved automatically. You do not need to manually manage `bblayers.conf` or
hunt down transitive dependencies.

## Layer overview

| Layer | Category | What it adds |
|-------|----------|--------------|
| [`meta-moonforge-distro`](./moonforge-distro/) | Base | Read-only rootfs, overlayfs-etc, CVE reports. **Required.** |
| [`meta-moonforge-qemu`](./moonforge-qemu/) | BSP | QEMU x86-64 target support |
| [`meta-moonforge-raspberrypi`](./moonforge-raspberrypi/) | BSP | Raspberry Pi 4 and 5 support |
| [`meta-moonforge-docker`](./moonforge-docker/) | Feature | Docker engine and docker-compose |
| [`meta-moonforge-podman`](./moonforge-podman/) | Feature | Podman and container registry access |
| [`meta-moonforge-graphics`](./moonforge-graphics/) | Feature | Weston compositor for kiosk displays |
| [`meta-moonforge-wpe`](./moonforge-wpe/) | Feature | WPE WebKit browser in fullscreen kiosk mode |
| [`meta-moonforge-rauc`](./moonforge-rauc/) | Feature | RAUC A/B OTA updates with bundle signing |
| [`meta-moonforge-rauc-update`](./moonforge-rauc-update/) | Feature | Automated RAUC update client on a timer |

## Composing layers

A typical kas configuration file looks like this:

```yaml
header:
  version: 16
  includes:
    # Base layer - always required
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-distro.yml
    # BSP layer - choose one for your target hardware
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-raspberrypi.yml
    # Feature layers - include as many as you need
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-docker.yml
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-rauc.yml

local_conf_header:
  20_meta-moonforge-distro: |
    OVERLAYFS_ETC_DEVICE = "/dev/mmcblk0p3"
  30_meta-moonforge-raspberrypi: |
    WKS_FILE = "moonforge-image-base-raspberrypi.wks.in"

repos:
  meta-moonforge:
    url: https://github.com/moonforgelinux/meta-moonforge.git
    commit: 42b1aeefb1327785c48925e62719fa13d55c8e13
    branch: main

machine: raspberrypi5
```

The numbered prefixes on `local_conf_header` keys (e.g., `20_`, `30_`) control the
order in which configuration snippets are written to `local.conf`. Higher numbers take
precedence. Use a value higher than any upstream layer when you need a downstream
override to win.

## Layer dependencies

Some layers depend on others:

- `meta-moonforge-wpe` requires `meta-moonforge-graphics` (Weston must be present for
  WPE to render into a Wayland surface).
- `meta-moonforge-rauc-update` requires `meta-moonforge-rauc` (the update client drives
  the RAUC daemon).

Each layer's documentation page lists its requirement, and the layer kas fragments encode these
dependencies.

## Writing your own layers

When the existing feature layers do not cover your needs, the recommended approach is to
create a downstream layer in your own repository. See
[Customize a Moonforge OS](../tutorials/customize/) for a step-by-step walkthrough,
including how to add packages, extend recipes with `.bbappend` files, and define a
custom distro configuration.
