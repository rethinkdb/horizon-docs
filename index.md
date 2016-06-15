---
layout: documentation
title: RethinkDB Horizon
id: index
permalink: /docs/
---

RethinkDB Horizon is an open-source developer platform for building realtime, scalable web apps. It is built on top of RethinkDB, and allows app developers to get started with building modern, engaging apps without writing any backend code.

Horizon consists of three components:

* **Horizon server:** a middleware server that connects to/is built on top of RethinkDB, and exposes a simple API/protocol to front-end applications.
* **Horizon client:** a JavaScript client library that wraps Horizon server's protocol in a convenient API for front-end developers.
* **Horizon CLI:** a command line tool, `hz`, aiding in scaffolding, development, and deployment.

## Using Horizon

* [Installing Horizon & RethinkDB](/install): an overview of installing the RethinkDB and Horizon servers.
* [Getting started](/docs/getting-started): get up to speed on Horizon's basics.
* The Horizon API: learn about the two JavaScript classes at the heart of Horizon:
    * [Horizon](/api/horizon) (the connection management class)
    * [Collection](/api/collection) (the data management class)
* [Permissions](/docs/permissions): how Horizon's permissions and schema enforcement system works.
* [Users and groups](/docs/users/): an overview of Horizon's user management system.
* [Authentication](/docs/auth): integrating Horizon apps with Github, Twitter and other OAuth providers.
* Running the Horizon server:
    * [CLI](/docs/cli): running the `hz` command line tool.
    * [Embedding](/docs/embed): using Horizon with web frameworks like Express and Koa.
* [Configuration](/docs/configuration): all about the Horizon configuration file, `.hz/config.toml`.
