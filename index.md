---
layout: documentation
title: RethinkDB Horizon
id: index
permalink: /docs/
---

RethinkDB Horizon is an open-source developer platform for building realtime, scalable web apps. It is built on top of RethinkDB, and allows app developers to get started with building modern, engaging apps without writing any backend code.

Horizon consists of three components:

* **Horizon Server:** a middleware server that connects to/is built on top of RethinkDB, and exposes a simple API/protocol to front-end applications.
* **Horizon Client:** a JavaScript client library that wraps Horizon server's protocol in a convenient API for front-end developers.
* **Horizon CLI:** a command line tool, `hz`, aiding in scaffolding, development, and deployment.

## Using Horizon

* [Installing Horizon & RethinkDB](/install)

    An overview of installing the RethinkDB and Horizon servers.

* [Quickstart](/quickstart)

    Quickly get up to speed on Horizon's basics.

* The Horizon API

    Learn about the two JavaScript classes at the heart of Horizon:

    * [Horizon](/api/horizon) (the connection management class)
    * [Collection](/api/collection) (the data management class)

* [Users and permissions](/users)

    How Horizon's users and permissions system works.

* [Authentication](/authentication)

    Integrating Horizon apps with Github, Twitter and other OAuth providers.

* [Configuration](/config-file)

    All about the Horizon configuration file, `.hz/config.toml`.

* [Deploying Horizon](/deploy)

    Deploying a Horizon application.

## Horizon Tutorial

*Coming soon:* Build a realtime chat application using Horizon and React.js, with support for multiple users and authorization integration with Github.
