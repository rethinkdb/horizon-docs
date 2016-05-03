---
layout: api
title: Horizon
---

The `Horizon` object instantiates and manages the connection to the Horizon server.

```js
// connect to the Horizon server
const Horizon = require('horizon');
const hz = Horizon();
```

* Table of Contents
{:toc}

# Horizon constructor arguments {#constructor}

All arguments are optional. Pass them to `Horizon` in an object with option keys: `{secure: true}`.

* `host`: the hostname of the Horizon server. This defaults to `window.location`, i.e., the machine that served the application to the client.
* `secure`: a boolean indicating whether the server should use secure websockets. Defaults to `true`.
* `path`: the reserved namespace for Horizon. Defaults to `"horizon"`.
* `lazyWrites`: a boolean indicating whether write operations should be performed in a "lazy" fashion (see below). Defaults to `false`.
* `authType`: a string indicating the authentication method to use for your application's users, one of `"unauthenticated"`, `"anonymous"`, or `"token"`. Defaults to `"unauthenticated"`. (See [Authentication](authentication.md).)

## Lazy writes

When `lazyWrites` is set to `true`, Horizon queries that modify a Collection will not perform their writes when the query is first executed, but instead perform then when the Observable returned by the query is iterated through.

```js
const hz = Horizon({lazyWrites: true});

// because lazyWrites is set, nothing is sent to the server here...
var query = messages.store([
    {
        from: "agatha",
        text: "Meet at Smugglers' Cove on Saturday"
    },
    {
        from: "bob",
        text: "Would Superman lose a fight against Wonder Woman?"
    }
]);

// ...but instead, writes are performed here
query.forEach(uuid => {
    console.log(`Document ${uuid} was created.`);
});
```

The function returned by a `Horizon` object simply returns a [Collection][c] object:

```js
// Return the messages Collection
const messages = hz('messages');
```

# Methods
{:.no_toc}

## Horizon.dispose() {#dispose}

Close a Horizon connection.

## Horizon.status() {#status}

Receive status updates about the connection to the Horizon server.

## Horizon.onConnected() {#onconnected}

Set a callback to execute when the client connects to the Horizon server.

## Horizon.onDisconnected() {#ondisconnected}

Set a callback to execute when the client disconnects from the Horizon server.

## Horizon.onSocketError() {#onsocketerror}

Set a callback to execute when a websocket error occurs.

## Horizon.hasAuthToken() {#hasauthtoken}

Check if the user has a valid authorization token (i.e., has logged in).

## Horizon.authEndpoint() {#authendpoint}

Return a previously-configured OAuth endpoint.

## Horizon.clearAuthTokens() {#clearauthtokens}

Clear authentication tokens from local storage.

```js
Horizon.clearAuthTokens();
```
