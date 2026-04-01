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

### Docker or Podman

`kas-container` uses a container engine to provide a hermetic Yocto build environment.
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

### kas

`kas` is the build orchestration tool used by all Moonforge projects:

```
pip install --user kas==5.0
kas-container --version
```

If `kas-container` is not found after installation, ensure `~/.local/bin` is in your
`PATH`:

```
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
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

A Moonforge-based product lives in a downstream repository. Clone the
`meta-derivative` example as a starting point:

```
git clone https://github.com/moonforgelinux/meta-derivative.git
cd meta-derivative
```

Or create a fresh repository from scratch:

```
mkdir meta-myproduct && cd meta-myproduct
git init
mkdir kas
```

## 4. Write a kas configuration file

Create a kas configuration file that targets QEMU:

```
cat > kas/myproduct-image-qemux86-64.yml << 'EOF'
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

repos:
  meta-moonforge:
    url: https://github.com/moonforgelinux/meta-moonforge.git
    commit: 42b1aeefb1327785c48925e62719fa13d55c8e13
    branch: main

machine: qemux86-64
EOF
```

## 5. Set up the cache directories

Export the cache paths before building so that downloads and shared-state objects are
stored outside the build tree and survive `cleansstate` or directory removal:

```
export KAS_CONTAINER_ENGINE=docker   # or: podman
export DL_DIR=$HOME/moonforge-workspace/cache/downloads
export SSTATE_DIR=$HOME/moonforge-workspace/cache/sstate-cache
```

Add these exports to your shell profile if you want them to persist across sessions.

## 6. Build the image

```
kas-container build kas/myproduct-image-qemux86-64.yml
```

The first build downloads all sources and populates the shared-state cache. Expect it
to take **1–3 hours** depending on your hardware. Subsequent builds of the same
configuration typically complete in **5–15 minutes** thanks to the cache.

## 7. Boot the image in QEMU

Decompress the disk image and launch it:

```
bzip2 -dk build/tmp/deploy/images/qemux86-64/moonforge-image-base-qemux86-64.rootfs.wic.bz2

kas-container --runtime-args "--device=/dev/kvm" \
  shell kas/myproduct-image-qemux86-64.yml -- \
  runqemu snapshot kvm nographic slirp ovmf qemux86-64 \
  tmp/deploy/images/qemux86-64/moonforge-image-base-qemux86-64.rootfs.wic
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
