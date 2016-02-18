---
layout: documentation
title: Installing Horizon
---

If necessary, install RethinkDB.

Install Node.js:

    brew update
    brew install node

Clone the Horizon repository:

    git clone https://github.com/rethinkdb/horizon.git

Install the client first:

    cd horizon/client
    npm install

Then install the server:

    cd ../server
    npm install
    npm link

If RethinkDB isn't running, start its server.

Start Horizon's server in development mode:

    horizon --dev

Running examples:

* Execute `test/serve.js`
* Serves from `client` directory at root, `examples` directory at `examples/`
* Must use full URI even for index, e.g.:
  `http://localhost:8181/examples/react-todo-app/index.html`
