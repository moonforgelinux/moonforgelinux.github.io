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

### `config`

Configure the Moonforge CLI tool

#### Usage

```
moonforge config [-h] [-q] [--fatal-warnings] [--project] [--user] ACTION [KEY] [VALUE]
```

#### Arguments

- `-h`, `--help`: Show the help message and exit.

- `-q`, `--quiet`: Suppress messages except warnings and errors.

- `--fatal-warnings`: Turn warnings into errors.

- `--project`: Use the project's local configuration file.

- `--user`: Use the user's global configuration file.

- `ACTION`: The action to perform: `list`, `get`, `set`

- `KEY`: The configuration key to use.

- `VALUE`: The value of the configuration key.

#### Description

The config command queries, sets, and lists the configuration options for the Moonforge CLI tool.

Each configuration key is composed of a section and a name, separated by a dot.

The config command supports the following actions:

- `list`: Lists all the keys set in the configuration files, along with their value
- `get`: Prints the value of the given key
- `set`: Sets the value of the given key

Moonforge uses different configuration files; they are, in order of precendence from least important to most important:

- user configuration, stored under `XDG_CONFIG_HOME/moonforge/config.toml`
- project configuration, stored in `.moonforge/config.toml` under the project's root

Unless specified differently, the config command will query all the configuration files in the same order.

#### Examples

- Read from all scopes:

```
moonforge config list
```

- Query the value of the `container.engine` key from project scope:

```
moonforge config --project get container.engine
```

- Set the value of the `container.engine` key in the user scope:

```
moonforge config --user set container.engine podman
```

- Set the value of the key in the current scope:

```
moonforge config set container.engine podman
```

### `build`

Build a Moonforge project.

#### Usage

```
moonforge build [-h] [-q] [--fatal-warnings]
                [--fragment FILE]
		[--runtime-args ARGS]
		PATH
```

#### Arguments

- `-h`, `--help`: Show this help message and exit.

- `-q`, `--quiet`: Suppress messages except warnings and errors.

- `--fatal-warnings`: Turn warnings into errors.

- `--fragment`: Additional kas fragment files.

- `--runtime-args`: Additional arguments for the container runtime

- `PATH`: the path of the project to build.

#### Description

The build command sets up a containerized environment to build the Moonforge project at the given `PATH`.

The container engine is defined by the `container.engine` configuration key.

The container image is defined by the `container.image_path`, `container.image_name`, and `container.image_version` configuration keys.

The cached sstate directory is defined by the `build.sstate_dir` configuration key.

The cached downloads directory is defined by the `build.download_dir` configuration key.

The values of `build.sstate_dir` and `build.download_dir` can contain these placeholders:

- `@XDG_CACHE_HOME@`: expands to the `XDG_CACHE_HOME` environment variable
- `@XDG_CONFIG_HOME@`: expands to the `XDG_CONFIG_HOME` environment variable
- `@HOME@`: expands to the `HOME` environment variable
- `@PROJECT_NAME@`: expands to the project's name
- `@MACHINE_NAME@`: expands to the project's machine name

The build artefacts are stored in the `build` directory.

#### Examples

- Build the project in the current directory:

```
moonforge build .
```

- Build the project in the Project/derivative directory:

```
moonforge build Project/derivative
```

- Build the project in the current directory, with an additional fragment:

```
moonforge build --fragment kas/common/debug.yml .
```

- Build the project in the current directory, passing an additional argument
  to the container engine:

```
moonforge build --runtime-args="--device /dev/kvm" .
```

### `shell`

Run a shell in the build environment of a Moonforge project.

#### Usage

```
moonforge shell [-h] [-q] [--fatal-warnings] [--fragment FILE] [--runtime-args RUNTIME_ARGS] PATH
```

#### Arguments

- `-h`, `--help`: Show this help message and exit.

- `-q`, `--quiet`: Suppress messages except warnings and errors.

- `--fatal-warnings`: Turn warnings into errors.

- `--fragment`: Additional kas fragment files.

- `--runtime-args`: Additional arguments for the container runtime.

- `PATH`: the path of the project.

#### Description

The shell command sets up a containerized environment and runs a shell inside it for the Moonforge project at the given `PATH`.

The container engine is defined by the `container.engine` configuration key.

The container image is defined by the `container.image_path`, `container.image_name`, and `container.image_version` configuration keys.

The cached sstate directory is defined by the `build.sstate_dir` configuration key.

The cached downloads directory is defined by the `build.download_dir` configuration key.

The values of `build.sstate_dir` and `build.download_dir` can contain these placeholders:

- `@XDG_CACHE_HOME@`: expands to the `XDG_CACHE_HOME` environment variable
- `@XDG_CONFIG_HOME@`: expands to the `XDG_CONFIG_HOME` environment variable
- `@HOME@`: expands to the `HOME` environment variable
- `@PROJECT_NAME@`: expands to the project's name
- `@MACHINE_NAME@`: expands to the project's machine name

The build artefacts are stored in the `build` directory.

#### Examples

- Run a shell for the project in the current directory:

```
moonforge shell .
```

- Run a shell for the project in the Project/derivative directory:

```
moonforge shell Project/derivative
```

- Run a shell for the project in the current directory, with an additional fragment:

```
moonforge shell --fragment kas/common/debug.yml .
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
