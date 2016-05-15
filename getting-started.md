---
layout: documentation
title: Getting started with Horizon
id: getting-started
permalink: /docs/getting-started
---

If you haven't installed Horizon, do so now. (Read the [Installation instructions][install] for more details.)

    npm install -g horizon

[install]: /install

## Initialization

First, create a new Horizon project by using `hz init`:

```bash
hz init myApp
cd myApp
```

In the boilerplate created by `hz init`, you can see that the Horizon client library is being
imported from the path `/horizon/horizon.js` served by the Horizon server.


```html
...
<head>
  ...
  <script src="/horizon/horizon.js"></script>
</head>
...
```

After this script is loaded, you can connect to your running instance of the Horizon server.


```js
const horizon = Horizon();

// Calling `.connect()` is optional. If you omit it, the connection will be established
// on first use of the `horizon` object.
horizon.connect();
```

From here you can start to interact with Horizon collections. Having `--dev` mode enabled on
the Horizon Server creates collections and indexes automatically so you can get your
application setup with as little hassle as possible.

> **Note:** With `--dev` mode enabled, collections and indexes will
be created automatically, once you run a query that uses them.

```js
// This automatically creates the "messages" collection
const chat = horizon("messages");
```

Now, `chat` is a Horizon collection of documents. You can perform a
variety of operations on this collection to filter them down to the ones
you need. This most basic operations are [`.store`][store] and [`.fetch`][fetch]:

### Storing documents

To store documents into the collection, we use [`.store`][store].

```js
// Object being stored
let message = {
  text: "What a beautiful horizon!",
  datetime: new Date(),
  author: "@dalanmiller"
}

// Storing a document
chat.store(message);
```

If we wanted, we could also add `.subscribe` at the end of [`.store`][store] and handle the document `id`s created by the server as well as any errors that occur with storing. Check out the `[`store` documentation][store] for details.

## Retrieving documents

To retrieve messages from the collection we use [`.fetch`][fetch]. Here, we pass both a result and an error handler function to `.subscribe`.

```js
chat.fetch().subscribe(
  (items) => {
    items.forEach((item) => {
      // Each result from the chat collection
      //  will pass through this function
      console.log(item);
    })
  },
  // If an error occurs, this function
  //  will execute with the `err` message
  (err) => {
    console.log(err);
  })
```

## Removing documents

To remove documents from a collection, you can use either [`.remove`][remove] or [`.removeAll`][removeAll]:

```js
// These two queries are equivalent and will remove the document with id: 1.
chat.remove(1).subscribe((id) => { console.log(id) })
chat.remove({id: 1}).subscribe((id) => {console.log(id)})
```

Or, if you have a set of documents that you'd like to remove you can pass them in as an array to [`.removeAll`][removeAll].

```js

// Will remove documents with ids 1, 2, and 3 from the collection.
chat.removeAll([1, 2, 3])
```
As with the other functions, you can chain `.subscribe` onto the remove functions and provide response and error handlers.

## Watching for changes

We can also "listen" to an entire collection, query, or a single document by using [`.watch`][watch].
This is very convenient for building apps that want to update state immediately as data changes
in the database. Here are a few variations of how you can use [`.watch`][watch]:

```js
// Watch all documents, if any of them change, call the handler function.
chat.watch().subscribe((docs) => { console.log(docs)  })

// Query all documents and sort them in ascending order by datetime,
//  then if any of them change, the handler function is called.
chat.order("datetime").watch().subscribe((docs) => { console.log(docs)  })

// Find a single document in the collection, if it changes, call the handler function
chat.find({author: "@dalanmiller"}).watch().subscribe((doc) => { console.log(doc) })
```

By default, the handler you pass to `.subscribe` chained on [`.watch`][watch] will receive
the entire collection of documents when one of them changes. This makes it easy when
using frameworks such as [Vue][vue] or [React][react]
allowing you to replace the current state with the new array given to you by Horizon.

```js

// Our current state of chat messages
let chats = [];

// Query chats with `.order` which by default
//  is in ascending order.
chat.order("datetime").watch().subscribe(

  // Returns the entire array
  (newChats) => {

    // Here we replace the old value of `chats` with the new
    //  array. Frameworks such as React will re-render based
    //  on the new values inserted into the array. Preventing you
    //  from having to do modifications on the original array.
    //
    // In short, it's this easy! :cool:
    chats = newChats;
  },

  (err) => {
    console.log(err);
  })
```

To learn more about how Horizon works with React, check out the [complete Horizon & React example][example-apps].

[vue]: https://vuejs.org/
[react]: https://facebook.github.io/react/

# Putting it all together

Now that we have the basics covered, let's pretend we are building a
simple chat application where the messages are displayed
in ascending order. Here are some basic functions that would allow
you to build such an app.

```js

let chats = [];

// Retrieve all messages from the server
const retrieveMessages = () => {
  chat.order('datetime')
  // fetch all results as an array
  .fetch()
  // Retrieval successful, update our model
  .subscribe((newChats) => {
      chats = chats.concat(newChats);
    },
    // Error handler
    error => console.log(error),
    // onCompleted handler
    () => console.log('All results received!')
    )
};

// Retrieve an single item by id
const retrieveMessage = id => {
  chat.find(id).fetch()
    // Retrieval successful
    .subscribe(result => {
      chats.push(result);
    },
    // Error occurred
    error => console.log(error))
};

// Store new item
const storeMessage = (message) => {
   chat.store(message)
    .subscribe(
      // Returns id of saved objects
      result => console.log(result),
      // Returns server error message
      error => console.log(error)
    )
};

// Replace item that has equal `id` field
//  or insert if it doesn't exist.
const updateMessage = message => {
  chat.replace(message);
};

// Remove item from collection
const deleteMessage = message => {
  chat.remove(message);
};
```

And lastly, the [`.watch`][watch] method basically creates a listener on the chat collection. Using just `chat.watch()`, and the new updated results will be pushed to you any time they change on the server. You can also [`.watch`][watch] changes on a query or a single document.


```js

chat.watch().subscribe(chats => {
  // Each time through it will returns all results of your query
    renderChats(allChats)
  },

  // When error occurs on server
  error => console.log(error),
)
```

You can also get notifications when the client connects and disconnects from the server

``` js
  // Triggers when client successfully connects to server
  horizon.onReady().subscribe(() => console.log("Connected to Horizon Server"))

  // Triggers when disconnected from server
  horizon.onDisconnected().subscribe(() => console.log("Disconnected from Horizon Server"))
```

From here, you could take any framework and add these functions to create a realtime chat application
without writing a single line of backend code.

There's also plenty of other functions in the Horizon Client library to meet your needs, including:
[above][above], [below][below], [limit][limit], [replace][replace], and [upsert][upsert].


# Bringing your app to Horizon

If you already have an application in place but want to leverage
the power of Horizon for your realtime data, here are a few scenarios that will
be relevant to you:

## Do I need to output all my files into the `dist` folder?

The short and long answer is, **_no_**.

If you are already using some other process to serve your static files, you absolutely
do not need to now do Yet Another Refactor™️ just to get the power of Horizon. From your already existing code base you have two options to get include and then `require` the Horizon Client library:

1. Use `horizon.js` served by Horizon Server (simplest option)
1. Install `@horizon/client` as a dependency in your project

We recommend using the `horizon.js` library as served by Horizon Server for solely the
reason that there will be no mismatches between your client library version and your
current running version of Horizon Server.

This means somewhere in your application, you'll need to have:

```html
<script src="localhost:8181/horizon/horizon.js"></script>
```

And then when you init the Horizon connection you need to specify the `host` property:

```js
const horizon = Horizon({host: 'localhost:8181'});
```

However, if requesting the .js library at page load time isn't desirable, or you are using [webpack][webpack] and similar build setups for your front-end code, just add `npm install @horizon/client` to your project, and dependency wise, you'll be good to go.

Just remember that when you make connections to Horizon Server to specify the port number (which is by default `8181`) when connecting.

[webpack]: https://webpack.github.io/

## How do I add Horizon to X?

If you already have an application written with React, Angular, or another framework, you should first check our [example applications](/example-apps) for different ways on how we have integrated Horizon into these frameworks.



[above]: /api/collection#above
[below]: /api/collection#below
[Collection]: /api/collection
[fetch]: /api/collection#fetch
[find]: /api/collection#find
[findAll]: /api/collection#findall
[Horizon]: /api/horizon
[limit]: /api/collection#limit
[order]: /api/collection#order
[remove]: /api/collection#remove
[removeAll]: /api/collection#removeall
[replace]: /api/collection#replace
[store]: /api/collection#store
[upsert]: /api/collection#upsert
[watch]: /api/collection#watch
