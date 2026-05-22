---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 5
description: >
  Install the required tools, set up your workspace, and build your first Moonforge
  image.
---

This page gets you from zero to a running Moonforge image as quickly as possible. It
uses the QEMU x86-64 target, so that you do not need any physical hardware to complete
it.

## 1. Install prerequisites

You need a Linux host with the following tools installed.

### Moonforge-cli

The `moonforge` command line tool simplifies the initialization and build of
a Moonforge project.

See the [`moonforge` documentation](./moonforge-cli/) for installation
instructions and the list of available commands.

### Docker or Podman

`moonforge build` uses a container engine to provide a hermetic Yocto build environment.
Install either Docker or Podman:

```
# Fedora
sudo dnf install docker
sudo systemctl enable --now docker

# Ubuntu
sudo apt update && sudo apt install docker.io
sudo systemctl enable --now docker
```

If you prefer Podman (daemonless, no root required):

```
# Fedora (Podman is usually pre-installed)
sudo dnf install podman

# Ubuntu
sudo apt install podman
```

### Disk space

Allocate at least **100 GB** of free disk space for the first build. Most of this is
the Yocto shared-state cache and download directory, which are reused across builds.

## 2. Set up your workspace

Create a workspace directory to hold your project and the build cache:

```
mkdir -p ~/moonforge-workspace/cache
cd ~/moonforge-workspace
```

## 3. Create your derivative repository

A Moonforge-based product lives in a downstream repository. Use the
`moonforge init` command to initialize a Moonforge project:

```
moonforge init --name=myproduct meta-myproduct
```

The command amove will create a new Moonforge project repository under the
`meta-myproduct` directory, targeting QEMU.

Inside the repository you will find a kas configuration file called `kas/myproduct-image-base-qemux86-64.yml`.

## 4. Set up the cache directories

Export the cache paths before building so that downloads and shared-state objects are
stored outside the build tree and survive `cleansstate` or directory removal:

```
cd meta-myproduct
moonforge config set container.engine docker  # or: podman
moonforge config set build.download_dir $HOME/moonforge-workspace/cache/downloads
moonforge config set build.sstate_dir $HOME/moonforge-workspace/cache/sstate-cache
```

The configuration will be stored inside the project directory.

You can also configure `moonforge`'s default settings for your user, so they
will apply to all Moonforge projects.

## 6. Build the image

```
moonforge build meta-myproduct
```

The first build downloads all sources and populates the shared-state cache. Expect it
to take **1–3 hours** depending on your hardware. Subsequent builds of the same
configuration typically complete in **5–15 minutes** thanks to the cache.

## 7. Boot the image in QEMU

Decompress the disk image and open a shell inside the Moonforge build
environment:

```
bzip2 -dk build/tmp/deploy/images/qemux86-64/moonforge-image-base-qemux86-64.rootfs.wic.bz2

moonforge shell --runtime-args "--device=/dev/kvm" meta-moonforge
```

Then launch the disk image with `runqemu`:

```
runqemu snapshot kvm nographic slirp ovmf qemux86-64 /build/tmp/deploy/images/qemux86-64/moonforge-image-base-qemux86-64.rootfs.wic
```

Log in as `root` (no password on debug builds). Explore the running system:

```
$ mount | grep ' / '        # confirm read-only rootfs
$ cat /etc/os-release       # confirm distribution identity
$ df -h /data               # confirm persistent data partition is mounted
```

Press `Ctrl-A X` to exit the QEMU serial console.

## What's next

- Add feature layers (Docker, RAUC, a graphical session) - see
  [Customize a Moonforge OS](../tutorials/customize/).
- Build for real hardware - see
  [meta-moonforge-raspberrypi](../layers/moonforge-raspberrypi/).
- Enable OTA updates - see
  [meta-moonforge-rauc](../layers/moonforge-rauc/) and
  [meta-moonforge-rauc-update](../layers/moonforge-rauc-update/).
- Understand the release model and partition layout in depth - see
  [Design](../design/).
