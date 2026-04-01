---
title: 'FAQ'
linkTitle: 'FAQ'
weight: 90
description: >
  Answers to common questions about what Moonforge is, how it compares to alternatives,
  and how to solve typical build and runtime issues.
---

## What is Moonforge?

Moonforge is a set of well-maintained, composable Yocto layers that lets you assemble
a custom, immutable, production-ready Linux OS for embedded devices or
single-application environments. It is not a general-purpose Linux distribution and it
is not a complete embedded distro you flash as-is, it is a framework you build on top
of.

## Is Moonforge a Linux distribution?

Not in the traditional sense. A traditional Linux distribution (Debian, Fedora, Alpine)
ships a fixed set of packages you install and update at runtime using a package manager.
Moonforge produces immutable OS images: the root filesystem is built once, is read-only
at runtime, and is replaced atomically via A/B partition switching when an update is
applied. There is no package manager running on the device.

## How is Moonforge different from Poky, the Yocto reference distribution?

[Poky](https://docs.yoctoproject.org/ref-manual/terms.html#term-Poky) is Yocto's
reference distribution, a minimal starting point that documents the Yocto workflow.
Moonforge goes further by providing:

- A curated set of feature layers (containerisation, OTA updates, graphics, WebKit
  kiosk) with sensible defaults and tested interactions between them.
- `kas`-based build orchestration with per-layer configuration fragments, eliminating
  the need to manually manage `bblayers.conf` and `local.conf`.
- A CI/CD pipeline that automatically produces OTA bundles, CVE reports, and SBOM
  metadata alongside each OS image.
- A clear upstream/downstream separation model so your product code stays in your own
  repository and survives upstream Moonforge updates cleanly.

## Can I use Moonforge for a commercial product?

Yes. Moonforge is licensed under the
[MIT](https://github.com/moonforgelinux/meta-moonforge/blob/main/LICENSE)
license. The underlying Yocto and OpenEmbedded components carry their own
licenses (MIT, GPLv2, etc.), which are standard for embedded Linux stacks.
Review the SBOM output from your build for a full list of licenses in your
specific image.

For commercial support and consulting, [Igalia](https://www.igalia.com/contact/) offers
professional services.

## What hardware does Moonforge support?

Currently Moonforge ships BSP layers for:

- **QEMU x86-64**, for development and CI, no physical hardware required.
- **Raspberry Pi 4 and 5**, widely available and well-suited for prototyping.

Support for additional hardware is added as separate BSP layers. Contributions of new
BSP layers are welcome, see the
[GitHub repository](https://github.com/moonforgelinux) for how to contribute.

## How long does a first build take?

A first build typically takes **1–3 hours** depending on your CPU, available RAM, and
internet connection speed. Subsequent builds of the same or similar configuration use
the Yocto shared-state cache and typically complete in **5–15 minutes**.

Allocate at least **100 GB** of free disk space for the download and shared-state
cache directories.

## My build failed. Where do I look?

BitBake build failures are logged under:

```
build/tmp/work/<machine>/<recipe>/<version>/temp/log.do_<task>
```

The most common causes are:

- **Missing host dependencies**. Run the build inside `kas-container` rather than
  natively to avoid this.
- **Network timeouts** during source fetching. Retry the build; BitBake resumes from
  where it stopped.
- **Incorrect `local_conf_header` ordering**. Check that numbered prefixes on your
  `local_conf_header` keys place your overrides at a higher number than the upstream
  layer's defaults.

To inspect the fully expanded kas configuration and `local.conf` before building:

```
$ kas-container dump kas/your-image.yml
$ kas-container shell kas/your-image.yml
$ cat conf/local.conf
```

## Can I add packages not available in Moonforge layers?

Yes. Create a downstream layer in your own repository (see
[Customize a Moonforge OS](../tutorials/customize/)) and add a `.bbappend` file for the
`moonforge-image-base` recipe. For example:

```bitbake
IMAGE_FEATURES += "ssh-server-openssh"

CORE_IMAGE_EXTRA_INSTALL += " \
    my-custom-package \
    python3 \
"
```

Any package available in OpenEmbedded layers can be added this way. You can also write
your own recipes for software that is not in any upstream layer.

## How does RAUC A/B updating work?

RAUC maintains two rootfs partitions (slot A and slot B). At any given time, the
device boots from one slot (the "active" slot). When an update is applied:

1. The new OS image is written to the inactive slot.
2. RAUC marks the inactive slot as the next boot target.
3. The device reboots into the new slot.
4. If the new slot boots successfully, RAUC marks it as good and it becomes the new
   active slot.
5. If the boot fails, the bootloader falls back to the old slot automatically.

The old slot is never modified until the next update, so there is always a fallback
available.

## How do I change the URL displayed by the WPE kiosk?

Set `WAYLAND_COG_LAUNCH_URL` in your kas `local_conf_header`:

```yaml
local_conf_header:
  30_meta-moonforge-wpe: |
    WAYLAND_COG_LAUNCH_URL = "https://my-application.example.com"
```

This is a build-time setting. If you need to change the URL at runtime without
rebuilding, read the URL from a file on the persistent data partition (`/data`) using a
custom systemd service or environment file in your downstream layer.

## How do I report a bug or request a feature?

Open an issue on the appropriate repository under
[github.com/moonforgelinux](https://github.com/moonforgelinux). For general questions
and discussion, use
[GitHub Discussions](https://github.com/orgs/moonforgelinux/discussions).
