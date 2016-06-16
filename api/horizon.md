---
layout: api
title: Horizon
id: api-horizon
permalink: /api/horizon/
---

The `Horizon` object instantiates and manages the connection to the Horizon server.

```js
// connect to the Horizon server
const Horizon = require('horizon');
const hz = Horizon();
```

* Table of Contents
{:toc}

## Horizon constructor arguments {#constructor}

All arguments are optional. Pass them to `Horizon` in an object with option keys: `{secure: true}`.

* `host`: the hostname of the Horizon server. This defaults to `window.location`, i.e., the machine that served the application to the client.
* `secure`: a boolean indicating whether the server should use secure websockets. Defaults to `true` if static site is served via HTTPS, `false` otherwise.
* `path`: the path the Horizon endpoint can be found under on `host`. Defaults to `"horizon"`.
* `lazyWrites`: a boolean indicating whether write operations should be performed in a "lazy" fashion (see below). Defaults to `false`.
* `authType`: either a string indicating the authentication method to use for your application's users, or a JSON Web Token to use for authentication. This is useful for bootstrapping the admin user, or for integrating with non-browser-based authentication methods.
    * Valid strings are `"unauthenticated"`, `"anonymous"`, or `"token"`. Defaults to `"unauthenticated"`.
    * To use a JWT, pass an object of the following form: `{token: <TOKEN>, storeLocally: [true|false]}`. If `storeLocally` is `true` (the default), the token will be stored in the browser's local storage; if `false`, you'll need to re-authenticate every time you create a Horizon object.
    * See [Authentication][auth] for more information about authentication methods and token creation.

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
query.subscribe(uuid => {
    console.log('Document ${uuid} was created.');
});
```

A `Horizon` object can be called as a function, taking a string as its argument. It returns a [Collection][col] object:

[col]: /api/collection

```js
// Return the messages Collection
const messages = hz('messages');
```

# Methods
{:.no_toc}

## Horizon.connect {#connect}

Establish a Horizon connection.

Note that you can create a [Collection][col] from the `Horizon` instance without calling `connect()` first. Once you start using the collection, the connection will be automatically established.

```js
const hz = Horizon();

// Get access to the messages collection
const messages = hz('messages');

// Start establishing the Horizon connection.
// This step is optional. We can skip it and go directly to the next line.
hz.connect();

// Start using the collection
messages.store({ msg: 'Hello World!' });
```

## Horizon.disconnect {#disconnect}

Close a Horizon connection.

## Horizon.status {#status}

Receive status updates about the connection to the Horizon server.

Calling `status()` without any arguments returns an [RxJS Observable][rjso]. Alternatively you can pass in a callback to execute when the status of the connection changes. Any arguments passed to `status()` will be treated as if passed to `Observable.subscribe(args...)`.

[rjso]: http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html

The emitted status objects can be one of the following:

* `{ type: 'unconnected' }`: The initial status before any connection has been established.
* `{ type: 'connected' }`: A websocket connection has been established. However the Horizon connection is not ready yet, since the Horizon handshake hasn't been performed yet.
* `{ type: 'ready' }`: The `Horizon` instance is now fully usable.
* `{ type: 'error' }`: An error has occurred. A separate `Error` object with the specific error message will be emitted separately through the error callback (if any is subscribed on the `Observable`).
* `{ type: 'disconnected' }`: The websocket was closed.

## Horizon.onReady {#onready}

Similar to `Horizon.status()`, but only emits `{ type: 'ready' }` events.

## Horizon.onDisconnected {#ondisconnected}

Similar to `Horizon.status()`, but only emits `{ type: 'disconnected' }` events.

## Horizon.onSocketError {#onsocketerror}

Similar to `Horizon.status()`, but only emits `{ type: 'error' }` events.

## Horizon.hasAuthToken {#hasauthtoken}

Check if the user has a valid authorization token (i.e., has logged in).

See [Authentication][auth] for more details.

## Horizon.currentUser {#currentuser}

Returns a query for the current user, that you can run by calling either [`watch()`][watch] or [`fetch()`][fetch].

The query result is a user object as described in [Users and groups][users], or an empty object if the user is unauthenticated.

```js
const hz = Horizon();
hz.currentUser().fetch().subscribe( (user) => console.log(JSON.stringify(user)) );
```

`currentUser()` requires read [permissions][perm] on the user's document in the `"users"` collection. The following rule enables the required permissions:

```toml
[groups.default.rules.read_current_user]
template = "collection('users').find({id: userId()})"
```

See [Permissions][perm] for more information on how to configure access rules, and [Authentication][auth] to find out how to configure user authentication on the Horizon server.

[watch]: /api/collection/#watch
[fetch]: /api/collection/#fetch
[users]: /docs/users
[perm]: /docs/permissions

## Horizon.authEndpoint {#authendpoint}

Return a previously-configured OAuth endpoint.

See [Authentication][auth] for more details.

## Horizon.clearAuthTokens {#clearauthtokens}

Clear authentication tokens from local storage.

```js
Horizon.clearAuthTokens();
```

See [Authentication][auth] for more details.

[auth]: /docs/auth
