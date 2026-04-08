---
title: 'meta-moonforge-rauc'
type: 'docs'
linkTitle: 'RAUC'
weight: 40
description: >
  Adds RAUC A/B over-the-air update support, including bundle signing and demo
  certificates for local development.
---

`meta-moonforge-rauc` integrates [RAUC](https://rauc.io/), the Robust Auto-Update
Controller, into a Moonforge image. RAUC implements atomic A/B partition switching so
that a failed update always leaves the device in a known-good state.

## What it does

- Installs the RAUC client daemon and its dependencies.
- Provides a common recipe for building signed RAUC update bundles as part of the
  normal image build, every build produces both an OS image and an OTA bundle.
- Includes demo development certificates (key and certificate chain) so that bundles
  can be signed and tested locally without needing to set up a PKI from scratch.
- Configures the image's bootloader integration (via RAUC's bootloader backends) to
  mark slots as good or bad after an update attempt.

## Why use it

Reliable OTA updates are non-negotiable for any device deployed in the field. RAUC's
A/B approach means:

- **No bricking on bad updates**, if the new image fails to boot correctly, the
  bootloader falls back to the previous slot automatically.
- **Atomic delivery**, the update bundle is verified and applied as a unit; there is
  no partial-update state.
- **Cryptographic verification**, bundles are signed with an X.509 certificate chain,
  and the device verifies the signature before applying any update.

## Dependencies

- [`meta-moonforge-distro`](../moonforge-distro/), required by all layers.
- A BSP layer (`meta-moonforge-qemu` or `meta-moonforge-raspberrypi`), the partition
  layout provided by the BSP layer must include an A and B rootfs slot.

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
      file: kas/include/layer/meta-moonforge-rauc.yml

  # This layer requires a RAUC specific partitioning scheme. For example:
  40_meta-moonforge-rauc-raspberrypi: |
    # Sets the description file for partitioning the image
    WKS_FILE = "moonforge-image-rauc-raspberrypi.wks.in"
```

Replace `meta-moonforge-raspberrypi.yml` with `meta-moonforge-qemu.yml` for local
development, or the corresponding board support for other hardware.

## Signing bundles

### Using the demo certificates (development only)

The layer ships self-signed demo certificates. They are suitable for local testing
but must **not** be used in production.

The demo certificates are used automatically when the relevant environment variables
(`RAUC_KEY_FILE`, `RAUC_CERT_FILE`, `RAUC_KEYRING_FILE`) are not set.
No additional configuration is required for local development.

### Using your own certificates (production)

You can generate a certificate authority and signing certificate using the
`openssl-ca.sh` script provided by `meta-rauc`:

```
$ cd meta-rauc/scripts
$ ./openssl-ca.sh
```

This creates an `openssl-ca/` directory containing:

| File | Description |
|------|-------------|
| `openssl-ca/dev/private/development-1.key.pem` | Private signing key |
| `openssl-ca/dev/development-1.cert.pem` | Public signing certificate |
| `openssl-ca/dev/ca.cert.pem` | CA keyring (installed on devices) |

Store the private key securely (e.g., in a secrets manager or HSM). Then export the
paths before building:

```
$ export RAUC_KEY_FILE=/path/to/development-1.key.pem
$ export RAUC_CERT_FILE=/path/to/development-1.cert.pem
$ export RAUC_KEYRING_FILE=/path/to/ca.cert.pem
```

Then pass them to `kas-container` like this:

```
kas-container --runtime-args "-e RAUC_KEY_FILE -e RAUC_CERT_FILE -e RAUC_KEYRING_FILE"
```

### Rotating certificates

To rotate signing certificates without re-flashing devices, install the new CA keyring
via a RAUC bundle signed with the old certificate before the old certificate expires.
Plan this rotation well in advance, there is no recovery path if the certificate on
deployed devices expires before the keyring update is applied.

## Testing an update

See [`meta-moonforge-rauc-update`](../moonforge-rauc-update/) for a complete walkthrough
of building a bundle, serving it from a local HTTP server, and applying it to a running
QEMU instance.

See: [meta-moonforge-rauc on GitHub](https://github.com/moonforgelinux/meta-moonforge/blob/main/kas/include/layer/meta-moonforge-rauc.yml)
