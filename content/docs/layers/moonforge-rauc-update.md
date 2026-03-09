---
title: 'meta-moonforge-rauc-update'
type: 'docs'
---

This layer adds a simple update service for RAUC.

## What it does

* Provides a custom recipe to trigger RAUC client updates on a timer.

## How to use it

To use this layer, include the following kas file:

```yml
header:
  version: 16
  includes:
    - kas/include/layer/meta-moonforge-rauc-update.yml

local_conf_header:
  40_meta-moonforge-rauc-update: |
    RAUC_BUNDLE_URL = "http://10.0.2.2:3333/LATEST.raucb"
    RAUC_FORCE_REBOOT_ON_UPDATE = "1"
```

Note `RAUC_FORCE_REBOOT_ON_UPDATE` will force a system reboot, therefore running applications must have inhibitors in place to handle this scenario.

## Testing

Set *RAUC_BUNDLE_URL* in the example kas file to your server host, build the image, and run a web server as follows:

```sh
$ cd meta-moonforge-rauc-update/backend
$ cp ../../build/tmp/deploy/images/qemux86-64/moonforge-bundle-base-qemux86-64.raucb data/LATEST.raucb
$ podman compose up
$ podman compose down --volumes
```

On the client, run the service:

```sh
$ systemctl start rauc-update.service
$ systemctl status rauc-update.service -l
$ reboot
```
