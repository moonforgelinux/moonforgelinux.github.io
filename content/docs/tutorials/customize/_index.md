---
title: Customize a Moonforge OS
description: How to customize your Moonforge-based OS.
weight: 100
---

The Moonforge base image can be customized by adding other Yocto layers to the kas configuration file.

### Reusing existing layers

Besides the distro and BSP-related layers, Moonforge provides additional layers for common features like *docker* support for running containerized applications, *RAUC* for handling OS updates and more. These layers are specially tailored to Moonforge (e.g., these modify related recipes to write data to the persistent partition).

These additional layers are available for inclusion as separate kas fragments, which already handle default configurations and the inclusion of other related repositories and layers. Check the [meta-moonforge](https://github.com/moonforgelinux/meta-moonforge) repository for the full list of additional layers.

To add any of these layers, simply include the layer to the kas configuration:

**kas/derivative-image-base-raspberrypi5.yml:**

```diff
       file: kas/include/layer/meta-moonforge-distro.yml
     - repo: meta-moonforge
       file: kas/include/layer/meta-moonforge-raspberrypi.yml
+    - repo: meta-moonforge
+      file: kas/include/layer/meta-moonforge-docker.yml
```

Note that, depending on the layer, additional configurations could be needed under *local_conf_header*. Check each layer documentation for more details.

### Writing custom layers

When existing layers aren't enough, the recommended approach is to write custom layers. Through these custom layers, it's possible to extend the contents of the base image, that is, to extend existing recipes or provide custom recipes, distribution and machine configuration files.

The containerized environment, provided by *kas-container*, can be used to add custom layers since it includes all the necessarily tooling (e.g., *bitbake-layers*).

To add a new layer, enter and the *kas-container* shell and run *bitbake-layers* as follows:

```sh
$ cd meta-derivative
$ kas-container shell kas/derivative-image-base-raspberrypi5.yml
$ cd /work/
$ bitbake-layers create-layer meta-derivative-distro --priority 20
$ exit
```

Note that, to ensure that the changes made in this custom layer take precedence over the existing layers, the layer priority must be higher than the priority of the existing layers. Use `bitbake-layers show-layers` to list and inspect the existing layers.

After `bitbake-layers create-layer` runs, there will be a new directory structure under `meta-derivative/meta-derivative-distro` for the contents of this custom layer. To extend the base image contents, simply add a *.bbappend* file for the base image recipe:

```sh
$ cd meta-derivative
$ mkdir -p meta-derivative-distro/recipes-core/images/
$ vi meta-derivative-distro/recipes-core/images/moonforge-image-base.bbappend
```

The following *.bbappend* file will add an SSH server and Python runtime to the base image recipe:

**meta-derivative-distro/recipes-core/images/moonforge-image-base.bbappend:**

```bbappend
IMAGE_FEATURES += " \
    ssh-server-openssh \
"

CORE_IMAGE_EXTRA_INSTALL += " \
    python3 \
"
```

Check the [Yocto documentation](https://docs.yoctoproject.org/5.0.14/) for complementary materials on how to extend and write recipes.

To effectively include this new layer to the image, it must be explicitly added to the kas configuration file:

**kas/derivative-image-base-raspberrypi5.yml:**

```diff
     url: https://github.com/moonforgelinux/meta-moonforge.git
     commit: ce8ff3de25dda42215b7c8dfd201fd39c3960b1f
     branch: main
+  meta-derivative:
+    layers:
+      meta-derivative-distro:
```

The new feature and related packages will be available once the image is rebuilt.

## Customize the distro configuration

Up until now the kas configuration file has defaulted to the *moonforge* distro configuration. This configuration can be extended using the custom layer to change any aspect of the distribution (e.g., the name of the distribution).

The following distro configuration file adds a new distro named *derivative* by extending the *moonforge* configuration:

```sh
$ mkdir -p meta-derivative-distro/conf/distro/
$ vi meta-derivative-distro/conf/distro/derivative.conf
```

**meta-derivative-distro/conf/distro/derivative.conf**:

```conf
require conf/distro/moonforge.conf

DISTRO = "derivative"
DISTRO_NAME = "Derivative OS (Derivative Linux Distribution)"
DISTRO_VERSION = "5.0"

MAINTAINER = "Someone <someone@derivative.example>"
```

Note that it's recommended to simply extend the *moonforge* configuration, to ensure compatibility. Otherwise, check the [Yocto documentation](https://docs.yoctoproject.org/5.0.14/dev-manual/custom-distribution.html) for more details on how to create custom distributions.

To effectively use this new distro, it must be explicitly added to the kas configuration file:

**kas/derivative-image-base-raspberrypi5.yml:**

```diff
+distro: derivative
 machine: raspberrypi5
```

To double check that the new distro was added, once the Raspberry Pi 5 is flashed and booted up, run the following command from the Raspberry Pi's terminal:

```sh
$ cat /etc/os-release
```

Note that the same steps can be used to customize other types of configuration, like the machines configurations.

## Tips and tricks

To see the kas configuration file in its full expanded form:

```sh
$ cd meta-derivative
$ kas-container dump kas/derivative-image-base-raspberrypi5.yml
```

The above command will process all of the included kas fragments, output an expanded version of the kas configuration file, and thus display *all* of the configurations, repositories and patches in use.

To double check that the kas file configurations are added in the right order:

```sh
$ cd meta-derivative
$ kas-container shell kas/derivative-image-base-raspberrypi5.yml
$ cat conf/local.conf
```

The *local.conf* file should include all of the configurations added under *local_conf_header* and be correctly sorted according to their numbered prefixes.

To reuse Moonforge's common kas fragments:

```sh
$ cd meta-derivative
$ kas-container checkout kas/derivative-image-base-raspberrypi5.yml
$ cp -r meta-moonforge/kas/common kas/common
$ kas-container build kas/derivative-image-base-raspberrypi5.yml:kas/common/debug.yml
```

The meta-moonforge repository provides common kas fragments useful for common tasks while developing locally or in continuous integration environments (e.g., *:kas/common/debug.yml*).

Since the kas CLI doesn't allow to directly reference fragments provided by external repositories, this limitation can be worked around with a combination of other steps before building the image.

To run *kas-container* with private repositories:

```sh
$ kas-container --ssh-dir $HOME/.ssh --ssh-agent {checkout,dump,build,shell}
```
