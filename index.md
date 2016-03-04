---
layout: documentation
title: RethinkDB Horizon
---

## RethinkDB Horizon

RethinkDB Horizon is an open-source developer platform for building
realtime, scalable web apps. It is built on top of RethinkDB, and
allows app developers to get started with building modern, engaging
apps without writing any backend code.

* Quickstart
* Installing Horizon & RethinkDB
* Tutorial: build a chat application
    * Overview
    * Install React.js
	* Build basic chat
	* Add users
	* Integrate with Github
* Security
* Deploying Horizon apps
* API reference

## About Horizon

Horizon consists of three components:

- **Horizon Server** -- a middleware server that connects to/is built on
  top of RethinkDB, and exposes a simple API/protocol to front-end
  applications.
- **Horizon Client** -- a JavaScript client library that wraps
  Horizon server's protocol in a convenient API for front-end
  developers.
- **Horizon CLI** -- a command line tool, `hz`, aiding in scaffolding,
  development, and deployment.

The first version of Horizon will expose the following services to
developers:

- **Subscribe** -- a streaming API for building realtime apps directly from the
  browser without writing any backend code.
- **Auth** -- an authentication API that connects to common auth providers
  (e.g. Facebook, Google, GitHub).
- **Identity** -- an API for listing and manipulating user accounts.
- **Permissions** -- a security model that allows the developer to protect
  the data from unauthorized access.

Upcoming versions of Horizon will likely expose the following
additional services:

- **Session management** -- manage browser session and session
  information.
- **Geolocation** -- an API that makes it very easy to build
  location-aware apps.
- **Presence** -- an API for detecting presence information for a given
  user and sharing it with others.
- **Plugins** -- a system for extending Horizon with user-defined services
  in a consistent, discoverable way.
- **Backend** -- an API/protocol to integrate custom backend code with
  Horizon server/client-libraries.
