---
layout: documentation
title: Running the Horizon server
id: server
permalink: /docs/server
---

The `hz serve <project path>` command starts a Horizon server for the given Horizon project. Once started, it serves HTTP(S) requests for your application on the configured port.

Every horizon server requires a RethinkDB server to connect to. Use the `--connect <RethinkDB host>` option to connect to an existing RethinkDB server, or use `--start-rethinkdb` to automatically start a local RethinkDB server.

# Command-line options {#options}

`hz serve` supports the following command-line options:

## General options
* `--project-name NAME, -n NAME` Name of the Horizon project. Determines the name of the RethinkDB database that stores the project data. Default: Last component of the project path
* `--serve-static [PATH]` Enable serving static files via HTTP(S). You can additionally specify the path from which static files will be served (default: `./dist`).
* `--config PATH` Which [config file][config-file] to use. Default: `.hz/config.toml`
* `--debug [yes|no]` Print additional debug output. Default: `no`

## Network options
* `--bind HOST, -b HOST` The host name or IP address that the Horizon server should listen on for incoming requests. Can be specified multiple times to bind to multiple addresses. Default: `localhost`
* `--port PORT, -p PORT` The port number the Horizon server should listen on for incoming requests. Default: `8181`
* `--connect HOST:PORT, -c HOST:PORT` The host and port of the RethinkDB server to connect to. Default: `localhost:28015`
* `--key-file PATH` The key file to use for the HTTPS server. Default: `./key.pem`
* `--cert-file PATH` The certificate to use for the HTTPS server. Default: `./cert.pem`

## Authentication options

* `--token-secret SECRET` A key string for signing JWTs. If not specified, a new random secret is used on each server start.
* `--allow-unauthenticated [yes|no]` Allow unauthenticated users. See [Authentication][auth] for details. Default: `no`
* `--allow-anonymous [yes|no]` Allow anonymous users. See [Authentication][auth] for details. Default: `no`
* `--auth PROVIDER,ID,SECRET` Enable an auth provider with the given options. E.g. `facebook,<id>,<secret>`. See [Authentication][auth] for details.
* `--auth-redirect URL` The URL to redirect to upon completing authentication. Default: `/`

## Development options

* `--dev` Runs the server in [development mode](#development-mode).
* `--secure [yes|no]` Serve websockets and files over encrypted (HTTPS) connections. Ignores the `--key-file` and `--cert-file` options if set to `no`. Default: `yes`
* `--permissions [yes|no]` Check [permissions][permissions] on requests. Warning: Disabling this on a production server is a security risk and can be used by an attacker to exhaust the server's resources. Only recommended for development use. Default: `yes`
* `--start-rethinkdb [yes|no]` Start up a RethinkDB in the current directory. Ignores `--connect` if enabled. Default: `no`
* `--auto-create-collection [yes|no]` Create collections automatically on first use. Warning: Enabling this on a production server is a security risk and can be used by an attacker to exhaust the server's resources. Only recommended for development use. Default: `no`
* `--auto-create-index [yes|no]` Create indexes automatically on first use. Warning: Enabling this on a production server is a security risk and can be used by an attacker to exhaust the server's resources. Only recommended for development use.  Default: `no`

[auth]: /docs/authentication
[config-file]: /docs/config-file
[permissions]: /docs/permissions

# Development mode {#development-mode}

In development mode (`hz serve --dev`), the following flags are enabled by default:
* `--secure no`
* `--permissions no`
* `--auto-create-collection yes`
* `--auto-create-index yes`
* `--start-rethinkdb yes`
* `--allow-unauthenticated yes`
* `--allow-anonymous yes`
* `--serve-static ./dist`

Development mode makes it easy to run a local Horizon instance during application development. Because permission checking is disabled and collections and indexes get automatically created, new application code can be tested without additional configuration.

Development mode should never be enabled on a production server that is publicly accessible. An attacker can exploit a development server by sending unintended requests. These requests can be used to read and/or modify arbitrary data stored for your Horizon application, or can be used to exhaust the resources of your server.

# Scaling Horizon {#scaling}

Horizon supports horizontal scalability. You can serve the same application from multiple Horizon servers across different machines or even data centers.

Servers in a Horizon cluster access a common RethinkDB cluster in order to synchronize the state of your application's state and any internal metadata.

To serve your application from multiple machines, simply copy the static application files to each machine. Then start one Horizon server per machine, using the `--connect <RethinkDB host>` option to connect to a common RethinkDB cluster. Make sure that the `--project-name` option is identical among all servers, or they will be unable to synchronize the application state.

See the [RethinkDB documentation][rethinkdb-scaling] on information on how to horizontally scale a RethinkDB cluster.

[rethinkdb-scaling]: http://www.rethinkdb.com/docs/sharding-and-replication/
