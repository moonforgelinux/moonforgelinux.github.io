---
title: Design
---

**Note**: This is a living document that describes the current design policy
for Moonforge.

For a high level description of the design goals of Moonforge, see the
[Goals]({{< ref "/goals" >}}) page.

## Releases

### Channels

Moonforge releases follow three channels:

- **stable**, the current stable release
- **next**, the next stable release
- **main**, the main development branch

Each channel uses a separate branch tracks the corresponding Yocto branch:

- **stable** tracks the current LTS release branch
- **next** tracks the next LTS release branch, once it's available
- **main** tracks the current development branch

Each branch tracks one version of Yocto only.

Every time a new Yocto LTS release is finalized, the **stable** branch is
archived, and the **next** branch becomes the new **stable** one.

### Cadence

The **stable** channel releases every month.

The **next** and **main** channels do not have releases.

### Support

Only the **stable** channel is supported.
