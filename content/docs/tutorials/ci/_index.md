---
title: 'Continuous Integration'
linkTitle: 'Continuous Integration'
weight: 100
description: >
  A guide to using Moonforge's GitHub actions for Continuous Integration.
---

This tutorial briefly describes how derivative projects can leverage Moonforge's GitHub actions, to enable Continuous Integration (CI) workflows and provide OS images and other build artifacts (e.g., RAUC update bundles).

## Preparing the build machine

Building images is a compute intensive task. Therefore, the most common approach to reduce cost is to rely on [self-hosted runners](https://docs.github.com/en/actions/concepts/runners/self-hosted-runners).

GitHub's integration for self-hosted runner it's straightforward but, to use Moonforge's GitHub actions, the build machine requires a few preparations.

The specifications for the build machine are no different from the build machine used in the other tutorials. For reference, see following requirements:

* A physical or virtual (VM) Linux machine. Preferably with the latest [supported](https://github.com/actions/runner-images/tree/main/images/ubuntu) Ubuntu LTS release, to improve compatibility with third-party GitHub actions.
* At least 140 Gbytes of free disk space.
* At least 32 Gbytes of RAM.
* At least 8 CPU cores.

Note that the above specifications are the [minimal requirements](https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html#compatible-linux-distribution). It's strongly recommended to provide additional computing resources when possible to maximize build performance.

Once installed, the Linux system installation requires:

* A non-privileged user (e.g., *github-runner*) with no sudo access. See security related notes below.
* The Docker package properly installed and configured. Make sure to include the non-privileged user to the *docker* group, so container actions can be executed.
* Creation of the cache directories for the downloads and shared-state cache (e.g., `mkdir -p ~/kas/cache/{downloads,sstate-cache}`).

Finally, the installation of the GitHub runner service as described in the [official documentation](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/add-runners). It's recommended to install the runner [as a service](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/configure-the-application) so it can automatically restart when the systems restarts.

To test the runner, add a minimal workflow file to your repository:

```yaml
# .github/workflows/main.yml
jobs:
  build:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v6
      - run: echo "It's working!"
```

See GitHub's official [quickstart guide](https://docs.github.com/en/actions/get-started/quickstart) for details.

## Building images

The benefit of containerized solutions, like *kas-container*, is that it guarantees a consistent environment for local and remote builds. Therefore, it's recommended to use *kas-container* also in CI workflows.

To facilitate this, Moonforge provides [build-moonforge-action](https://github.com/moonforgelinux/build-moonforge-action). This GitHub action handles the proper setup and execution of *kas-container*, and provides cleaning and validation steps to minimize its footprint on the build machine. Additionally, it handles other Moonforge specifics (e.g., propagation of the IMAGE_VERSION).

To build an image using this action, extend your workflow file as follows:

```yaml
# .github/workflows/main.yml
jobs:
  build:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v6
      - uses: moonforgelinux/build-moonforge-action@v0.1.1
        with:
          kas_file: kas/image.yml
          dl_dir: /home/github-runner/kas/cache/downloads
          sstate_dir: /home/github-runner/kas/cache/sstate-cache
          image_id: ${{ github.ref_name }}
          image_version: ${{ github.run_id }}
```

The OS images and other build artifacts can be found under the `./build/tmp/deploy/images/` directory as usual.

**Note**: The above example assumes a single runner. If multiple self-hosted runners are used, the [cached](https://wiki.yoctoproject.org/wiki/Enable_sstate_cache) directories must be shared among the runners (e.g., through NFS).

See the actions's [README](https://github.com/moonforgelinux/build-moonforge-action/blob/main/README.md) for details.

## Uploading images

Considering the size and the amount of images and other build artifacts, the GitHub artifacts offering can be [limiting](https://docs.github.com/en/actions/reference/limits#storage-limits-for-all-github-hosted-runners). A common alternative is to upload images to other storage solutions like AWS S3 and compatible services.

To facilitate this, Moonforge provides [upload-moonforge-action](https://github.com/moonforgelinux/upload-moonforge-action). This GitHub action can upload selective files and entire directories to a S3 bucket or to a compatible service.

To upload images using this action, extend your workflow file as follows:

```yaml
# .github/workflows/main.yml
jobs:
  build:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v6
      - uses: moonforgelinux/build-moonforge-action@v0.1.1
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

Each storage service provides different solutions for browsing these uploaded files. If none is provided, a custom solution can be built using [s3fs](https://github.com/s3fs-fuse/s3fs-fuse) in combination with [nginx](https://nginx.org/).

See actions's [README](https://github.com/moonforgelinux/upload-moonforge-action/blob/main/README.md) for details.

## Security considerations

GitHub [recommends](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/add-runners) the use self-hosted runners exclusively for private repositories for security reasons.

Self-hosted runners are disabled for public repositories by default, but this can be [enabled](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/managing-access-to-self-hosted-runners-using-groups#changing-the-access-policy-of-a-self-hosted-runner-group) by creating a runners group, enabling the *Allow public repositories* option, and moving the self-hosted runners to that group.

The following are recommended GitHub organization and repository settings when using self-hosted runners with public repositories:

* Enable specific repositories in the runners group configuration. This is to prevent the accidental use of self-hosted runners from other organization repositories with improper settings.
* Reduce default *GITHUB_TOKEN* permissions to *Read repository contents and packages permissions* only, without write permissions. This is to prevent a malicious actor from having write permissions to the repository, from a modified workflow (e.g., triggered by a pull-request).
* Enable the *Require approval for all external contributors* option. This is to prevent workflows from running at all without the explicit approval of the project maintainers.
* Use *CODEOWNERS* for sensitive project files (e.g., everything under *.github/*). This is for requesting additional approvals before merging sensitive changes.
* Avoid using *pull_request_target* in your workflows. This is to prevent the exposure of repository secrets to external contributors.

Additionally, it's also recommended to take a few extras steps setting up the build machine:

* Double check that the runner's user has limited privileges (e.g., can't sudo). This is to prevent a malicious actor from modifying the build machine.
* Ensure the build machine has no access to sensitive networks. This is to prevent a malicious actor from sniffing from office traffic.
* Ideally, use [ephemeral runners](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#ephemeral-runners-for-autoscaling) when possible. Alternatively, [force](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/customize-containers) all jobs to execute in containers. 

## A working example

A full working [example](https://github.com/moonforgelinux/meta-moonforge/blob/main/.github/workflows/main.yml) of all the topics covered is this tutorial is available at the meta-moonforge repository. Check it out!

