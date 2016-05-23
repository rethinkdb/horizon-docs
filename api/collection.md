---
layout: api
title: Collection
id: api-collection
permalink: /api/collection/
---

The `Collection` object represents a group of related documents, and is backed by a RethinkDB table. Documents in a `Collection` are identified by a unique key stored in the `id` field.

```js
// connect to the Horizon server
const Horizon = require("horizon");
const hz = Horizon();

// get a handle to a Collection
const messages = hz("messages");
```

Methods on a `Collection` object allow you to create, read, update and delete documents. Selections can be performed by matching on any field by passing an object to match against.

```js
// Store a message
messages.store({
    id: 1,
    url: avatar_url,
    from: "bob",
    text: "Hello from RethinkDB"
});

// get the first message from Bob
messages.find({from: "bob"}).fetch().subscribe(msg => console.log(msg));

// the same, using a Horizon object directly
hz("messages").find({from: "bob"}).fetch().subscribe(msg => console.log(msg));

// get the message with ID 101; a "shortcut" that only works when fetching based on the
// unique document ID (the equivalent long form is `find({id: 101})`)
messages.find(101).fetch().subscribe(msg => console.log(msg));

// get all messages from Bob, ordered by ID
messages.order("id").findAll({from: "bob"}).fetch().subscribe(msg => console.log(msg));
```

# Methods
{:.no_toc}

* Table of Contents
{:toc}

## Collection.fetch {#fetch}

Return a [RxJS Observable](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html) containing the query result set.

```js
Collection.fetch()
```

Unlike [watch](#watch), the `fetch` command does not update in real time, but rather returns a "snapshot" of the result set as it exists when `fetch` is executed. The `fetch` or `watch` command ends a Horizon query.

The Observable returned by `fetch` or `watch` may be iterated through with [subscribe](#subscribe), or converted to an array with `toArray`.

```js
const hz = Horizon();

hz("messages").fetch().subscribe(
    result => console.log('Result:', result),
    err => console.error(err),
    () => console.log('Results fetched')
);
```

## Collection.subscribe {#subscribe}

Provide handlers to a Collection result set.

```js
Collection.fetch().subscribe(readFunction, errorFunction, completedFunction)
Collection.store().subscribe(writeFunction, errorFunction)
Collection.watch().subscribe(changefeedFunction, errorFunction)
```

This method is not actually part of the `Collection` class, but is instead a [RxJS method](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-subscribe).

When `subscribe` is chained off a read function (i.e., [fetch](#fetch)) it takes three callback functions:

* `next(result)`: a callback that receives a single document as the Collection is iterated through
* `error(error)`: a callback that receives error information if an error occurs
* `complete()`: a callback executed when the result set has been iterated through completely

```js
const hz = Horizon();

hz("messages").fetch().subscribe(
    result => console.log('Result:', result),
    err => console.error(err),
    () => console.log('Results fetched')
);
```

When chained off a write function (e.g., [store](#store), [upsert](#upsert)), it takes two callback functions:

* `write(id)`: a callback that receives the `id` of the documents written, one at a time
* `error(error)`: a callback that receives error information if an error occurs

```js
hz("messages").store([
    {
        from: "agatha",
        text: "Meet at Smugglers' Cove on Saturday"
    },
    {
        id: 3,
        from: "bob",
        text: "Would Superman lose a fight against Wonder Woman?"
    }
]).subscribe(
    (id) => console.log("id value:", id),
    (err) => console.error(err)
);
```

This would produce output similar to:

    id value: f8dd67dc-2301-487a-85ab-c4b573acad2d
    id value: 3

In the first case, the `id` value is an automatically generated UUID; in the second, it was supplied with Bob's message.

When `subscribe` is chained off [watch](#watch), it takes two callback functions:

* `changefeed(result)`: a callback that receives a changefeed result document
* `error(error)`: a callback that receives error information if an error occurs

Read the documentation for `watch` for more details on returned changefeed dowcuments.

## Collection.watch {#watch}


Convert a query into a [changefeed][feed]. This returns an Observable which receives real-time updates of the query's result set.

[feed]: https://rethinkdb.com/docs/changefeeds/javascript/

```js
Collection.watch({options})
```

The `watch` command takes one option, `rawChanges`. If set `true`, your application will receive the actual change documents from the RethinkDB changefeed, rather than having Horizon update the result set for you based on the change documents. (Read the RethinkDB [changefeed documentation][feed] for more details on change documents.)

```js
const hz = Horizon();

// get a handle to the channels table in a multi channel chat application
const channels = hz("channels");

// receive all active channels, listing them every time a channel is
// added, deleted or changed
channels.watch().subscribe(allChannels => {
    console.log('Channels: ', allChannels);
});
```

That will return output such as this:

    Channels: []
    Channels: [{id: 1, name: 'everyone'}]
    Channels: [{id: 1, name: 'everyone'}, {id: 2, name: 'rethinkdb'}]
    Channels: [{id: 1, name: 'general'}, {id: 2, name: 'rethinkdb'}]

To get raw change documents:

```js
channels.watch({rawChanges: true}).subscribe(allChannels => {
    console.log('Change: ', allChannels)
});
```

The changes in the example above would produce these raw changes:

    Change: {type: 'state', state: 'synced'}
    Change: {type: 'add', new_val: {id: 1, name: 'everyone'}, old_val: null}
    Change: {type: 'add', new_val: {id: 2, name: 'rethinkdb'}, old_val: null}
    Change: {type: 'change', new_val: {id: 1, name: 'general'},
             old_val: {id: 1, name: 'everyone'}}

You can also use `watch` to turn more complex Horizon queries into changefeeds.

```js
// only watch channels with 10 or more active users
channels.above({users: 10}, "closed").watch().subscribe(allChannels => {
    console.log('Popular channels: ', allChannels);
});

// maintain an updating "top 10" channel list
channels.order("users", "descending").limit(10).watch().subscribe(allChannels => {
    console.log('Popular channels: ', allChannels);
});
```

## Collection.above {#above}

Restrict the range of results returned to values that sort above a given value.

```js
Collection.above(id | object[, "closed" | "open"])
```

The `above` method may be called with either a key-value pair to match against (e.g., `{name: "agatha"}` or an `id` value to look up. The second optional parameter must be the string `"closed"` or `"open"`, indicating that the specified value will be included (closed) or excluded (open) from the result set. The default is excluded (open): `above(10)` will return documents with `id` values higher than (but not equal to) 10.

Values in key-value pairs may be numbers, strings, or even arrays or objects; non-numeric values will be sorted lexicographically, and strings are sorted by UTF-8 codepoint. (Read about [Sorting order][so] and [ReQL data types][dt] in general.)

[so]: https://rethinkdb.com/docs/data-types/#sorting-order
[dt]: https://rethinkdb.com/docs/data-types/

The `above` method is often used in conjunction with [order](#order), but it may appear after any Horizon method with the exception of [find](#find) and [limit](#limit). (However, `limit` may appear after `above`.)

```js
const hz = Horizon();
const messages = hz("messages");

// get all messages with an ID over 100, sorted
messages.order("id").above({id: 100}).fetch();

// the same as above, but using the shorthand for fetching by document ID
messages.order("id").above(100).fetch();

// get all messages with an ID between 101 and 200, sorted
messages.order("id").below(200).above(100).fetch();

// get all users with a reputation score of 50 or over, unsorted
users.above({reputation: 50}, "closed").fetch();
```

## Collection.below {#below}

Restrict the range of results returned to values that sort below a given value.

```js
Collection.below(id | object[, "closed" | "open"])
```

The `below` method may be called with either a key-value pair to match against (e.g., `{name: "agatha"}` or an `id` value to look up. The second optional parameter must be the string `"closed"` or `"open"`, indicating that the specified value will be included (closed) or excluded (open) from the result set. The default is excluded (open): `below(10)` will return documents with `id` values lower than (but not equal to) 10.

Values in key-value pairs may be numbers, strings, or even arrays or objects; non-numeric values will be sorted lexicographically, and strings are sorted by UTF-8 codepoint. (Read about [Sorting order][so] and [ReQL data types][dt] in general.)

The `below` method may _only_ be used after [order](#order), although other methods may be used after it.

```js
const hz = Horizon();
const messages = hz("messages");

// get all messages with an ID below 100, sorted
messages.order("id").below({id: 100}).fetch();

// the same as above, but using the shorthand for fetching by document ID
messages.order("id").below(100).fetch();

// get all messages with an ID between 101 and 200, sorted
messages.order("id").below(200).above(100).fetch();

// get all users with a reputation score of 50 or below, sorted
users.order("reputation").below({reputation: 50}, "closed").fetch();
```

## Collection.find {#find}

Retrieve a single document from a Collection.

```js
Collection.find(id | object)
```

The `find` method may be called with either a key-value pair to match against (e.g., `{name: "agatha"}` or an `id` value to look up.

```
const hz = Horizon();
const messages = hz("messages");

// get the first message from Bob
messages.find({from: "bob"});

// get the message with ID 101
messages.find({id: 101});

// because we are fetching by the document ID, we can use a shorthand
messages.find(101);
```

You can use the Observables `defaultIfEmpty` function to handle non-existing keys:
```py
var messages = hz('messages');
messages.find(id).fetch().defaultIfEmpty().subscribe((msg) => {
  if (msg == null) {
    console.log("Message not found");
    return;
  }
  ...
});
```

## Collection.findAll {#findall}

Retrieve multiple documents from a Collection.

```js
Collection.findAll(object[, object, ...])
```

The `findAll` method can be called with one or more key-value pairs to match against (e.g., `{email: "bob@example.com"}`. Every document that matches the pairs will be returned in a list. (If no documents match, an empty list, `[]`, will be returned.)

```
const hz = Horizon();
const messages = hz("messages");

// get all messages from Bob, Agatha and Dave
messages.findAll({from: "bob"}, {from: "agatha"}, {from: "dave"});

// get all messages from Jane and all messages with a high priority
messages.findAll({from: "jane"}, {priority: "high"});
```

## Collection.limit {#limit}

Limit the results of the query to a maximum number of returned documents.

```js
Collection.limit(integer)
```

The `limit` command takes a single argument, an integer representing the number of items to limit the results to.

```js
const hz = Horizon();
const users = hz("users");

// get the 10 most prolific posters
users.order("postCount", "descending").limit(10);
```

## Collection.order {#order}

Sort the results of the query by the values of a given field.

```js
Collection.order(field[, direction])
```

Fields passed to `order` may contain numbers, strings, or even arrays or objects; non-numeric values will be sorted lexicographically, and strings are sorted by UTF-8 codepoint. (Read about [Sorting order][so] and [ReQL data types][dt] in general.)

[so]: https://rethinkdb.com/docs/data-types/#sorting-order
[dt]: https://rethinkdb.com/docs/data-types/

The optional second argument must be a string indication sort direction, either `"ascending"` or `"descending"`. The default is `"ascending"`.

```js
const hz = Horizon();
const messages = hz("messages");

// get all messages, ordered ascending by ID value
messages.order("id", "ascending");

// "ascending" is the default, so it can be left out
messages.order("id");

// get all messages ordered by time, most recent first
messages.order("time", "descending");
```

## Collection.remove {#remove}

Delete a single document from a Collection.

```js
Collection.remove(id | object)
```

The `remove` method may be called with either an object to be deleted or an `id` value. In the object case, the object must include an `id` key.

```js
const hz = Horizon();
const messages = hz("messages");

// get the message with an ID of 101
var messageObject = messages.find(101);

// delete that object
messages.remove(messageObject);

// it may also be deleted by passing an object just with an ID key
messages.remove({id: 101});

// because we are deleting based on the document ID, we can use a shorthand
messages.remove(101);
```

## Collection.removeAll {#removeall}

Delete multiple documents from a Collection.

```js
Collection.removeAll([id, id, ...] | [object, object, ...])
```

The `removeAll` method must be called with an array of objects to be deleted, or `id` values to remove. The objects must have `id` keys. You can mix `id` values and objects within the array.

```js
const hz = Horizon();
const messages = hz("messages");

// get all messages from Bob and Agatha...
var messageList = messages.findAll({from: "bob"}, {from: "agatha"});

// ...and delete them
messages.removeAll(messageList);

// delete messages with the IDs 101, 103 and 109
messages.removeAll([101, 103, 109]);
```

## Collection.replace {#replace}

Replace one or more existing documents within a Collection.

```js
Collection.replace(object | list of objects)
```

The `replace` method can be called either with an object representing a single document, or a list of objects. The objects must have `id` values that already exist in the collection or an error will be raised.

```js
const hz = Horizon();
const messages = hz("messages");

// Replace a single document. This will raise an error if there is not an
// existing document in the messages collection with an id value of 1.
messages.replace({
    id: 1,
    from: "bob",
    text: "Hello from RethinkDB"
});

// Replace multiple documents at once. All documents must already exist.
messages.replace([
    {
        id: 2,
        from: "agatha",
        text: "Meet at Smugglers' Cove on Saturday"
    },
    {
        id: 3,
        from: "bob",
        text: "Would Superman lose in a fight against Wonder Woman?"
    }
]);
```

## Collection.store {#store}

Insert one or more new documents into a Collection.

```js
Collection.store(object | list of objects)
```

The `store` method can be called either with an object representing a single document, or a list of objects. The objects must have unique `id` values, and must have `id` values that do not already exist in the collection or an error will be raised.

```js
const hz = Horizon();
const messages = hz("messages");

// Store a single document. There must not be a document with an id of 1 in
// the messages collection.
messages.store({
    id: 1,
    from: "bob",
    text: "Hello from RethinkDB"
});

// Store multiple documents at once. All documents must be new.
messages.store([
    {
        id: 2,
        from: "agatha",
        text: "Meet at Smugglers' Cove on Saturday"
    },
    {
        id: 3,
        from: "bob",
        text: "Would Superman lose a fight against Wonder Woman?"
    }
]);
```

## Collection.upsert {#upsert}

Insert one or more documents into a Collection, replacing existing ones or inserting new ones based on `id` value.

```js
Collection.upsert(object | list of objects)
```

The `upsert` method can be called either with an object representing a single document, or a list of objects. Depending on whether the documents passed to `upsert` already exist in the collection as determined by their `id` values, the documents will either be inserted into the documents as new members of the collection (cf. [store](#store)) or replace the existing documents with the same `id` (cf. [replace](#replace)).

```js
const hz = Horizon();
const messages = hz("messages");

// Upsert (update or insert) a single document. If there is an existing
// document in the messages collection with an id of 1, it will be replaced;
// otherwise, it will be inserted.
messages.upsert({
    id: 1,
    from: "bob",
    text: "Hello from RethinkDB"
});

// Upsert multiple documents at once. The previously inserted document with
// id of 1 will be replaced.
messages.upsert([
    {
        id: 2,
        from: "agatha",
        text: "Meet at Smugglers' Cove on Saturday"
    },
    {
        id: 1,
        from: "bob",
        text: "Would Superman lose in a fight against Wonder Woman?"
    }
]);
```
