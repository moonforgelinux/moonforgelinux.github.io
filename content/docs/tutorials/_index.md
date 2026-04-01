---
title: "Tutorials"
linkTitle: "Tutorials"
weight: 30
description: >
  Step-by-step guides for building and customising a Moonforge-based OS image.
---

These tutorials walk you through the most common Moonforge workflows from start to
finish. Each tutorial builds on the previous one, so if you are new to Moonforge it is
best to work through them in order.

## Before you begin

All tutorials assume the following tools are installed on your Linux host:

- **Docker** or **Podman**, used by `kas-container` to provide a hermetic build
  environment. See the [Docker install guide](https://docs.docker.com/engine/install/)
  or [Podman install guide](https://podman.io/getting-started/installation).
- **kas 5.0**, the build orchestration tool:

  ```
  pip install --user kas==5.0
  kas-container --version
  ```

- **bmaptool**, for flashing SD cards (Raspberry Pi tutorials only):

  ```
  # Fedora
  sudo dnf install bmap-tools

  # Ubuntu / Debian
  sudo apt install bmap-tools
  ```

You will need about **100 GB** of free disk space for downloads and the Yocto
shared-state cache on your first build. This first build usually takes a long
time, but subsequent builds are significantly faster thanks to caching.

## Tutorials

### [Create a Moonforge OS](./derive/)

Start here. You will set up a workspace, write your first kas configuration file, build
a base OS image for the Raspberry Pi 5 (or QEMU), and boot it.

**Time:** 1–3 hours (mostly build time on first run).

### [Customize a Moonforge OS](./customize/)

Extend the base image you built in the previous tutorial. You will add feature layers
(Docker, RAUC), write a custom downstream BitBake layer, add packages, and define your
own distribution name and version.

**Time:** 30–60 minutes (build cache is warm from the previous tutorial).

## What you will build

By the end of both tutorials you will have:

- A working `meta-derivative` repository structure you can use as the foundation for
  your own product.
- A reproducible kas configuration file that pins upstream Moonforge to a specific
  commit.
- An immutable OS image with a read-only root filesystem and a persistent data
  partition.
- A RAUC OTA update bundle generated automatically alongside the image.
- A custom distro name and version visible in `/etc/os-release` on the running device.
