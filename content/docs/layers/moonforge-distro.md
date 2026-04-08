---
title: 'meta-moonforge-distro'
linkTitle: 'Base distro'
weight: 10
description: >
  The mandatory base layer that establishes the Moonforge distribution identity,
  the default partition scheme, and read-only root filesystem.
---

`meta-moonforge-distro` is the foundation of every Moonforge OS image. It must always
be included, all other layers depend on it.

## What it does

- Defines the `moonforge` Yocto distribution, built on top of a Long-Term Support (LTS)
  OpenEmbedded core release.
- Enables `read-only-rootfs` and `overlayfs-etc` distro features, making the root
  filesystem immutable at runtime while still allowing runtime configuration changes
  (such as network settings) to persist across reboots via an overlay on the data
  partition.
- Creates an empty `/data` mount point for the persistent partition and ensures recipes
  that need a writable location (e.g., container storage, log rotation) are configured
  to use it.
- Provides a custom recipe that post-processes Yocto's raw CVE report output into a
  human-readable format, making it straightforward to review security status as part of
  CI.

## Why use it

Without this layer there is no Moonforge image. It is not optional.

The immutable rootfs policy is central to Moonforge's security and
operational model: all devices will run exactly the OS image that was tested and
released, with no risk of drift from runtime package installation or configuration
changes made directly to `/etc`.

## Dependencies

None inside Moonforge. Internally it depends on openembedded-core, but this is
not exposed to derivative distributions.

## How to use it

Include the following kas fragment in your configuration:

```yaml
header:
  version: 16
  includes:
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-distro.yml

local_conf_header:
  20_meta-moonforge-distro: |
    # Device node of the persistent data partition on your target hardware
    # and target partitioning scheme.
    # Adjust for your board:
    #   Raspberry Pi (SD card): /dev/mmcblk0p3
    #   QEMU (virtual disk):    /dev/sda3
    OVERLAYFS_ETC_DEVICE = "/dev/sda3"
```

### Configuration variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OVERLAYFS_ETC_DEVICE` | Yes | Block device node for the persistent data partition |
| `IMAGE_DATA_MIN_SIZE` | No | Minimum size of the data partition in KiB (default: `4096M`) |

### Verifying the rootfs is read-only

Once booted, you can confirm the root filesystem is mounted read-only:

```
$ mount | grep ' / '
/dev/sda2 on / type ext4 (ro,relatime)
```

And confirm the `/etc` overlay is active:

```
$ mount | grep overlay
overlay on /etc type overlay (rw,relatime,...)
```

See: [meta-moonforge-distro on GitHub](https://github.com/moonforgelinux/meta-moonforge/blob/main/kas/include/layer/meta-moonforge-distro.yml)
