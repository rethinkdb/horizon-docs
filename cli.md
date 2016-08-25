---
layout: documentation
title: The Horizon CLI
id: cli
permalink: /docs/cli/
---

The `hz` command line tool's primary function is to start a standalone Horizon server. It also provides utilities for initializing a new Horizon project, creating a self-signed SSL certificate, and other tasks.

**Commands**

* Table of Contents
{:toc}

# init

Initialize a new Horizon project. `hz init` with no argument will initialize a project in the current directory; `hz init <directory>` will initialize a project in the specified directory, creating the directory if necessary. The project will be given the name of the specified directory (the current directory if no argument is given).

# serve

The `hz serve <project path>` command starts a Horizon server for the given Horizon project. Once started, it serves HTTP(S) requests for your application on the configured port.

Every Horizon server requires a RethinkDB server to connect to. Use the `--connect <RethinkDB host>` option to connect to an existing RethinkDB server, or use `--start-rethinkdb` to automatically start a local RethinkDB server. (Note that the `--dev` option for development mode includes `--start-rethinkdb` by default.)

Note that if you are using a database from a Horizon 1.x application, the `serve` command will exit with an error. Use [hz migrate]{#migrate} to upgrade your database in place.

## Command-line options {#serve-options}

`hz serve` supports the following command-line options:

### General options

* `--project-name NAME, -n NAME` Name of the Horizon project. Determines the name of the RethinkDB database that stores the project data.
* `--serve-static [PATH]` Enable serving static files via HTTP(S). You can additionally specify the path from which static files will be served (default: `./dist`).
* `--debug [yes|no]` Print additional debug output. Default: `no`
* `--schema-file [PATH]` Use a given schema file for the database. If the specified schema conflicts with the existing database, a warning will be printed to the console and the option will be ignored. (Use `hz schema apply --force` to override this.)

### Network options

* `--bind HOST, -b HOST` The host name or IP address that the Horizon server should listen on for incoming requests. Can be specified multiple times to bind to multiple addresses. Default: `localhost`
* `--port PORT, -p PORT` The port number the Horizon server should listen on for incoming requests. Default: `8181`
* `--connect HOST:PORT, -c HOST:PORT` The host and port of the RethinkDB server to connect to, or a `rethinkdb://` URI string (see [RethinkDB options][rdbopts] in the configuration file documentation for details). Default: `localhost:28015`
* `--key-file PATH` The key file to use for the HTTPS server. Default: `./horizon-key.pem`
* `--cert-file PATH` The certificate to use for the HTTPS server. Default: `./horizon-cert.pem`
* `--rdb_timeout SECONDS`: timeout to make the connection to the RethinkDB cluster, in seconds. Default: 20

[rdbopts]: /docs/configuration/#rdbopts

The following options provide alternatives to specifying these values in the `--connect` string:

* `--rdb_host HOSTNAME`
* `--rdb_port PORT`
* `--rdb_user USERNAME`
* `--rdb_password PASSWORD`

### Authentication options

* `--token-secret SECRET` A key string for signing JWTs. If not specified, a new random secret is used on each server start.
* `--allow-unauthenticated [yes|no]` Allow unauthenticated users. See [Authentication][auth] for details. Default: `no`
* `--allow-anonymous [yes|no]` Allow anonymous users. See [Authentication][auth] for details. Default: `no`
* `--auth PROVIDER,ID,SECRET` Enable an auth provider with the given options. E.g. `facebook,ID,SECRET`. See [Authentication][auth] for details.
* `--auth-redirect URL` The URL to redirect to upon completing authentication. Default: `/`
* `--access-control-allow-origin`: A host that can access auth settings in production (i.e., set the [Access-Control-Allow-Origin HTTP header][acao]). `"*"` may be specified as a wildcard. Default: `''`

[acao]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS#Access-Control-Allow-Origin

### Development options

* `--dev` Runs the server in [development mode](#development-mode).
* `--secure [yes|no]` Serve websockets and files over encrypted (HTTPS) connections. Ignores the `--key-file` and `--cert-file` options if set to `no`. Default: `yes`
* `--permissions [yes|no]` Check [permissions][permissions] on requests. Warning: Disabling this on a production server is a security risk and can be used by an attacker to exhaust the server's resources. Only recommended for development use. Default: `yes`
* `--start-rethinkdb [yes|no]` Start up a RethinkDB in the current directory. Ignores `--connect` if enabled. Default: `no`
* `--auto-create-collection [yes|no]` Create collections automatically on first use. Warning: Enabling this on a production server is a security risk and can be used by an attacker to exhaust the server's resources. Only recommended for development use. Default: `no`
* `--auto-create-index [yes|no]` Create indexes automatically on first use. Warning: Enabling this on a production server is a security risk and can be used by an attacker to exhaust the server's resources. Only recommended for development use.  Default: `no`

[auth]: /docs/auth
[config-file]: /docs/configuration
[permissions]: /docs/permissions

## Development mode {#development-mode}

In development mode (`hz serve --dev`), the following flags are enabled by default:

* `--secure no`
* `--permissions no`
* `--auto-create-collection yes`
* `--auto-create-index yes`
* `--start-rethinkdb yes`
* `--allow-unauthenticated yes`
* `--allow-anonymous yes`
* `--serve-static ./dist`
* `--access-control-allow-origin '*'`
* `--schema-file .hz/schema.toml`

Development mode makes it easy to run a local Horizon instance during application development. Because permission checking is disabled and collections and indexes get automatically created, new application code can be tested without additional configuration.

Development mode should never be enabled on a production server that is publicly accessible. An attacker can exploit a development server by sending unintended requests. These requests can be used to read and/or modify arbitrary data stored for your Horizon application, or can be used to exhaust the resources of your server.

# create-cert

Create a private and public TLS certificate pair for development. Running this will create two files, `horizon-cert.pem` and `horizon-key.pem`. These can be specified as options to `hz serve` or placed in the [configuration file][config-file].

Note that the certificate created by `create-cert` uses no local identity information; the data is completely random. If you need to use an existing certificate or credentials, you'll have to create the certificate on your own using `openssl` or a similar tool.

# schema save

Save the currently defined Horizon schema, including validation rules, collection and index specifications, as a TOML file. For an example of this command in practice, read the section on "Configuring rules" in [Permissions and schema enforcement][perm].

[perm]: /docs/permissions/#configuring

Run `hz schema save -h` for details on options.

If you use the default schema filename (`.hz/schema.toml`), existing schema files will be preserved, renamed to `schema.toml_` with a datestamp appended.

# schema apply

Load a previously-extracted schema into a Horizon cluster. Run `hz schema apply -h` for details on options.

While the schema format changed with Horizon 2.0, `schema apply` will read pre-2.0 files. (Use `schema save` to rewrite them in the current format.)

# migrate

Migrate a Horizon database from the 1.x to 2.x internal format. This command must be used on databases created with Horizon 1.x applications; Horizon will exit with an error if the `serve` command is executed with an old format database.

# make-token

Manually create a JSON Web Token for a user, allowing user bootstrapping. This is necessary to log in as the Horizon admin user the first time. Simply pass the user ID value to `make-token` as the argument:

```
hz make-token [user-id]
```

The JWT will be printed to the console.

For more details, read "Making an admin auth token" in [Permissions and schema enforcement][admin].

[admin]: /docs/permissions/#admin
