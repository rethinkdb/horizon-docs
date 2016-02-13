---
layout: api
title: Collection
---

```js
// connect to the Fusion server
const Fusion = require("fusion");
const server = new Fusion("localhost:8181");

// create a Collection
const messages = server("messages");
```

The `Collection` object represents a group of related documents, and is backed by a RethinkDB table. Documents in a `Collection` are identified by a unique key stored in the `id` field.

```js
messages.store({
    id: 1,
    url: avatar_url,
    from: "bob",
    text: "Hello from RethinkDB"
});
```

Methods on a `Collection` allow you to create, read, update and delete documents. Selections can be performed by matching on any field by passing an object to match against.

```js
// get the first message from Bob
messages.find({from: "bob"});

// get the message with ID 101; a "shortcut" that only works if ID is
// integer values (otherwise use {id: "value"})
messages.find(101);

// get all messages from Bob, ordered by ID
messages.order("id").findAll({from: "bob"});
```

## Collection methods

**[Collection.find][cfi]:** retrieve a single object from a collection.

**[Collection.findAll][cfa]:** retrieve multiple objects from a collection.

**[Collection.limit][cli]:** limit a query's results to a certain number of documents.

**[Collection.order][cor]:** order a query's results based on a given field.

**[Collection.remove][cre]:** delete a single object from a collection.

**[Collection.removeAll][cra]:** delete multiple objects from a collection.

**[Collection.store][cst]:** insert one or more new objects into a collection.

**[Collection.replace][cre]:** replace one or more existing objects in a collection.

**[Collection.upsert][cup]:** replace or insert one or more objects in a collection.

**[Collection.watch][cwa]:** create a [changefeed][] query, returning an observable.

**[Collection.fetch][cfe]:** return a non-updating observable containing the query's results.

[changefeed]: http://rethinkdb.com/docs/changefeeds/ruby/
[cfi]: /api/collection-find/
[cfa]: /api/collection-findall/
[cli]: /api/collection-limit/
[cor]: /api/collection-order/
[cre]: /api/collection-remove/
[cra]: /api/collection-removeall/
[cst]: /api/collection-store/
[cre]: /api/collection-replace/
[cup]: /api/collection-upsert/
[cwa]: /api/collection-watch/
[cfe]: /api/collection-fetch/
