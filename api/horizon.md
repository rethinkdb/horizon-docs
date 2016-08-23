---
layout: api
title: Horizon
id: api-horizon
permalink: /api/horizon/
---

The `Horizon` object instantiates and manages the connection to the Horizon server.

```js
// connect to the Horizon server, after Horizon has been loaded via
// <script> tag or require
const hz = new Horizon();
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
const hz = new Horizon({lazyWrites: true});

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
const hz = new Horizon();

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
const hz = new Horizon();
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

## Horizon.aggregate {#aggregate}

Combine the results of multiple Horizon queries into one result set.

```js
const hz = new Horizon();

var userId = 100;
hz.aggregate({
    userId: userId,
    user: hz('users').find(userId),
    activity: {
        posts: hz('posts').findAll({user: userId}),
        topComments: hz('comments').findAll({user: userId}).order('rating', 'descending').limit(10)
    }
}).watch().subscribe(subscribeFunction);
```

(In real code, [subscribe()][sub] would contain a callback function to receive the Observable results from [watch()][watch].)

[sub]: /api/collection/#subscribe

The values for fields in aggregates may contain:

* Horizon queries
* Literals (as in the `userId` field above)
* Objects (as in the `activity` field above)
* Observables
* Promises
* Arrays

Observables inside an aggregate are called when the aggregate is subscribed to, and behave identically whether the aggregate is called with `fetch()` or `watch()`. So an aggregate such as the following:

```js
hz.aggregate({
    counter: Observable.timer(0, 1000),
    result: hz('foo').find('bar')
}).watch().subscribe({ next(x) { console.log(x) }});
```

will be emitted every time the document in the `result` query is changed and every time the counter is incremented.

Arrays are not flattened. A field/value such as `dogShow: [hz('owners'), hz('pets')]` will result in output similar to `[['bob', 'agatha', ...], ['fluffy', 'fido', ...]]`. (Use [merge()][merge] for a union query.)

[merge]: /api/collection/#merge

Aggregates can be nested, although this is equivalent to simply using objects for the "inner" aggregates.

```js
/// this...
hz.aggregate({
    foo: hz.aggregate({ ... })
});

// ...is equivalent to this
hz.aggregate({
    foo: { ... }
});
```

## Horizon.model {#model}

Create a template for aggregates, using parameters.

The [aggregate example](#aggregate) could be rewritten with `model` this way:

```js
const hz = new Horizon();

const userModel = hz.model((userId) => {
    return {
        userId: userId,
        user: horizon('users').find(userId),
        activity: {
            posts: horizon('posts').findAll({user: userId}),
            topComments: horizon('comments').findAll({user: userId}).order('rating', 'descending').limit(10)
        }
    }
});

userModel(100).watch().subscribe(subscribeFunction);
```

You can do anything with `model` that you can do with `aggregate`, including nesting models.
