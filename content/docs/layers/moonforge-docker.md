---
title: 'meta-moonforge-docker'
linkTitle: 'Docker'
weight: 30
description: >
  Adds Docker engine and docker-compose to a Moonforge image, with container storage
  configured on the persistent data partition.
---

`meta-moonforge-docker` installs and configures the Docker container runtime. Because
the root filesystem is read-only, this layer redirects Docker's image and container
storage to the persistent data partition so that pulled images and running containers
survive reboots.

## What it does

- Installs the Docker engine and its dependencies into the base image.
- Provides a custom recipe that configures Docker's data root
  (`/var/lib/docker` → `/data/docker`) so that container images are stored on the
  writable persistent partition rather than the read-only rootfs.
- Includes `docker-compose` (the `docker compose` plugin) for multi-container
  application definitions.

## Why use it

Docker is the most widely supported container runtime and the natural choice when your
team already uses Docker images and Compose files for development and deployment. If you
prefer a daemonless runtime, see [`meta-moonforge-podman`](../moonforge-podman/) instead.

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
      file: kas/include/layer/meta-moonforge-docker.yml
```

No additional `local_conf_header` entries are required. The layer's kas fragment sets
sensible defaults for container storage automatically.

## Testing

After booting the image, you can verify that Docker is running:

```
$ systemctl status docker
$ docker info
```

Pull and run a test container:

```
$ docker run --rm hello-world
$ docker run -it ubuntu bash
```

Verify that `docker compose` is available:

```
$ cat > /tmp/docker-compose.yml << EOF
services:
  hello:
    image: hello-world
EOF
$ docker compose -f /tmp/docker-compose.yml up
```

Confirm that pulled images are stored on the data partition:

```
$ ls /data/docker/image/
```

See: [meta-moonforge-docker on GitHub](https://github.com/moonforgelinux/meta-moonforge/blob/main/kas/include/layer/meta-moonforge-docker.yml)
