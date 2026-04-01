---
title: Design
linkTitle: Design
weight: 150
description: >
  Architecture, partition layout, and release policy for Moonforge.
---

> **Note:** This is a living document that describes the current design policy for Moonforge.
> For the high-level goals that motivate these decisions, see the [Goals](/goals/) page.

## Architecture overview

A Moonforge OS image is assembled by composing a set of Yocto layers managed with
[kas](https://kas.readthedocs.io/). The layers are grouped into three categories:

- **Distro layer** (`meta-moonforge-distro`): the mandatory base. Establishes the
  distribution identity, partition layout, and the read-only root filesystem policy.
- **BSP layers** (`meta-moonforge-qemu`, `meta-moonforge-raspberrypi`, …):hardware
  support. Exactly one BSP layer is included per image.
- **Feature layers** (`meta-moonforge-docker`, `meta-moonforge-rauc`, …): optional
  capabilities. Any combination may be included.

Every layer ships a kas YAML fragment that declares its external repository dependencies,
activates the required BitBake layers, and sets sensible default `local.conf` values.
This means adding a feature to your image is a one-line change to your kas configuration
file.

Your own product-specific code should live in a **downstream layer** (conventionally named
`meta-<your-product>`), tracked in your own repository and referencing 
upstream Moonforge layers by commit hash. This keeps upstream and downstream changes
cleanly separated.

### Read-only root filesystem

One of the core principles in Moonforge is that it is an **immutable OS**.
This means, that the root filesystem is mounted read-only (`read-only-rootfs`
distro feature).

This has several advantages, including:
- **Predictable behaviour**: no package manager runs on the device at runtime, so the
  running OS is always identical to what was tested and released.
- **Reliability and Stability**: the read only root filesystem cannot be easily
  corrupted by user error or failed application updates.
- **Enhanced Security**: malicious software cannot easily make persistent
  changes to system binaries.

To allow for configurations to be set independent of the image build, the `/etc`
directory is managed via `overlayfs-etc`, which overlays a writable copy stored on the
persistent data partition on top of the read-only `/etc` from the rootfs. This allows
runtime configuration changes (e.g., network settings written by `systemd-networkd`) to
survive reboots without compromising the immutability of the base image.

The `OVERLAYFS_ETC_DEVICE` variable in your kas configuration must point to the data
partition device node for the target board (e.g., `/dev/mmcblk0p3` for a Raspberry Pi SD
card, `/dev/mmcblk0p4` for a Raspberry Pi configured for RAUC updates, `/dev/sda3` for a
QEMU virtual disk).

### Partition layout for resilient updates

When configured for getting automatic updates, Moonforge uses the following
partition scheme:

| Partition | Contents | Mount point | Writable |
|-----------|----------|-------------|----------|
| Boot | Bootloader, kernel, initramfs | `/boot` | No |
| Rootfs A | OS image (slot A) | `/` | No |
| Rootfs B | OS image (slot B, for A/B updates) | - | No |
| Data | Persistent application data | `/data` | Yes |

This is used for **A/B updates**, which perform the update on the currently
inactive partition, and on a failed reboot can fall back to the previously
working image.

OTA updates are delivered as signed RAUC bundles. The CI/CD pipeline automatically builds
a bundle alongside each OS image, so every release is always accompanied by an update
artefact that can be pushed to deployed devices.

### CI/CD pipeline outputs

A Moonforge build produces the following artefacts:

| Artefact | Description |
|----------|-------------|
| OS image (`.wic.bz2`) | The partitioned disk image, ready to flash |
| RAUC bundle (`.raucb`) | A signed OTA update bundle for the same release |
| CVE report | A list of known vulnerabilities in the image packages |
| SBOM (`.spdx.json`) | A Software Bill of Materials in SPDX format |

The CVE report and SBOM are generated using standard Yocto recipes and require no
additional tooling beyond a normal Moonforge build. The kas fragments needed for
enabling them are located under `meta-monforge/kas/common`.

## Releases

**NOTE**: This documentation is currently aspirational. For now we only have the main branch.

### Channels

Moonforge releases follow three channels:

| Channel | Branch | Yocto base | Purpose |
|---------|--------|------------|---------|
| `stable` | `stable` | Current LTS | Production use |
| `next` | `next` | Upcoming LTS | Integration testing |
| `main` | `main` | Development tip | Active development |

Each channel tracks exactly one version of Yocto. When a new Yocto LTS release is
finalized, the `stable` branch is archived and `next` becomes the new `stable`.

### Cadence

The **stable** channel releases on a monthly cadence. The `next` and `main` channels
do not have scheduled releases; they receive commits continuously and are built by CI
on every push.

### Support

Only the **stable** channel is supported. If you are building a product intended for
long-term deployment, always pin your downstream kas configuration to a specific commit
on the `stable` branch. Using the `main` branch in production is not recommended.

### Pinning a release

In your kas configuration file, reference a specific commit rather than a branch name:

```yaml
repos:
  meta-moonforge:
    url: https://github.com/moonforgelinux/meta-moonforge.git
    commit: ce8ff3de25dda42215b7c8dfd201fd39c3960b1f
    branch: stable
```

Pinning to a commit ensures that your build is fully reproducible. When you want to
update, change the `commit` value to the latest stable release commit and rebuild.
