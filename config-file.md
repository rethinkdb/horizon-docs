---
layout: documentation
title: Configuration files
id: configuration
permalink: /docs/configuration/
---

Options for Horizon apps are set by two [TOML][] files in the app's root directory:

* `.hz/config.toml`: most configuration options
* `.hz/secrets.toml`: authentication/token secrets

(A third configuration file, `.hz/schema.toml`, is optionally used for the database schema.) The example configuration files installed by `hz init` are fully commented; this documentation is taken from there.

[TOML]: https://github.com/toml-lang/toml

Note that any of these options can be overridden with environment variables of the form `HZ_<OPTION>`, where `<OPTION>` is the name of the option listed in this file. To override the `serve_static` setting, for instance, you could use:

```sh
export HZ_SERVE_STATIC="./static"
```

This would override the setting in the `config.toml` file. However, if the `--serve-static` option were passed to the `hz serve` command, that would take priority over both the configuration file and any environment variables: configuration precedence is command line flags first, environment variables next, configuration file last.

Options are shown with their default values.

## Networking options (config.toml)

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
access_control_allow_origin = ""
```

## App options (config.toml)

* `project_name`: sets the name of the RethinkDB database used to store the application state.
* `serve_static`: serve static files from the given directory.
* `debug`: enable debug logging statements.

```toml
project_name = "horizon"
serve_static = "dist"
debug = false
```

## RethinkDB options (config.toml)

* `connect`: connect to an existing RethinkDB instance at the specified host/port.
* `start_rethinkdb`: run an internal RethinkDB instance for Horizon.
* `auto_create_collection`: creates a collection with its corresponding RethinkDB table when one is accessed but does not exist.
* `auto_create_index`: creates an index when one is needed but does not exist.

```toml
connect = "localhost:28015"
start_rethinkdb = false
auto_create_collection = false
auto_create_index = false
```

__Warning:__ Tables and indexes are not lightweight objects, and allowing ad hoc creation could potentially expose your service to denial-of-service attacks. The `auto_create_*` options should not be enabled on a publicly-accessible service.

## Authentication options (config.toml)

* `allow_anonymous`: issues new accounts to users without an auth provider.
* `allow_unauthenticated`: allows connections that are not tied to a user id.
* `auth_redirect`: specifies where users will be redirected to after login.
* `access_control_allow_origin`: specifies a host that can access auth settings in production (i.e., set the [Access-Control-Allow-Origin HTTP header][acao]). `"*"` may be specified as a wildcard.

[acao]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS#Access-Control-Allow-Origin

```toml
allow_anonymous = false
allow_unauthenticated = false
auth_redirect = "/"
access_control_allow_origin = ""
```

## secrets.toml

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

In addition, the `secrets.toml` file contains the `token_secret` used for token encryption. When this file is created by `hz init`, the `token_secret` will be set to a randomly-generated base64 string.
