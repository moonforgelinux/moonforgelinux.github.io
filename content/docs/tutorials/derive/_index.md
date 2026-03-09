---
title: Create a Moonforge OS
description: How to create your Moonforge-based OS.
weight: 100
---

This document explains how to reproduce this example repository, that is, how to build a custom Linux distribution based on Moonforge.

## Setup the build environment

In order to prepare the host build machine, the following system tools are required:

* [docker](https://www.docker.com/) or [podman](https://podman.io/)
* python3-pip

To install *docker* see the official documentation for common Linux distributions like [Fedora](https://docs.docker.com/engine/install/fedora/) or [Ubuntu](https://docs.docker.com/engine/install/ubuntu/). If you're using a recent Fedora release, it's likely that *podman* is already available.

To ensure truly reproducible builds, Moonforge uses [bitbake](https://docs.yoctoproject.org/bitbake/) in a containerized environment provided by [kas-container](https://kas.readthedocs.io/en/latest/). Therefore the following step is also required:

```sh
$ pip install --user kas==5.0
$ kas-container --version
```

It's recommended to create a workspace directory to hold both project and a persistent cache:

```sh
$ mkdir workspace && cd workspace
$ mkdir cache
$ mkdir meta-derivative # or, e.g., meta-name-of-company
```

Note that the *cache* directory will hold both the downloads and shared-state cache directories, which will significantly speed up subsequent builds.

## Describe the image

Moonforge uses kas configuration files to determine what goes into an image, therefore, it's the recommended way to describe these images. Write a *kas* configuration file as follows:

```sh
$ cd meta-derivative
$ mkdir kas
$ vi kas/derivative-image-base-raspberrypi5.yml
```

The following kas file sets up the bare minimum, which is, a reference to a specific release of the *meta-moonforge* repository and the basic layers needed to build an image for the Raspberry Pi 5:

**kas/derivative-image-base-raspberrypi5.yml:**

```yml
header:
  version: 16
  includes:
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-distro.yml
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-raspberrypi.yml

local_conf_header:
  30_meta-moonforge-raspberrypi: |
    # Specifies description file for partitioning the image
    WKS_FILE = "moonforge-image-base-raspberrypi.wks.in"
  20_meta-moonforge-distro: |
    # Sets up persistent data partition
    OVERLAYFS_ETC_DEVICE = "/dev/mmcblk0p3"
    # Sets minimum size of the data partition
    IMAGE_DATA_MIN_SIZE = "4096M"

repos:
  meta-moonforge:
    url: https://github.com/moonforgelinux/meta-moonforge.git
    commit: ce8ff3de25dda42215b7c8dfd201fd39c3960b1f
    branch: main

machine: raspberrypi5
```

Note that the configurations under *local_conf_header* are needed to adapt the base image to the Raspberry Pi 5. The numbered prefixes *30_* and *20_* are needed to ensure that these configurations take precedence over the default configurations of these layers.

Check the [meta-moonforge](https://github.com/moonforgelinux/meta-moonforge) repository for more [examples](https://github.com/moonforgelinux/meta-moonforge/blob/main/kas/examples).

## Build the image

Before building, both the download and shared-stated cache directories must be set via environment variables:

```sh
$ cd meta-derivative
$ export KAS_CONTAINER_ENGINE=docker
$ export DL_DIR=$PWD/../cache/downloads/
$ export SSTATE_DIR=$PWD/../cache/sstate-cache/
```

Then, use *kas-container* to build the image:

```sh
$ cd meta-derivative
$ kas-container build kas/derivative-image-base-raspberrypi5.yml
```

## Test the image

Once the image is successfully built, use a tool like [bmaptool](https://docs.yoctoproject.org/dev-manual/bmaptool.html) to flash the SD card:

```sh
$ cd meta-derivative
$ sudo bmaptool copy ./build/tmp/deploy/images/raspberrypi5/moonforge-image-base-raspberrypi5.rootfs.wic.bz2 /dev/sdX
```

Insert the flashed SD card into the Raspberry Pi 5 and boot it up.
