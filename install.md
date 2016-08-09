---
layout: documentation
title: Installing Horizon
id: install
permalink: /install/
active: install
---

## Before you start

Horizon is a rapidly evolving framework, and depending on your use case, there's still a chance of encountering bugs or unusual quirks. Read the [limitations][l] page for important notes and caveats.

[l]: /docs/limitations

## Prerequisites

**Before installing Horizon, you must install the RethinkDB server.** Consult [Installing RethinkDB][ir] for downloads and installation instructions.

[ir]: http://rethinkdb.com/docs/install/

Horizon is a [Node.js][njs] application. Please install the current stable versions of Node.js and npm (the Node.js package manager). Version 4.4 or higher of Node is required.

[njs]: https://nodejs.org/

## Installing Horizon

Install Horizon from npm:

    npm install -g horizon

This will install Horizon and its command line tool, `hz`. (The same tool will also be installed as `horizon`.)

**Now, go on to [Getting Started][gs]!**

[gs]: /docs/getting-started

## Working on Horizon itself

If you want to install a development environment _for developing Horizon, not Horizon applications,_ use the following installation instructions instead of installing from npm.

Clone the Horizon repository:

    git clone https://github.com/rethinkdb/horizon.git

Link the client, server, and CLI directories using our handy setup script:

    cd horizon/test
    ./setupDev.sh

This will make the `hz` tool available globally, but will link it to your local copy of Horizon. When you update your copy of the Horizon repository, you'll need to run these commands again if the Horizon dependencies have been altered.

**Now, go on to [Getting Started][gs]!**
