---
title: 'meta-moonforge-docker'
type: 'docs'
---

This layer adds and configures Docker to be able to run containers.

## What it does

* Provides a custom recipe to configure Docker storage (e.g., to store images on the persistent partition).
* Extends the base image with Docker recipes and its dependencies.

## How to use it

To use this layer, include the following kas file:

```yml
header:
  version: 16
  includes:
    - kas/include/layer/meta-moonforge-docker.yml
```

## Testing

Once in the device, test that `docker` is installed:

```sh
$ docker run -it hello-world
$ docker run -it ubuntu bash
```

Additionally, `docker-compose` is also available:

```sh
$ cat > docker-compose.yml << EOF
services:
  hello:
    image: hello-world
EOF
$ docker compose up
```
