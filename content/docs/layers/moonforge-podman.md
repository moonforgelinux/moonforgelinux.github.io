---
title: 'meta-moonforge-podman'
type: 'docs'
linkTitle: 'Podman'
weight: 35
description: >
  Adds Podman to a Moonforge image, with container storage configured on the
  persistent data partition and public registry access enabled out of the box.
---

`meta-moonforge-podman` installs and configures [Podman](https://podman.io/), a
daemonless container runtime that is compatible with Docker's CLI and image format.
Like the Docker layer, it redirects container storage to the persistent data partition
to survive reboots of the read-only rootfs.

## What it does

- Installs Podman and its runtime dependencies into the base image.
- Configures Podman's storage location to `/data/containers` on the writable persistent
  partition.
- Adds `ca-certificates` so that images can be pulled from public registries (Docker
  Hub, GitHub Container Registry, etc.) without additional setup.

## Why use it

Podman is a compelling choice for embedded products for two reasons. First, it is
daemonless, containers are launched directly by the calling process rather than
through a long-running background daemon, which reduces the attack surface and
eliminates a potential single point of failure. Second, Podman supports rootless
containers, allowing applications to run container workloads without requiring root
privileges on the host.

If your team already uses Docker images and Compose files, Podman remains compatible:
`podman` accepts the same CLI flags as `docker`, and `podman compose` is also
available.

Choose `meta-moonforge-podman` over `meta-moonforge-docker` when daemon-free operation
or rootless containers are a requirement. If you need both runtimes on the same image,
they can be included together, though this is unusual in practice.

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
      file: kas/include/layer/meta-moonforge-podman.yml
```

No additional `local_conf_header` entries are required.

## Testing

After booting the image, you can verify that Podman can pull and run a container:

```
$ podman run --rm hello-world
```

Run an interactive container:

```
$ podman run -it alpine sh
```

Confirm that image storage is on the data partition:

```
$ podman info | grep graphRoot
graphRoot: /data/containers/storage
```

Test `podman compose` with a minimal Compose file:

```
$ cat > /tmp/compose.yml << EOF
services:
  hello:
    image: hello-world
EOF
$ podman compose -f /tmp/compose.yml up
```

See: [meta-moonforge-podman on GitHub](https://github.com/moonforgelinux/meta-moonforge/blob/main/kas/include/layer/meta-moonforge-podman.yml)
