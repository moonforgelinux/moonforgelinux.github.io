---
title: 'meta-moonforge-rauc-update'
type: 'docs'
linkTitle: 'RAUC update'
weight: 45
description: >
  Adds an automated RAUC update client that polls a remote URL for new bundles on a
  configurable timer.
---

`meta-moonforge-rauc-update` provides a lightweight systemd service that periodically
checks a configured URL for a new RAUC bundle and applies it automatically. It is
designed to complement [`meta-moonforge-rauc`](../moonforge-rauc/), which handles the
low-level bundle verification and partition switching.

## What it does

- Provides a custom `rauc-update.service` systemd unit that downloads and installs a
  RAUC bundle from a given URL.
- Runs on a systemd timer, so that devices check for updates automatically at regular
  intervals without any operator intervention.
- Optionally forces a reboot immediately after a successful update, so that the new
  slot is activated as soon as possible.

## Why use it

For products that need a simple, fire-and-forget update mechanism, where devices should
just stay current without requiring any fleet management infrastructure, this layer
provides a minimal but functional solution out of the box. More sophisticated update
orchestration (staged rollouts, canary deployments, rollback policies) can be layered
on top by replacing or wrapping the service from a downstream layer.

## Dependencies

- [`meta-moonforge-distro`](../moonforge-distro/), required by all layers.
- [`meta-moonforge-rauc`](../moonforge-rauc/), the RAUC daemon must be present for
  the update client to drive.

## How to use it

```yaml
header:
  version: 16
  includes:
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-distro.yml
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-rauc.yml
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-rauc-update.yml

local_conf_header:
  40_meta-moonforge-rauc-update: |
    # URL where the device will look for the latest RAUC bundle.
    RAUC_BUNDLE_URL = "https://updates.example.com/LATEST.raucb"
    # Set to "1" to reboot immediately after a successful update.
    RAUC_FORCE_REBOOT_ON_UPDATE = "1"
```

### Configuration variables

| Variable | Required | Description |
|----------|----------|-------------|
| `RAUC_BUNDLE_URL` | Yes | Full URL to the RAUC bundle file (`.raucb`) |
| `RAUC_FORCE_REBOOT_ON_UPDATE` | No | Set to `"1"` to reboot after a successful update |

> **Note:** When `RAUC_FORCE_REBOOT_ON_UPDATE` is `"1"`, any application running on
> the device will be terminated abruptly at reboot time. Applications that cannot
> tolerate sudden shutdown should register systemd inhibitor locks or listen for the
> `PrepareForShutdown` signal to perform a clean shutdown sequence.

## Testing locally with a QEMU instance

This walkthrough tests a full update cycle using QEMU and a local HTTP server.

**Step 1: Build the initial image and bundle:**

```
$ kas-container build kas/moonforge-image-full-qemux86-64.yml
```

**Step 2: Serve the bundle from a local HTTP server:**

```
$ cd meta-moonforge-rauc-update/backend
$ cp ../../build/tmp/deploy/images/qemux86-64/moonforge-bundle-base-qemux86-64.raucb \
    data/LATEST.raucb
$ podman compose up -d
```

This server listens on port `3333` of the host. Inside a QEMU instance launched with
`slirp` networking, the host should be reachable at `10.0.2.2`.

**Step 3: Update the kas configuration to point at the local server:**

```yaml
local_conf_header:
  40_meta-moonforge-rauc-update: |
    RAUC_BUNDLE_URL = "http://10.0.2.2:3333/LATEST.raucb"
    RAUC_FORCE_REBOOT_ON_UPDATE = "1"
```

**Step 4: Boot the image in QEMU and trigger the update:**

```
$ runqemu snapshot kvm nographic slirp ovmf qemux86-64 \
    tmp/deploy/images/qemux86-64/moonforge-image-base-qemux86-64.rootfs.wic
```

Once logged in, manually start the update service and observe the output:

```
$ systemctl start rauc-update.service
$ journalctl -fu rauc-update.service
```

After the bundle is downloaded and verified, the service will reboot the device. On the
next boot, RAUC will switch to the new slot. You can verify this with:

```
$ rauc status
```

**Step 5: Tear down the local server**

```
$ podman compose down --volumes
```

See: [meta-moonforge-rauc-update on GitHub](https://github.com/moonforgelinux/meta-moonforge/blob/main/kas/include/layer/meta-moonforge-rauc-update.yml)
