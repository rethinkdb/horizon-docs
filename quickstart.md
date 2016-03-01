---
layout: documentation
title: Horizon Quickstart
---

Interactions with Horizon are performed with the `hz` application, which has only two commands:

* `init [directory]`: initialize a new Horizon application
* `serve`: serve the project in the current directory

## Initialize an example application ##

Let's create a new Horizon application. Go to a directory you'd like to install this application into and type:

    hz init example-app

This will create the `example-app` directory and install a few files into it. (If you run `hz init` without giving it a directory, it will install these files into the current directory.)

<div style="margin:0 auto;text-align:center"><img src="images/hz-dirs.png" width="370" height="250" /></div>

Here's what these files and directories are:

* `dist` is for static files. You can create files directly here, or use it as the output directory for a build system of your choice.
* `src` is for source files for your build system. This isn't a convention you have to follow; Horizon doesn't touch anything in this directory.
* `index.html` is a sample file. You'll replace this as you develop your application, but there's enough in it to verify that Horizon is installed and working.
* `.hzconfig` is a Horizon configuration file.

## Start the server ##

Start Horizon to test it out:

    hz serve

You'll see a series of output messages as Horizon starts a RethinkDB server, ending with `Metadata synced with server, ready for queries.` Now, go to <http://localhost:8181>.

You should see the message "It works!" scrolling across your screen. While this isn't leveraging RethinkDB for anything yet, it's technically a Horizon application, as the message is created by successfully executing the [horizon.onConnected()][hc] function. Also, we're sorry for the `<marquee>` tag.

[hc]: /api/horizon-onconnected

