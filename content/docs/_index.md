---
title: Documentation
linkTitle: Learn more
menu: {main: {weight: 10}}
description: >
  Everything you need to build, customise, and ship your own OS image based on Moonforge.
---

Moonforge is an open-source collection of Yocto layers and build tooling maintained by
[Igalia](https://www.igalia.com). It gives system integrators and product teams a
production-ready starting point for building custom, immutable embedded Linux operating
systems, without duplicating the integration work that every project ends up doing
from scratch.

## Where to start

If you are new to Moonforge, work through the documentation in this order:

1. **[Design](./design/)** - understand the release channels and overall architecture before
   writing any configuration.
2. **[Layers](./layers/)** - browse the available feature layers and decide which ones your
   product needs.
3. **[Tutorials](./tutorials/)** - follow step-by-step guides to build and customise your
   first OS image.

If you already have a working image and want to extend it, jump directly to
[Customize a Moonforge OS](./tutorials/customize/).

## Prerequisites

Before building a Moonforge image you will need:

- A Linux host with [Docker](https://docs.docker.com/engine/install/) or
  [Podman](https://podman.io/getting-started/installation) installed.
- `kas` 5.0 or later installed via pip:

  ```
  pip install --user kas==5.0
  ```

- Sufficient disk space. A first build typically requires around **100 GB** for downloads
  and the shared-state cache. Subsequent builds reuse the cache and are much faster.

## Key concepts

**Layers** are the building blocks of a Moonforge image. Each layer encapsulates a
single feature: hardware support, containerisation, Over-the-air updates, a graphical session,
and so on. Layers are designed to be composed freely; you include only what your product
needs.

**kas** is the build orchestration tool Moonforge uses to manage layer composition and
build configuration. Each Moonforge layer ships a kas YAML fragment that declares its
own dependencies, so including a layer is as simple as adding one line to your kas
configuration file.

**kas-container** wraps kas in a container, giving you a fully reproducible build
environment that works the same on a developer laptop, a private CI runner, or a public
cloud agent (GitHub Actions, GitLab CI, Bitbucket Pipelines).

**meta-derivative** shows the recommended pattern for your own product repository.
It is a ready-to-use example of a thin downstream layer that references
upstream Moonforge layers and adds your product-specific recipes, machine
configuration, and distribution name. You can fork the
[meta-derivative](https://github.com/moonforgelinux/meta-derivative/)
repository and adapt it to your needs, or you can create your own if you prefer so.

## Getting help

- **GitHub Discussions**: ask questions and share ideas at
  [github.com/orgs/moonforgelinux/discussions](https://github.com/orgs/moonforgelinux/discussions).
- **Issue tracker**: report bugs or request features on the relevant repository under
  [github.com/moonforgelinux](https://github.com/moonforgelinux).
- **Igalia**: for commercial support and consulting, contact
  [Igalia](https://www.igalia.com/contact/).
