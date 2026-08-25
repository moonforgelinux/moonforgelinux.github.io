---
title: Modern Yocto Linux Best Practices
linkTitle: Modern Yocto Linux Best Practices
description: An overview of best practices when using Yocto to build your distribution.
date: 2026-07-17
tags: ["moonforge", "yocto"]
categories: ["general"]
author: Martín Abente
---

If you've been working with the [Yocto Project](https://www.yoctoproject.org/) for a while, you already know it's the de facto standard for building custom embedded Linux distributions. What you might not know is how much the tooling around it has improved.

Many teams adopted Yocto years ago and have kept roughly the same workflows ever since, copying setup scripts between projects, letting each developer figure out how to clone layers, and building releases manually on dedicated machines. These were the common patterns at the time, but by now they are just unnecessary friction.

The Yocto community and surrounding ecosystem have introduced tools and practices that significantly improve reproducibility, onboarding, and CI integration. But lack of comprehensive documentation for how to integrate these improvements has likely kept some people from adopting them. This post aims to cover the most important ones and close that gap.

## The Pain Points

Up until very recently, the Yocto documentation and tutorials encouraged developers to work with local files and gave little guidance on project structure. This nudged teams toward a set of common but costly habits: bootstrapping new projects by **copying scripts** from existing ones, leaving each developer to **figure out layer setup on their own**, and **relying on specific machines** for production builds. The result was projects that were fragile to onboard, hard to reproduce, and difficult to scale.

## Modern Approach

The most mature and tested solution to the problem today is [kas](https://kas.readthedocs.io/en/latest/). Kas is an open-source setup and automation tool to better manage Yocto layers. It simplifies the process of configuring a Yocto build environment into a single, declarative configuration that covers all of the issues mentioned before.

Defining your layers, repositories, revisions, and build settings in one place makes it straightforward to spin up new projects and onboard new team members: **a fresh clone and a single command is all it takes to get a working build environment**. Kas also provides container integration and CI/CD tooling out of the box, so the same environment that runs on a **developer's laptop runs identically in your CI pipeline**. And because everything is expressed in YAML, **project definitions can be versioned, diffed, and collaborated on like any other source file**.

Here's what a complete project definition looks like in practice:

```yaml
# kas/derivative-image-base-raspberrypi5.yml
header:
  version: 16
  includes:
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-distro.yml
    - repo: meta-moonforge
      file: kas/include/layer/meta-moonforge-raspberrypi.yml

local_conf_header:
  30_meta-moonforge-raspberrypi: |
    WKS_FILE = "moonforge-image-base-raspberrypi.wks.in"
  20_meta-moonforge-distro: |
    OVERLAYFS_ETC_DEVICE = "/dev/mmcblk0p3"

repos:
  meta-moonforge:
    url: https://github.com/moonforgelinux/meta-moonforge.git
    commit: 628d710b7e076be1daa2376065ea12bb8eeded3a
    branch: main

distro: moonforge
machine: raspberrypi5
```

The above example is enough to build a working image of [Moonforge Linux](https://moonforgelinux.org/) for the Raspberry Pi 5. If you are curious about Moonforge, check out this [tutorial](https://moonforgelinux.org/docs/tutorials/derive/) for how to create your own distribution.

An official alternative to kas, called [bitbake-setup](https://docs.yoctoproject.org/bitbake/dev/bitbake-user-manual/bitbake-user-manual-environment-setup.html), has recently been released by the maintainers of the Yocto project. Although still not as feature rich as kas, being part of BitBake makes bitbake-setup worth exploring and considering. A recent [comparison](https://sigma-star.at/blog/2025/10/the-evolving-landscape-of-yocto-project-setup-bitbake-setup-vs.-kas/) from Richard Weinberger clarifies the similarities and differences.

## Containerized Environments

Using containerized build environments is now common practice across Desktop Operating Systems. Containers enable members of a team to use the same environment regardless of what they are running on their development machines.

However, in the world of Yocto, many teams still follow the old practice of installing packages locally and then struggling to be able to reproduce the same conditions when building on a different machine. Nowadays, the best approach is to use a container that includes all the needed build dependencies at the desired version.

Kas integrates naturally with containers by using `kas-container`:

```sh
$ kas-container build kas/derivative-image-base-raspberrypi5.yml
```

This containerized approach fits perfectly for CI workflows as well, so the conditions are *always* the same regardless of the machine or the stage of development.

## CI/CD Integration and Build Automation

The manual way of installing dependencies has meant that many teams have stayed away from CI/CD when using Yocto, building releases manually on developer machines. This tends to hold teams back, as errors are discovered too late and then problems might be hard to reproduce and solve.

Running automated pipelines like GitHub Actions, GitLab CI, or even Jenkins with the same containers as those used locally by developers ensures consistency and reduces human error.

Once these pipelines are enabled, remote computing resources and processes can be shared across multiple projects within the organization, and made available via reusable GitHub actions. As can be seen in this example:

```yaml
# .github/workflows/main.yml
jobs:
  build:
    runs-on: [self-hosted, builder]
    steps:
      - uses: actions/checkout@v6
      - uses: moonforgelinux/build-moonforge-action@v0.1.1
        with:
          kas_file: kas/derivative-image-base-raspberrypi5.yml
          dl_dir: /home/github-runner/kas/cache/downloads
          sstate_dir: /home/github-runner/kas/cache/sstate-cache
          image_id: ${{ github.ref_name }}
          image_version: ${{ github.run_id }}
      - uses: moonforgelinux/upload-moonforge-action@v0.2.1
        with:
          host_base: ${{ secrets.S3_HOST_BASE }}
          access_key: ${{ secrets.S3_ACCESS_KEY }}
          secret_key: ${{ secrets.S3_SECRET_KEY }}
          bucket: ${{ secrets.S3_BUCKET }}
          source: build/tmp/deploy/images
          destination: 'builds/${{ github.run_id }}/'
          exclude: '*'
          include: '*.wic.bz2'
          use_https: true
```

The above example shows how Moonforge's GitHub actions can be reused to build and publish OS images. These actions are available to all Moonforge derivative projects. Check this [tutorial](https://moonforgelinux.org/docs/tutorials/ci/) for how to reuse these actions with your own distribution.

## Extending

The OpenEmbedded build system allows you to isolate different types of customizations into multiple layers. Each layers can provide a specific solution that is reusable and extensible. In general, layers can:

* Provide recipes for new software.
* Modify recipes from existing layers.
* Provide new configuration files for machines and distributions.

What layers can't do:

* Modify existing distribution or machine configurations.
* Manage dependencies on external repositories and layers.
* Set sensible defaults to the local configuration.

All of these limitations can be overcome by a sensible combination of layers and kas fragments. Simply put, fragments are YAML files that can be reused by other kas files.

Kas support for *include* directives can help structure these fragments in reusable blocks. These fragments can also be pulled from remote repositories. With this, derivative projects can reuse existing combinations of layers and fragments to build their own distributions, reducing duplication, manual steps, increasing reproducibility and having a clear upstream and downstream separation.

The next two examples illustrate this:

```yaml
# kas/include/repo/meta-raspberrypi.yml
header:
  version: 16

repos:
  meta-raspberrypi:
    url: https://git.yoctoproject.org/meta-raspberrypi
    commit: 5240b5c200e594b494a7f1a8f9d81e7c09bc8939
    branch: scarthgap
```

The above example shows how external layers can be made available to other fragments, keeping them pinned to a specific upstream release to ensure reproducibility.

```yaml
# kas/include/layer/meta-moonforge-raspberrypi.yml
header:
  version: 16
  includes:
    - kas/include/repo/meta-lts-mixins.yml
    - kas/include/repo/meta-raspberrypi.yml
    - kas/include/layer/meta-moonforge-distro.yml

local_conf_header:
  20_meta-moonforge-raspberrypi: |
    ENABLE_UART = "1"
    RPI_USE_U_BOOT = "1"
    LICENSE_FLAGS_ACCEPTED += "synaptics-killswitch"

repos:
  meta-moonforge:
    layers:
      meta-moonforge-raspberrypi:

```

The above example shows how having separate fragments for each layer can address the above limitations, like managing dependencies on external layers, setting sensible defaults to the local configuration and more.

## Conclusion

Yocto remains the de facto standard for embedded Linux customization, but teams have to catch up with evolving workflows. Modern tooling and processes makes Yocto more reproducible, easier to use, maintain and scale.

As a last recap, remember to:

* Use kas for build configuration.
* Use containers for build environments.
* Automate builds in CI/CD.
* Keep layers modular and provide reusable fragments.
* Track everything in version control.

And avoid:

* Manual setup scripts.
* Host-dependent builds.
* Unpinned dependencies.

