---
title: "Moonforge CLI"
linkTitle: "Moonforge CLI"
weight: 30
description: >
  Tool for developing and maintaining a Moonforge project.
---

The Moonforge CLI is a tool for managing Moonforge projects. It allows you
to quickly set up a Moonforge project for a given target machine, and enable
features that are part of the Moonforge project, in a way that conforms to
the design goals of Moonforge.

## Installation

### From GitHub

1. Clone the [moonforge-cli](https://github.com/moonforgelinux/moonforge-cli) repository
2. Enter the cloned directory and run: `pip install --user -e .`

## Commands

### `init`

Initialize a Moonforge project.

#### Usage

```
moonforge init [-h] [-q] [--fatal-warnings] [--name NAME]
               [--machine MACHINE] [--feature FEATURE]
               [--variable KEY=VALUE] [--vcs VCS]
               PATH
```

#### Arguments

- `-h`, `--help`: Show the help message and exit.

- `-q`, `--quiet`: Suppress messages except warnings and errors.

- `--fatal-warnings`: Turn warnings into errors.

- `--name=NAME`: The project name.

- `--machine=MACHINE`: The target machine. See `machine --list` for a list
  of available machines.

- `--feature=FEATURE`: A feature to enable. See `feature --list` for a list
  of available features. You can specify multiple features per project.

- `--variable=KEY=VALUE`: Set a layer variable `KEY` to a given `VALUE`.

- `--vcs=VCS`: Initializes the project directory using the given VCS. By
  default, the project is initialized as a `git` repository.

- `PATH`: The path of the project to initialize. If unspecified, it will use
  the current directory.

#### Description

The `init` command initializes a Moonforge project at the given `PATH`. You
can specify the target machine and the features for the project at
initialization time; the `init` command will take care of defining the
various kas files and the necessary repositories, as well as creating the
distro layer for your Moonforge project.

#### Examples

- Initialize a new project called "derivative" in the `meta-derivative`
  directory:

```
moonforge init --name=derivative meta-derivative
```

- Initialize a new project for Raspberry Pi 5 in the current directory:

```
moonforge init --machine=raspberrypi5 --name=derivative
```

- Initialize a new project with RAUC simple updates and Podman support:

```
moonforge init \
  --name derivative \
  --feature rauc-simple \
  --feature podman \
  meta-derivative
```

- Initialize the current directory as a Moonforge project with Docker support,
  and WPE support, pointing to the Moonforge website:

```
moonforge init \
  --name derivative \
  --feature docker
  --feature graphics-wpe \
  --variable WAYLAND_COG_LAUNCH_URL=https://moonforgelinux.org
  meta-derivative
```

### `feature`

Show feature information.

#### Usage

```
moonforge feature [-h] [-q] [--fatal-warnings] [--list] [features ...]
```

#### Arguments

- `-h`, `--help`: Show the help message and exit.

- `-q`, `--quiet`: Suppress messages except warnings and errors.

- `--fatal-warnings`: Turn warnings into errors.

- `--list`: List all the available features.

- `features ...`: The features to inspect.

#### Description

The `feature` command shows the information related to a Moonforge feature.
Each feature can be used in a Moonforge project at initialization time.

### `machine`

Show machine information.

#### Usage

```
moonforge machine [-h] [-q] [--fatal-warnings] [--list] [machines ...]
```

#### Arguments

- `-h`, `--help`: Show the help message and exit.

- `-q`, `--quiet`: Suppress messages except warnings and errors.

- `--fatal-warnings`: Turn warnings into errors.

- `--list`: List all available machines.

- `machines ...`: The machines to inspect.

#### Description

The `machine` command shows the information related to a target machine.
Each machine can be used in a Moonforge project at initialization time.
