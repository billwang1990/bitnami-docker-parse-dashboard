[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-parse-dashboard/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-parse-dashboard/tree/master)
[![Slack](https://img.shields.io/badge/slack-join%20chat%20%E2%86%92-e01563.svg)](http://slack.oss.bitnami.com)
[![Kubectl](https://img.shields.io/badge/kubectl-Available-green.svg)](https://raw.githubusercontent.com/bitnami/bitnami-docker-parse-dashboard/master/kubernetes.yml)

# What is Parse Dashboard?

> Parse Dashboard is a standalone dashboard for managing your Parse apps. You can use it to manage your Parse Server apps and your apps that are running on Parse.com.

http://www.parse.com/

# TL;DR;

## Docker Compose

```bash
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-parse-dashboard/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

## Kubernetes

> **WARNING:** This is a beta configuration, currently unsupported.

Get the raw URL pointing to the `kubernetes.yml` manifest and use `kubectl` to create the resources on your Kubernetes cluster like so:

```bash
$ kubectl create -f https://raw.githubusercontent.com/bitnami/bitnami-docker-parse-dashboard/master/kubernetes.yml
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* Bitnami images are built on CircleCI and automatically pushed to the Docker Hub.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.

# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recomended with a version 1.6.0 or later.

# How to use this image

### Run the application using Docker Compose

This is the recommended way to run Parse Dashboard. You can use the following docker compose template:

```yaml
version: '2'
services:
  mongodb:
    image: 'bitnami/mongodb:latest'
    volumes:
      - 'mongodb_data:/bitnami'
  parse:
    image: 'bitnami/parse:latest'
    volumes:
      - 'parse_data:/bitnami'
    ports:
      - '1337:1337'
    depends_on:
      - mongodb
  parse-dashboard:
    image: 'bitnami/parse-dashboard:latest'
    ports:
      - '80:4040'
    volumes:
      - 'parse_dashboard_data:/bitnami'
    depends_on:
      - mongodb
volumes:
  mongodb_data:
    driver: local
  parse_data:
    driver: local
  parse_dashboard_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a network for the application, Parse Server and the database:

  ```bash
  $ docker network create parse_dashboard-tier
  ```

2. Start a MongoDB database in the network generated:

  ```bash
  $ docker run -d --name mongodb --net=parse_dashboard-tier bitnami/mongodb
  ```

  *Note:* You need to give the container a name in order to Parse to resolve the host.

3. Start a Parse Server container:

  ```bash
  $ docker run -d -p 1337:1337 --name parse --net=parse_dashboard-tier bitnami/parse
  ```

4. Run the Parse Dashboard container:

  ```bash
  $ docker run -d -p 80:4040 --name parse-dashboard --net=parse_dashboard-tier bitnami/parse-dashboard
  ```

Then you can access your application at http://your-ip/

## Persisting your application

If you remove the container all your data and configurations will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a volume at the `/bitnami` path. Additionally you should mount a volume for the persistence of [MongoDB](https://github.com/bitnami/bitnami-docker-mongodb#persisting-your-database) and [Parse](https://github.com/bitnami/bitnami-docker-parse#persisting-your-application) data.

The above examples define docker volumes namely `mongodb_data`, `parse_data` and `parse_dashboard_data`. The application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the `docker-compose.yml` template previously shown:
```yaml
version: '2'

services:
  mongodb:
    image: 'bitnami/mongodb:latest'
    volumes:
      - '/path/to/mongodb-persistence:/bitnami'
  parse:
    image: 'bitnami/parse:latest'
    ports:
      - '1337:1337'
    depends_on:
        - mongodb
    volumes:
      - '/path/to/parse-persistence:/bitnami'
  parse-dashboard:
    image: 'bitnami/parse-dashboard:latest'
    ports:
      - '80:4040'
    depends_on:
      - mongodb
    volumes:
      - '/path/to/parse_dashboard-persistence:/bitnami'

```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```bash
  $ docker network create parse_dashboard-tier
  ```

2. Create a MongoDB container with host volume:

  ```bash
  $ docker run -d --name mongodb \
    --net parse-dashboard-tier \
    --volume /path/to/mongodb-persistence:/bitnami \
    bitnami/mongodb:latest
  ```

  *Note:* You need to give the container a name in order to Parse to resolve the host.

3. Start a Parse Server container:

  ```bash
  $ docker run -d -name parse -p 1337:1337 \
    --net parse-dashboard-tier
    --volume /path/to/parse-persistence:/bitnami \
    bitnami/parse:latest
  ```

4. Run the Parse Dashboard container:

  ```bash
  $ docker run -d --name parse-dashboard -p 80:4040 \
  --volume /path/to/parse_dashboard-persistence:/bitnami \
  bitnami/parse-dashboard:latest
  ```

# Upgrade this application

Bitnami provides up-to-date versions of Parse Dashboard, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the Parse Dashboard container.

1. Get the updated images:

```bash
$ docker pull bitnami/parse-dashboard:latest
```

2. Stop your container

 * For docker-compose: `$ docker-compose stop parse-dashboard`
 * For manual execution: `$ docker stop parse-dashboard`

3. Take a snapshot of the application state

```bash
$ rsync -a /path/to/parse-persistence /path/to/parse-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
```

Additionally, snapshot the [MongoDB](https://github.com/bitnami/bitnami-docker-mongodb#step-2-stop-and-backup-the-currently-running-container) and [Parse server](https://github.com/bitnami/bitnami-docker-parse#step-2-stop-and-backup-the-currently-running-container) data.

You can use these snapshots to restore the application state should the upgrade fail.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm parse-dashboard`
 * For manual execution: `$ docker rm parse-dashboard`

5. Run the new image

 * For docker-compose: `$ docker-compose start parse-dashboard`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name parse-dashboard bitnami/parse-dashboard:latest`

# Configuration

## Environment variables

When you start the parse-dashboard image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:

```yaml
parse-dashboard:
  image: bitnami/parse-dashboard:latest
  ports:
    - 80:4040
  environment:
    - PARSE_DASHBOARD_PASSWORD=my_password
  volumes:
    - 'parse_dashboard_data:/bitnami'
  depends_on:
    - parse
```

 * For manual execution add a `-e` option with each variable and value:

```bash
 $ docker run -d -e PARSE_DASHBOARD_PASSWORD=my_password -p 80:4040 --name parse-dashboard -v /your/local/path/bitnami/parse_dashboard:/bitnami --network=parse_dashboard-tier bitnami/parse-dashboard
```

Available variables:
 - `PARSE_DASHBOARD_USER`: Parse Dashboard application username. Default: **user**
 - `PARSE_DASHBOARD_PASSWORD`: Parse Dashboard application password. Default: **bitnami**
 - `PARSE_HOST`: This host is for Parse Dashboard knows how to form the urls to Parse Server.
 - `PARSE_PORT_NUMBER`: Parse Server Port. Default: **1337**
 - `PARSE_APP_ID`: Parse Server App Id. Default: **myappID**
 - `PARSE_MASTER_KEY`: Parse Server Master Key. Default: **mymasterKey**
 - `PARSE_DASHBOARD_APP_NAME`: Parse Dashboard application name. Default: **MyDashboard**

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-parse-dashboard/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-parse-dashboard/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-parse-dashboard/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_APP_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# Community

Most real time communication happens in the `#containers` channel at [bitnami-oss.slack.com](http://bitnami-oss.slack.com); you can sign up at [slack.oss.bitnami.com](http://slack.oss.bitnami.com).

Discussions are archived at [bitnami-oss.slackarchive.io](https://bitnami-oss.slackarchive.io).

# License

Copyright 2016-2017 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
