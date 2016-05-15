---
layout: documentation
title: Horizon Quickstart
id: quickstart
permalink: /docs/quickstart
---

If you haven't installed Horizon, do so now. (Read the [Installation instructions][install] for more details.)

    npm install -g horizon

[install]: /install

## Using the Horizon CLI ##

Interactions with Horizon are performed with the `hz` application. `hz` has a number of commands, of which we are going to use the following two:

* `init [directory]`: initialize a new Horizon application
* `serve`: serve the project in the current directory

## Initialize an example application ##

Let's create a new Horizon application. Go to a directory you'd like to install this application into and type:

    hz init example-app

This will create the `example-app` directory and install a few files into it. (If you run `hz init` without giving it a directory, it will install these files into the current directory.)

<div style="margin:0 auto;text-align:center"><img src="images/hz-dirs.png" width="351" height="181" /></div>

Here's what these files and directories are:

* `dist` is for static files. You can create files directly here, or use it as the output directory for a build system of your choice.
* `src` is for source files for your build system. This isn't a convention you have to follow; Horizon doesn't touch anything in this directory.
* `index.html` is a sample file. You'll replace this as you develop your application, but there's enough in it to verify that Horizon is installed and working.
* `.hz/config.toml` is a [TOML][] configuration file for the Horizon server.

[TOML]: https://github.com/toml-lang/toml

## Start the server ##

Start Horizon to test it out:

    hz serve --dev

You'll see a series of output messages as Horizon starts a RethinkDB server, ending with `Metadata synced with server, ready for queries.` Now, go to <http://localhost:8181>. You should see the message "It works!" scrolling across your screen.

Here's what `hz serve` actually does:

* Start the Horizon API server, a Node.js application.
* Starts an HTTP server which serves the Horizon client library, `horizon.js`.

Passing the `--dev` flag to `hz serve` puts it in development mode, which makes the following changes. (All of these can also be set individually with separate flags to `serve`.)

* A RethinkDB server is automatically started (`--start-rethinkdb`). This server is specifically for this Horizon application, and will create a `rethinkdb_data` folder in the working directory when started.
* Horizon is served in "insecure mode," without requiring SSL/TLS (`--secure no`).
* The permissions system is disabled (`--permissions no`).
* Tables and indexes will automatically be created if they don't exist (`--auto-create-table` and `--auto-create-index`).
* Static files will be served from the `dist` directory (`--serve-static`).

You can find the complete list of [command line flags][server] for `hz serve` in the documentation for the [Horizon server][server].

In production (i.e., without the `--dev` flag), you'll use the `.hz/config.toml` file to set these and other options. See [Configuring Horizon][configuration] for details.

[server]: /docs/server
[config-file]: /docs/configuration

## Talk to Horizon ##

Load the `index.html` file in `example-app`. It's pretty short:

```html
<!doctype html>
<html>
  <head>
    <meta charset="UTF-8">
    <script src="/horizon/horizon.js"></script>
    <script>
      var horizon = Horizon();
      horizon.onReady(function() {
        document.querySelector('h1').innerHTML = 'It works!'
      });
      horizon.connect();
    </script>
  </head>
  <body>
   <marquee><h1></h1></marquee>
  </body>
</html>
```

The two `script` tags do the work here. The first loads the actual Horizon client library, `horizon.js`; the second is a (very tiny) Horizon application:

* `var horizon = Horizon()` instantiates a [Horizon][ho] object. This object only has a few methods on it, for handling connection-related events and for instantiating Horizon [Collections][co].
* [onReady()][hc] is an event handler that's executed when the client makes a successful connection to the server.
* Our connection function simply fills in `"It works!"` into the `<h1>` tag in the document. Since this function only gets executed on a successful connection, it *does* verify that Horizon is working, but it's not leveraging RethinkDB for anything yet.
* Also, we're sorry for the `<marquee>` tag.

[ho]: /api/horizon
[co]: /api/collection
[hc]: /api/horizon/#onready
