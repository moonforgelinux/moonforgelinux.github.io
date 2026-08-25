---
title: "Let's talk about Moonforge"
linkTitle: "Let's talk about Moonforge"
date: 2026-06-17
tags: ["moonforge"]
categories: ["general"]
author: Emmanuele Bassi
description: >
  What Moonforge is, and what it does.
---


Back in March, Igalia [announced Moonforge](https://www.igalia.com/2026/03/09/Introducing-Moonforge-A-Yocto-Based-Linux-OS.html), a project we started working on in 2025. It's been quite the rollercoaster, and now that the dust has settled, it's as good a time as any to talk a bit about what Moonforge is, its goal, and its constraints.

Of course, as soon as somebody announces a new Linux-based OS, folks immediately think it's a new general purpose Linux distribution, as that's the [square shaped hole](https://www.youtube.com/watch?v=cUbIkNUFs-4) where everything OS-related ends up. So, first things first, let's get a couple of things out of the way about [Moonforge](https://moonforgelinux.org):

- Moonforge is **not** a general purpose Linux distribution
- Moonforge is **not** an embedded Linux distribution

### What is Moonforge

Moonforge is a set of feature-based, well-maintained layers for [Yocto](https://yoctoproject.org), that allows you to assemble your own OS for embedded devices, or single-application environments, with specific emphasys on immutable, read-only root file system OS images that are easy to deploy and update, through tight integration with CI/CD pipelines.

### Why?

Creating a whole new OS image out of whole cloth is not as hard as it used to be; on the desktop (and devices where you control the hardware), you can reasonably [get away](https://store.steampowered.com/steamos/) with using existing Linux distributions, filing off the serial numbers, and removing any extant packaging mechanism; or you can rely on the [containerised tech stack](https://universal-blue.org/), and boot into it.

When it comes to embedded platforms, on the other hand, you're still very much working on bespoke, artisanal, locally sourced, organic operating systems. A good number of device manufacturers coalesced their [BSPs](https://en.wikipedia.org/wiki/Board_support_package) around the [Yocto Project](https://www.yoctoproject.org/) and [OpenEmbedded](https://www.openembedded.org/wiki/Main_Page), which simplifies adaptations, but you're still supposed to build the thing mostly as a one off.

While Yocto has improved leaps and bounds over the past 15 years, putting together an OS image, especially when it comes to bundling features while keeping the overall size of the base image down, is still an exercise in artisanal knowledge.

### A little detour: Poky

Twenty years ago, a consultancy called OpenedHand started working on a project that took the work done by OpenEmbedded, providing a good set of defaults and layers, in order to create a "reference distribution" that would help people getting started with their own project. That reference was called [Poky](https://web.archive.org/web/20070402231645/http://projects.o-hand.com/poky).

These days, Poky exists as part of the Yocto Project, and it's still the reference distribution for it, but since it's part of Yocto, it has to abide to the basic constraint of the project: you still need to set up your OS using shell scripts and copy-pasting layers and recipes inside your own repository. The Yocto project is working on [a setup tool](https://github.com/kanavin/bitbake/commits/akanavin/bitbake-setup) to
simplify those steps, but there are alternatives…

### Another little detour: Kas

One alternative is [kas](https://kas.readthedocs.io/en/latest/), a tool that allows you to generate the `local.conf` configuration file used by bitbake through various YAML fragments exported by each layer you're interested in, as well as additional fragments that can be used to set up customised environments.

Another feature of kas is that it can spin up the build environment inside a container, which simplifies enourmously its set up time. It avoids unadvertedly contaminating the build, and it makes it very easy to run the build on CI/CD pipelines that already rely on containers.

### What Moonforge provides

Moonforge lets you create a new OS in minutes, selecting a series of features you care about from various [available layers](https://moonforgelinux.org/docs/layers/).

Each layer provides a single feature, like:

- support for a specific architecture or device (QEMU x86\_64, RaspberryPi)
- containerisation (through Docker or Podman)
- A/B updates (through RAUC, systemd-sysupdate, and more)
- graphical session, using Weston
- a [WPE](https://webkit.org/wpe/) environment

Every layer comes with its own kas fragment, which describes what the layer needs to add to the project configuration in order to function.

Since every layer is isolated, we can reason about their dependencies and interactions, and we can combine them into a final, custom product.

Through various tools, including kas, we can set up a Moonforge project that generates and validates OS images as the result of a CI/CD pipeline on platforms like GitLab, GitHub, and BitBucket; OS updates are also generated as part of that pipeline, just as comprehensive CVE reports and Software Bill of Materials (SBOM) through custom Yocto recipes.

More importantly, Moonforge can act both as a reference when it comes to hardware enablement and support for BSPs; and as a reference when building applications that need to interact with specific features coming from a board.

While this is the beginning of the project, it's already fairly usable; we are planning a lot more in this space, so keep an eye out on [the repository](https://github.com/moonforgelinux).

### Trying Moonforge out

If you want to check out Moonforge, you should checkout our [tutorials](https://moonforgelinux.org/docs/tutorials/), as well as the [meta-derivative](https://github.com/moonforgelinux/meta-derivative/) repository, which should give you a good overview on how Moonforge works, and how you can use it.
