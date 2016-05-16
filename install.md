---
layout: documentation
title: Installing Horizon
id: install
permalink: /install
active: install
---

## Prerequisites

**Before installing Horizon, you must install the RethinkDB server.** Consult [Installing RethinkDB][ir] for downloads and installation instructions.

[ir]: http://rethinkdb.com/docs/install/

Horizon is a [Node.js][njs] application. Please install the current stable versions of Node.js and npm (the Node.js package manager).

[njs]: https://nodejs.org/

## Installing Horizon

Install horizon from npm:

    npm install -g Horizon

This will install Horizon and its command line tool, `hz`. (The same tool will also be installed as `horizon`.)

**Now, go on to the [Quickstart][q]!**

[q]: /docs/quickstart

## Working on Horizon itself

If you want to install a development environment _for developing Horizon, not Horizon applications,_ use the following installation instructions instead of installing from npm.

Clone the Horizon repository:

    git clone https://github.com/rethinkdb/horizon.git

Link the client, server, and CLI directories:

    cd horizon/client
    npm link
    cd ../server
    npm link
    cd ../cli
    npm link

This will make the `hz` tool available globally, but will link it to your local copy of Horizon. When you update your copy of the Horizon repository, you'll need to run these commands again if the Horizon dependencies have been altered.

**Now, go on to the [Quickstart][q]!**
