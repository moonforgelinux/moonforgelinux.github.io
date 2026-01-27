---
title: Design Goals
description: 'The goals of the Moonforge project'
linkTitle: Goals
type: docs
menu: { main: { weight: 10 } }
---

Moonforge is a Linux distribution built on the Yocto and OpenEmbedded
projects. As such, it's a collection of Yocto layers and auxiliary kas
configuration files, arranged so that the following goals can be achieved:

- Good balance between having turn-key solutions and the flexibility to
  extend beyond these solutions, to support a wide range of products.
- Clear separation between upstream and downstream components, to support
  the creation of derivative products with predictable processes and
  releases.
- Best practices in place, to provide a simplified yet powerful integration
  experience.

### Balance

Building a Linux distribution often comes down to selecting a base
distribution, an integration tool and adjusting system components to an
overall system design. Experience shows that these tasks result in a lot of
duplicated effort that can be inconsequential for the actual product.

Moonforge's proposition is to free the system integrator from that burden by
making core design decisions and providing common components tailored to
these decisions. Thus, reducing that duplicated effort and enabling the
integrator to focus on their own products.

At the same time, experience also shows that there isn't a one-size-fits-all
system design and, therefore, balance is required.

Moonforge's approach to balance is to leverage OpenEmbedded's build system
best capabilities with a mix of conventions and sensible defaults powered by
kas and the practices it enables. More concretely:

- High-level components are provided as separate Yocto layers that can be
  combined in different ways.
- Kas fragments are provided for each layer, so these can be combined and
  customized in a declarative fashion to build different products. These
  fragments handle the inclusion of external repositories, the activation of
  the required layers, and provide sensible defaults that can be overridden.

As Moonforge development continues, additional layers with distinct
components will be added while existing layers will be refined.

### Separation

These existing layers can cover many common needs, but can't cover every
need. Here is where the layers capabilities of the OpenEmbedded build system
provide an appropriate solution for:

- Having flexibility when existing layers aren't enough.
- Having a clear separation between components.
- Building new products by extending Moonforge and allowing vendor-specific
  components.

The key responsibilities of these layer kas fragments are:

- Inclusion of the external repositories that are required by this layer
  (e.g., meta-openembedded), to act as a single point of definition
  for external resources
- Activation of the specific layers, within an external repository, that are
  required by this layer (e.g., meta-oe).
- Inclusion of other layers required by this layer (e.g. a graphic session,
  a WPE environment, etc).
- Inclusion of downstream patches for the layer itself.
- Definition of sensible defaults for configurations that belong to the
  local.conf file (e.g., ` PREFERRED_PROVIDER_virtual`).

### Best practices

The goal of the Moonforge project is to adopt and showcase the best
practices for Linux-based operating systems:

- Rely on established technologies for building an OS image, like the Yocto
  Project, bitbake, and kas.
- Take full advantage of the modern Linux kernel and user space stacks, in
  order to target the next generation of devices and use cases.
- Generate immutable OS images that can be easily verified, deployed and
  updated by existing technologies, such as systemd, RAUC, and Mender
- Use CI/CD pipelines to build and publish OS images, security reports,
  Software Bill of Materials metadata, and update bundles.

Special consideration is reserved to the ability to set up, manage, and
deploy the build environment on multiple hosting systems: public cloud,
private cloud, or local infrastructure.
