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

## Horizon constructor arguments

All arguments are optional. Pass them to `Horizon` in an object with option keys: `{secure: true}`.

* `host`: the hostname of the Horizon server. This defaults to `window.location`, i.e., the machine that served the application to the client.
* `secure`: a boolean indicating whether the server should use secure websockets. Defaults to `true`.
* `path`: the reserved namespace for Horizon. Defaults to `"horizon"`.
* `lazyWrites`: a boolean indicating whether write operations should be performed in a "lazy" fashion (see below). Defaults to `false`.

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


## Horizon methods

The function returned by a `Horizon` object simply returns a [Collection][c] object:

```js
const messages = hz('messages');
```

**[Horizon.dispose][hd]**: close a Horizon connection.

**[Horizon.status][hs]**: receive status updates about the connection to the Horizon server.

**[Horizon.onConnected][hoc]**: set a callback to execute when the Horizon server is connected.

**[Horizon.onDisconnected][hod]**: set a callback to execute when the Horizon server is disconnected.

**[Horizon.onSocketError][hse]**: set a callback to execute when a websocket error occurs.

[hd]:  /api/horizon-dispose/
[hs]:  /api/horizon-status/
[hoc]: /api/horizon-onconnected/
[hod]: /api/horizon-ondisconnected/
[hse]: /api/horizon-onsocketerror/
