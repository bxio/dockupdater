# Frequently Asked Questions

* [How to configure Timezone](#how-to-configure-timezone)
* [How to stop Docupdater auto update](#how-to-stop-docupdater-auto-update)
* [How install Docupdater without docker](#how-install-docupdater-without-docker)
* [How to remove container after service update](#how-to-remove-container-after-service-update)
* [How to use with service but without stack](#how-to-use-with-service-but-without-stack)

***

## How to configure Timezone

To more closely monitor docupdater' actions and for accurate log ingestion, you can change the timezone of the container from UTC by setting the [`TZ`](http://www.gnu.org/software/libc/manual/html_node/TZ-Variable.html) environment variable like so:

```bash
docker run -d --name docupdater \
  -e TZ=America/Chicago \
  -v /var/run/docker.sock:/var/run/docker.sock \
  docupdater/docupdater
```

## How to stop Docupdater auto update

Your can add label `docupdater.disable="true"` on the container or service to disable auto update.

If your run a standalone container for docupdater:

```bash
docker run -d --name docupdater \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --label docupdater.disable="true"
  docupdater/docupdater
```

If your run docupdater with a stack:

```bash
version: "3.6"

services:
  docupdater:
    image: docupdater/docupdater
    deploy:
      labels:
        docupdater.disable: "true"
```

## How install Docupdater without docker

docupdater can also be installed via pip:

```bash
pip install docupdater
```

And can then be invoked using the docupdater command:

$ docupdater --interval 300 --log-level debug

> Docupdater need Python 3.6 or up

## How to remove containers after service update

By default Docker swarm keep 5 stop containers by service. You can configure that number at 0 to always remove old containers.

The update your docker swarm, run that command on a manager:

$ docker swarm update --task-history-limit=0


## How to use with service but without stack

You can start a service with the command line without using stack file :

```bash
docker service create --name test1 --replicas=2 --tty=true busybox
```

Unfortunately that cause an issue with Docupdater. They have 2 workarounds:

**Option 1**

Run Docupdater with the option [`--disable-containers-check`](Options.md#disable-containers-check). That will disable update for standalone containers.

**Option 2**

Add label [`docupdater.disable`](Labels.md#disable-update) on the containers, this will disable update for standalone containers, but service will be updated normally.

You can add label for all containers of a service like that:
```bash
docker service create --name test1 --replicas=2 --tty=true --container-label="docupdater.disable=true" busybox
```