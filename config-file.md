---
layout: documentation
title: The config.toml file
id: configuration
permalink: /docs/configuration/
---

Options for Horizon apps are set by a [TOML][] file in the app's root directory, `.hz/config.toml`. The example configuration file installed by `hz init` is fully commented; this documentation is taken from there.

[TOML]: https://github.com/toml-lang/toml

Note that any of these options can be overridden with environment variables of the form `HZ_<OPTION>`, where `<OPTION>` is the name of the option listed in this file. To override the `serve_static` setting, for instance, you could use:

```sh
export HZ_SERVE_STATIC="./static"
```

This would override the setting in the `config.toml` file. However, if the `--serve-static` option were passed to the `hz serve` command, that would take priority over both the configuration file and any environment variables: configuration precedence is command line flags first, environment variables next, configuration file last.

Options are shown with their default values.

## Networking options

* `bind` controls which local interfaces will be listened on.
* `port` controls which port will be listened on.
* `secure`: disable HTTPS and use HTTP instead when set to `false`.
* `key_file`: HTTPS key file.
* `cert_file`: HTTPS certificate file.

```toml
bind = [ "localhost" ]
port = 8181
secure = true
key_file = "horizon-key.pem"
cert_file = "horizon-cert.pem"
```

## App options

* `project_name`: sets the name of the RethinkDB database used to store the application state.
* `serve_static`: serve static files from the given directory.
* `debug`: enable debug logging statements.

```toml
project_name = "horizon"
serve_static = "dist"
debug = false
```

## RethinkDB options

* `connect`: connect to an existing RethinkDB instance at the specified host/port.
* `start_rethinkdb`: run an internal RethinkDB instance for Horizon.
* `auto_create_table`: creates a table when one is needed but does not exist.
* `auto_create_index`: creates an index when one is needed but does not exist.

```toml
connect = "localhost:28015"
start_rethinkdb = false
auto_create_table = false
auto_create_index = false
```

__Warning:__ Tables and indexes are not lightweight objects, and allowing ad hoc creation could potentially expose your service to denial-of-service attacks. The `auto_create_*` options should not be enabled on a publicly-accessible service.

## Authentication options

Horizon supports OAuth authentication for Facebook, Google, Twitter, Github and Twitch. Each enabled service appears in its own TOML table and has two key/value pairs, `id` and `secret`.

```toml
[auth.facebook]
id = "000000000000000"
secret = "00000000000000000000000000000000"

[auth.google]
id = "00000000000-00000000000000000000000000000000.apps.googleusercontent.com"
secret = "000000000000000000000000"

[auth.twitter]
id = "0000000000000000000000000"
secret = "00000000000000000000000000000000000000000000000000"

[auth.github]
id = "00000000000000000000"
secret = "0000000000000000000000000000000000000000"

[auth.twitch]
id = "0000000000000000000000000000000"
secret = "0000000000000000000000000000000"
```

If the `id` and `secret` pairs are set for a given service, that service is enabled for authentication.

In addition, there are three authentication-related options that are not tied to a specific service:

* `allow_anonymous`: issues new accounts to users without an auth provider. Default: `false`
* `allow_unauthenticated`: allows connections that are not tied to a user id. Default: `false`
* `auth_redirect`: specifies where users will be redirected to after login. Default: `"/"`
