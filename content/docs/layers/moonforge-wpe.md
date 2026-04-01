---
title: 'meta-moonforge-wpe'
type: 'docs'
linkTitle: 'WPE'
weight: 55
description: >
  Adds WPE WebKit to display a configurable URL in a fullscreen kiosk window via the
  Cog browser.
---

`meta-moonforge-wpe` installs [WPE WebKit](https://webkit.org/wpe/) and the
[Cog](https://github.com/Igalia/cog) browser shell, configured to launch a given URL
in a fullscreen Wayland window at boot.

WPE (Web Platform for Embedded) is a WebKit port designed specifically for 
embedded targets: it is lightweight, GPU-accelerated when the hardware supports it, 
and actively maintained by Igalia.

## What it does

- Installs WPE WebKit, Cog, and their runtime dependencies.
- Provides a custom systemd service that launches Cog automatically at boot and
  navigates to the configured URL.
- Configures Cog to run fullscreen (no address bar, no tabs, no navigation controls)
  in a Wayland/Weston surface.

## Why use it

WPE is the standard WebKit port for embedded Linux devices. It gives you a
production-quality, standards-compliant browser engine in a kiosk shell with minimal
footprint. Common use cases include:

- Digital signage displaying web-based content.
- Kiosk terminals for point-of-sale, check-in, or self-service workflows.
- Industrial HMI panels driven by a web-based UI stack.
- In-vehicle infotainment displays rendering HTML5 applications.

Because WPE renders into a Wayland surface, it integrates cleanly with
`meta-moonforge-graphics` (Weston) without requiring a full desktop environment.

## Dependencies

- [`meta-moonforge-distro`](../moonforge-distro/), required by all layers.
- [`meta-moonforge-graphics`](../moonforge-graphics/), Weston must be running for WPE
  to have a Wayland surface to render into.

## How to use it

```yaml
header:
  version: 16
  includes:
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-distro.yml
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-raspberrypi.yml
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-graphics.yml
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-wpe.yml

local_conf_header:
  20_meta-moonforge-distro: |
    OVERLAYFS_ETC_DEVICE = "/dev/mmcblk0p3"
  30_meta-moonforge-raspberrypi: |
    WKS_FILE = "moonforge-image-base-raspberrypi.wks.in"
  30_meta-moonforge-wpe: |
    WAYLAND_COG_LAUNCH_URL = "https://example.com"

machine: raspberrypi5
```

### Configuration variables

| Variable | Required | Description |
|----------|----------|-------------|
| `WAYLAND_COG_LAUNCH_URL` | Yes | The URL Cog navigates to at boot |

Set `WAYLAND_COG_LAUNCH_URL` to any valid HTTP or HTTPS URL. For development and
testing, you can point it at a local server (e.g., `http://10.0.2.2:8080` when using
QEMU with `slirp` networking).

## Verifying the kiosk display

After booting, you can confirm that Weston and Cog are running:

```
$ systemctl status weston
$ systemctl status wayland-cog-launch
```

If the display is blank, check the Weston log for GPU or DRM errors:

```
$ journalctl -u weston --no-pager | tail -40
```

On hardware without a GPU (e.g., QEMU without `virgl`), WPE falls back to software
rendering, which is functional but slower.

See: [meta-moonforge-wpe on GitHub](https://github.com/moonforgelinux/meta-moonforge/blob/main/kas/include/layer/meta-moonforge-wpe.yml)
