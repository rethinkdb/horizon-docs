---
layout: documentation
title: Using Horizon with Docker
id: docker
permalink: /docs/docker/
---

It's quick and easy to get Horizon up and running with Docker! The only dependencies are current versions of `node` and `npm`.

## Our Docker images

We provide an automatically built [horizon](https://hub.docker.com/r/rethinkdb/horizon/) image on Docker Hub. Currently we support both a `latest` and `next` tag. `next` is what is currently in our [`next` branch](https://github.com/rethinkdb/horizon) on rethinkdb/horizon on Github and `latest` is our last stable release.

The container has the following requirements:

* Your Horizon application should be mounted at `/usr/app` within your container.
* The `RETHINKDB_URI` environment variable should be specified with the `-e` to `docker run`, specifying the host and port of an instance of RethinkDB visible to the container.

```
docker run -e RETHINKDB_URI=EXAMPLE_HOST:28015 -v ./:/usr/app rethinkdb/horizon
```

The container makes no other assumptions about your deployment configuration. To set other Horizon configuration options, you can set `HZ_*` environment variables, as described in [the config.toml file][config]. (All the configuration variables that can normally be specified in `.hz/config.toml` can be overridden with environment variables.)

[config]: /docs/configuration

## Using Docker Compose

Using Docker Compose is the quickest way to getting started with Horizon. Installing RethinkDB on your host system isn't even required when using the RethinkDB Docker image.

Copy the [development Docker Compose file][devdc] from the Horizon Github repo to the root of your Horizon application, then run:

[devdc]: https://github.com/rethinkdb/horizon/blob/next/docker-compose.dev.yml

```sh
docker-compose up
```

Now your Horizon application is being served on [localhost:8181](http://localhost:8181), and any changes you make will appear on page reload.

## A basic production deployment with Docker

Once you have your Horizon app ready, it's very easy to get it deployed with Docker Compose. With the following setup, we are going to:

1. Install Docker
2. Install Let's Encrypt and create certificates
3. Get your application to your server
4. Modify a `docker-compose.yml`
5. Deploy

### Installing Docker

The easiest way to install Docker is to use Digital Ocean's [Docker one-click deploy image](https://www.digitalocean.com/features/one-click-apps/docker/). Otherwise, you can follow the instructions to install Docker on Ubuntu [on Docker's web site](https://docs.docker.com/engine/installation/linux/ubuntulinux/).

### Setting up Let's Encrypt

In order to serve your app securely, we are going to use certificates provided by [Let's Encrypt](https://letsencrypt.org/). This is a wonderful service which provides free signed certificates for your domain, ensuring that anyone who connects to your application does so securely and privately. For the purposes of this example, we'll assume your public server is running Ubuntu 14.04 or greater.

In order for this to work, you need to make sure that your DNS settings are properly configured so your domain points to your server. This largely depends on which registrar you are using, so if you need help feel free to [ask us in Slack](http://slack.rethinkdb.com). You can also find [additional instructions online here](https://certbot.eff.org/#ubuntuxenial-other) via the EFF's website.

First, SSH into your server and install Let's Encrypt.

```sh
sudo apt-get install letsencrypt
```

After successfully installing, start Let's Encrypt:

```sh
letsencrypt certonly
```

**Note:** if you have anything currently running on port 80, it will block the Let's Encrypt process!

This will take you through the guided configuration, and the Let's Encrypt service will verify that you own the server and domain that is pointing to it via the CNAME DNS record.

Once this is complete, the certificates your Horizon app needs will be created in the `/etc/letsencrypt/live` directory.

### Getting your application to the server

To install your application onto your server, either `rsync` them to your server manually, or push your application repo to a **private** git repository online and clone the repo to your server. The private part is important if you have *any* secrets within your `.hz/config.toml` file, including OAuth secrets!

Then, you'll need to modify the Docker compose configuration file for your setup. Make a copy of the [`docker-compose.prod.yml`](https://github.com/rethinkdb/horizon/blob/next/docker-compose.prod.yml) file, rename it to `docker-compose.yml`, and put it into the root of your application directory (at the same level as the `.hz` folder).

Add the following under the `horizon` portion of the `docker-compose.yml` file. We need to mount the certificates from Let's Encrypt into the container, and tell Horizon where they are. Make sure to replace `your.domain.com` with the domain you setup with Let's Encrypt.

```yml
horizon:
  image: rethinkdb/horizon
  command: su -s /bin/sh horizon -c "hz serve --connect rethinkdb:28015 --bind all /usr/app"
  environment:
    - HZ_KEY_FILE=/letsencrypt/live/your.domain.com/privkey.pem
    - HZ_CERT_FILE=/letsencrypt/live/your.domain.com/fullchain.pem
    - HZ_SECURE=yes
  volumes:
    - ./:/usr/app
    - /etc/letsencrypt:/letsencrypt
  links:
    - rethinkdb
  ports:
    - "8181:80"
```

Instead of adding environment variables, you could modify the `command` property to add the equivalent `hz serve` flags. We think using the variables keeps the file more readable.

### Starting it up

You can now run `docker-compose up`, and after downloading the `rethinkdb` and `rethinkdb/horizon` Docker images, your application should now be running and secured with TLS (look for the green lock next to the URL in your browser). You can daemonize this setup by adding `-d` like so - `docker-compose up -d`.

You can also add `restart: always` to both the Horizon and RethinkDB configuration in `docker-compose.yml` to allow Docker to restart these containers in case of a crash.
