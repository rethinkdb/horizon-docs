---
layout: documentation
title: Embedding Horizon
id: embed
permalink: /docs/embed/
---

While you can start the Horizon server from the `hz` [command line tool][cli], it's also possible to embed it into a Node web app by importing `@horizon/server` and passing a server connection to the Horizon constructor.

[cli]: /docs/cli

For instance, using the [Express][] framework, the steps are:

[express]: http://expressjs.com

```js
#!/usr/bin/env node
'use strict'

const express = require('express');
const horizon = require('@horizon/server');

const app = express();
const httpServer = app.listen(8181);
const options = {
    project_name: 'myProject',
    auth: { token_secret: 'my_super_secret_secret' }
};
const horizonServer = horizon(httpServer, options);

console.log('Listening on port 8181.');
```

Express and Horizon are required, and Express is instantiated with `app.listen()`. Then the resulting `httpServer` object is passed to `horizon` along with an option object. Options that can be passed to the Horizon server constructor are identical to the similarly-named options that can be defined in the [configuration file][cf], with the same defaults:

* `project_name`: `'horizon'`
* `rdb_host`: `'localhost'`
* `rdb_port`: `28015`
* `auto_create_collection`: `false`
* `auto_create_index`: `false`
* `permissions`: `true`
* `path`: `'/horizon'`
* `auth`:
    * `success_redirect`: `'/'`
    * `failure_redirect`: `'/'`
    * `duration`: `'1d'`
    * `create_new_users`: `true`
    * `new_user_group`: `'authenticated'`
    * `token_secret`: `null`
    * `allow_anonymous`: `false`
    * `allow_unauthenticated`: `false`

**Note:** Passing options to the constructor is the only way to configure the Horizon server when it's embedded. The `.hz/config.toml` configuration file will not be read.

[cf]: /docs/configuration

For some examples with other frameworks, including Koa and Hapi, consult the Horizon [examples page][ex].

[ex]: /docs/examples

## Configuring OAuth providers

OAuth endpoints cannot be set up through the options passed to the Horizon server constructor. Instead, you'll need to use the `add_auth_provider` method on the instantiated Horizon server object.

```js
// ... initialization code as above for Express
const horizonServer = horizon(httpServer, options);

horizonServer.add_auth_provider(
    horizon.auth.github,
    { id: 'id', secret: 'secret', path: 'github' }
);
```

For more details on setting up Oauth, read the section in [Authentication][a].

[a]: /docs/auth/#oauth

## Attaching Horizon to multiple HTTP servers

It's possible to pass a list of HTTP servers to the Horizon constructor rather than just one. Here's an example script that attaches Horizon to a public HTTPS server on port 8181 and a local HTTP server on port 8282:

```js
'use strict';
const horizon = require('@horizon/server');

const fs = require('fs');
const http = require('http');
const https = require('https');

// Attach the horizon server to two http servers
// one on [::]:8181 over HTTPS and one on 127.0.0.1:8282 over HTTP
const onHttpRequest = (req, res) => {
    res.writeHead(404);
    res.end('File not found.');
};

const publicServer = https.createServer({
    key: fs.readFileSync('key.pem'),
    cert: fs.readFileSync('cert.pem'),
}, onHttpRequest);

const loopbackServer = http.createServer(on_http_request);

publicServer.listen(8181);
loopbackServer.listen(8282, '127.0.0.1');

const horizonServer = horizon([publicServer, loopbackServer], {
    project_name: 'myProjectName',
    auth: {       
      token_secret: 'foo-bar',
      allow_anonymous: true,
      allow_unauthenticated: true,
    },
});

// Add Twitch authentication
horizonServer.add_auth_provider(horizon.auth.twitch, {
    path: 'twitch',
    id: '0000000000000000000000000000000',
    secret: '0000000000000000000000000000000',
});

// Shut down the server after 60 seconds
setTimeout(() => {
    horizonServer.close();
    publicServer.close();
    loopbackServer.close();
}, 60000);
```

## Accessing the Rethinkdb `r` object

When running in embedded mode, you can write [native ReQL queries][reql] by accessing `r` on an instance of the `horizon` object.

[reql]: http://rethinkdb.com/docs/introduction-to-reql/

```js
// ... initialization code as above for Express
const horizonServer = horizon(httpServer, options);
const r = horizon.r;

horizonServer._reql_conn.ready().then((reql_conn) => {
  r.db('horizon').table('users').run(reql_conn.connection(), function(err, cursor) {
    if (err) throw err;
    cursor.toArray(function(err, result) {
        if (err) throw err;
        console.log(JSON.stringify(result, null, 2));
    });
  });
});
```
