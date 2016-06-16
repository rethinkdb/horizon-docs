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
docker run -e RETHINKDB_URI=172.0.0.2:28015 -v ./:/usr/app rethinkdb/horizon-dev
```

The production image also requires the `RETHINKDB_URI` environment variable and the mounting of the Horizon application at `/usr/app` within the container. No other opinions are made about your deployment configuration but the container is easily configurable by adding more environment variables.

> [The complete list of Horizon configuration variables](/configuration)

### Using Docker Compose

Using Docker Compose is the quickest way to getting started with Horizon. Installing RethinkDB on your host system isn't even required when using the RethinkDB Docker image.

In the root of your Horizon application, make a copy of the development Docker Compose file [found here](https://github.com/rethinkdb/horizon/blob/next/docker-compose.dev.yml) in the Horizon Github repo.

Once you have that copied, you can just run:
```
docker-compose up
```

And your Horizon application is now being served on [localhost:8181](http://localhost:8181) and any changes you make will appear on page reload.
