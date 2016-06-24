---
layout: documentation
title: Using Horizon with Docker
id: docker
permalink: /docker
---

## Using Horizon with Docker

Getting Horizon up and running using Docker is easy as singing "Row Row Row Your Boat" 🚢! Using Horizon with Docker makes for a great experience to quickly getting started with Horizon with only having `node` and `npm` as dependencies.

### Our Docker Images

We provide two Docker images currently up on Docker Hub. `horizon-dev` is for development purposes and `horizon` is for your production environment.

Image | `horizon-dev` | `horizon`
------| ------------- | --------------
`--dev`| ✅ | ❌
`--bind all` | ✅ | ✅

Used alone, both containers require that you inject a `RETHINKDB_URI` environment variable into the container which is the host and port of an instance of RethinkDB visible to the container. This is done with the `-e` flag in `docker run` - `-e RETHINKDB_URI=rethinkdb.local:28015`.

The development image only requires that you mount the root of your Horizon application into the container at `/usr/app`. For example:

```
docker run -e RETHINKDB_URI=EXAMPLE_HOST:28015 -v ./:/usr/app rethinkdb/horizon-dev
```

The production image also requires the `RETHINKDB_URI` environment variable and the mounting of the Horizon application at `/usr/app` within the container. No other opinions are made about your deployment configuration but the container is easily configurable by adding more environment variables.

> [The complete list of Horizon configuration variables](/configuration)

### Using Docker Compose

Using Docker Compose is the quickest way to getting started with Horizon. Installing RethinkDB on your host system isn't even required when using the RethinkDB Docker image.

In the root of your Horizon application, make a copy of the development Docker Compose file [found here](https://github.com/rethinkdb/horizon/blob/next/docker-compose.dev.yml) in the Horizon Github repo.

Once you have that copied, you can just run:
```sh
docker-compose up
```

And your Horizon application is now being served on [localhost:8181](http://localhost:8181) and any changes you make will appear on page reload.

### A basic production deployment with Docker

Once you have your Hoirzon app ready, it's very easy to get it deployed with Docker Compose. With the following setup, we are going to:

1. Install Let's Encrypt and create your certs
1. Install Docker
1. Get your application to your server
1. Modify a `docker-compose.yml`
1. Deploy

One way to make this deployment a bit faster is to use the [Docker one-click deploy image](https://www.digitalocean.com/features/one-click-apps/docker/) via Digital Ocean. You'll still need to setup Let's Encrypt, but you can skip the Docker installation section.

#### Setting up Let's Encrypt

In order to serve your app securely, we are going to use certificates provided by [Let's Encrypt](https://letsencrypt.org/). This is a wonderful service which provides free signed certificates for your domain ensuring that anyone who connects to your application does so securely and privately. For the purposes of this example, we are going to assume your public server is running Ubuntu 14.04 or greater.

In order for this to work, you need to make sure that your DNS settings are properly configured so that your domain points to your server. This largely depends on which registrar you are using, so if you need help feel free to [ask us in Slack](http://slack.rethinkdb.com). You can also find [additional instructions online here](https://certbot.eff.org/#ubuntuxenial-other) via the EFF's website.

First you need to SSH into your server, and install Let's Encrypt as well as Docker.

```sh
sudo apt-get install letsencrypt docker
```

After successfully installing, you can now run:

```sh
letsencrypt certonly
```

> Note that if you have anything currently running on port 80, it will block the letsencrypt process.

This will take you through the guided configuration and the Let's Encrypt service will verify that you own the server and domain that is pointing to it via the CNAME DNS record.

Once this is complete you will now have the certificates that your Horizon app needs to know about within the `/etc/letsencrypt/live` directory.

#### Installing Docker

For the most up-to-date way to install Docker on Ubuntu, [follow the instructions over at Docker](https://docs.docker.com/engine/installation/linux/ubuntulinux/).


#### Getting your application to the server

The next step is to get your app onto your server. You can either `rsync` them to your server manually or push your application repo to a **private** git repository online and then clone the repo to your server. The private part is extremely necessary if you have any secrets within your `.hz/config.toml` file including OAuth secrets.

#### Modifying `docker-compose.yml` for your setup

Make a copy of the [`docker-compose.dev.yml`](https://github.com/rethinkdb/horizon/blob/next/docker-compose.prod.yml), rename it to `docker-compose.yml`, then put it into the root of your application directory (at the same level as the `.hz` folder).

You now want to add the following under the `horizon` portion of the `docker-compose.yml` file. We need to mount the certificates from Let's Encrypt into the container, as well as tell Horizon where they are. Instead of the environment variables we are adding, you could also modify the `command` proprety to add the equivalent `hz serve` flags but this way keeps the file more readable. Make sure to replace `your.domain.com` with the domain you setup with Let's Encrypt.

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

#### Starting it up

You can now run `docker-compose up`, and after downloading the `rethinkdb` and `rethinkdb/horizon` Docker images, your application should now be running and secured with TLS (look for the green lock next to the URL in your browser). You can daemonize this setup by adding `-d` like so - `docker-compose up -d`.

You can also add `restart: always` to both the Horizon and RethinkDB configuration in `docker-compose.yml` to allow Docker to restart these containers in case of a crash.
