---
layout: documentation
title: Installing Horizon
---

If necessary, install RethinkDB.

Install Node.js:

    brew update
    brew install node

Clone the Fusion repository:

    git clone https://github.com/rethinkdb/fusion.git

Install the client first:

    cd fusion/client
    npm install

Then install the server:

    cd ../server
    npm install
    npm link

If RethinkDB isn't running, start its server.

Start Fusion's server in development mode:

    fusion --dev

The Fusion server's home URI appears to be `http://localhost:8181/fusion/`, and it actually serves `fusion.js` under that URI, regardless of what else it says.

Running examples:

* Execute `test/serve.js`
* Serves from `client` directory at root, `examples` directory at `examples/`
* Must use full URI even for index, e.g.:
  `http://localhost:8181/examples/react-todo-app/index.html`
