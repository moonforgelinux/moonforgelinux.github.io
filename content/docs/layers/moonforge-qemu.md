---
title: 'meta-moonforge-qemu'
linkTitle: 'QEMU'
type: 'docs'
weight: 20
description: >
  Adds QEMU x86-64 target support to Moonforge, enabling OS images to be built and
  tested in a virtual machine without physical hardware.
---

`meta-moonforge-qemu` provides the board support configuration needed to build a
Moonforge image that runs under QEMU's x86-64 emulator. It is the fastest way to
iterate on a Moonforge image during development, because the entire build-flash-boot
cycle is replaced by a build-run cycle on your workstation.

## What it does

- Extends the base image with the recipes, distro settings, and `local.conf`
  configurations required for `qemux86-64` targets.
- Provides a WKS (Wic Kickstart) file that describes the partition layout for a QEMU
  virtual disk image.

## Why use it

Running your Moonforge image in QEMU before flashing it to hardware saves significant
development time. The full boot sequence, including the read-only rootfs, overlayfs-etc,
and any systemd services you have added, behaves identically in QEMU and on real
hardware. RAUC A/B update flows can also be tested entirely in QEMU without touching a
physical device.

This layer is also used by Moonforge's own CI pipeline to validate each commit.

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
      file: kas/include/layer/meta-moonforge-qemu.yml

local_conf_header:
  20_meta-moonforge-distro: |
    OVERLAYFS_ETC_DEVICE = "/dev/sda3"
  20_meta-moonforge-qemu: |
    WKS_FILE = "moonforge-image-base-qemux86-64.wks.in"

machine: qemux86-64
```

### Configuration variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OVERLAYFS_ETC_DEVICE` | Yes | Persistent data partition, typically `/dev/sda3`, or `/dev/sda4` when RAUC updates are enabled |
| `WKS_FILE` | Yes | WKS partition layout file for QEMU |

## Building and running

Build the image and OVMF firmware together:

```
$ kas-container --runtime-args "--device=/dev/kvm" \
    shell kas/moonforge-image-base-qemux86-64.yml:kas/common/debug.yml
$ bitbake -c build moonforge-image-base ovmf
```

Decompress the disk image and launch it with `runqemu`:

```
$ bzip2 -dk tmp/deploy/images/qemux86-64/moonforge-image-base-qemux86-64.rootfs.wic.bz2
$ runqemu snapshot kvm nographic slirp ovmf qemux86-64 \
    tmp/deploy/images/qemux86-64/moonforge-image-base-qemux86-64.rootfs.wic
```

The `snapshot` flag tells QEMU to write all disk changes to a temporary overlay rather
than the image file, so each run starts from the same clean state. Remove it if you
want changes (such as `/data` writes) to persist between runs.

The `kvm` flag enables hardware acceleration. If KVM is not available on your build
host (e.g., inside a container without device passthrough), omit the flag and
`--device=/dev/kvm`, the image will still run, just more slowly.

### Logging in

The QEMU image boots to a serial console. Log in as `root` with no password (debug
builds) or the password set in your downstream layer.

### Networking

The `slirp` flag enables user-mode networking. From inside the guest, outbound
connections work immediately. To reach the guest from the host (e.g., SSH), use port
forwarding:

```
$ runqemu snapshot kvm nographic slirp ovmf qemux86-64 \
    tmp/deploy/images/qemux86-64/moonforge-image-base-qemux86-64.rootfs.wic \
    qemuparams="-netdev user,id=net0,hostfwd=tcp::2222-:22 -device virtio-net-pci,netdev=net0"
$ ssh root@localhost -p 2222
```

See: [meta-moonforge-qemu on GitHub](https://github.com/moonforgelinux/meta-moonforge/blob/main/kas/include/layer/meta-moonforge-qemu.yml)
