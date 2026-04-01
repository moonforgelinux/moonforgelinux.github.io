---
title: 'meta-moonforge-graphics'
linkTitle: 'Graphics'
weight: 50
description: >
  Adds the Weston Wayland compositor, configured for kiosk and single-application
  display environments.
---

`meta-moonforge-graphics` installs and configures [Weston](https://gitlab.freedesktop.org/wayland/weston),
the reference Wayland compositor, tuned for kiosk-style deployments where a single
application occupies the full screen.

## What it does

- Installs Weston and its required dependencies, including `gsettings-desktop-schemas`.
- Configures Weston for kiosk mode: the default panel and built-in demos are removed so
  that the compositor presents a bare Wayland surface ready for your application.
- Provides a `weston.ini` base configuration that can be extended from your downstream
  layer.

## Why use it

Embedded products that need a graphical output (digital signage, kiosk terminals,
industrial HMIs, in-vehicle displays) require a compositor to manage the display
surface. Weston is the standard choice in the embedded Linux ecosystem: it is
lightweight, well-maintained, and fully supported by the Yocto/OpenEmbedded layer
ecosystem.

This layer is necessary for [`meta-moonforge-wpe`](../moonforge-wpe/), which adds a
WebKit browser that runs inside the Weston compositor, turning any device into a
web-based kiosk in minutes.

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
      file: kas/include/layer/meta-moonforge-graphics.yml
```

### Extending the Weston configuration

To customise Weston's behaviour from your downstream layer, add a `.bbappend` for the
`weston` recipe and install your own `weston.ini`:

```
meta-derivative-distro/
  recipes-graphics/
    wayland/
      weston-init.bbappend
      files/
        weston.ini
```

**weston-init.bbappend:**

```
FILESEXTRAPATHS:prepend := "${THISDIR}/files:"
```

See: [meta-moonforge-graphics on GitHub](https://github.com/moonforgelinux/meta-moonforge/blob/main/kas/include/layer/meta-moonforge-graphics.yml)
