---
layout: documentation
title: Getting started with Horizon
id: getting-started
permalink: /docs/getting-started/
hero_image: /images/docs/getting-started.png
---

If you haven't installed Horizon, do so now. (Read the [Installation instructions][install] for more details.)

    npm install -g horizon

[install]: /install

* Table of Contents
{:toc}

## Using the Horizon CLI

Interactions with Horizon are performed with the `hz` application. `hz` has a number of commands, of which we are going to use the following two:

* `init [directory]`: initialize a new Horizon application
* `serve`: serve the project in the current directory

## Initialize an example application

Let's create a new Horizon application. Go to a directory you'd like to install this application into and type:

    hz init example_app

This will create the `example_app` directory and install a few files into it. (If you run `hz init` without giving it a directory, it will install these files into the current directory.)

```
$ tree example_app
example_app/
├── .hz
│   └── config.toml
├── dist
│   └── index.html
└── src
```
Here's what these files and directories are:

* `dist` is for static files. You can create files directly here, or use it as the output directory for a build system of your choice.
* `src` is for source files for your build system. This isn't a convention you have to follow; Horizon doesn't touch anything in this directory.
* `dist/index.html` is a sample file. You'll replace this as you develop your application, but there's enough in it to verify that Horizon is installed and working.
* `.hz/config.toml` is a [TOML][] configuration file for the Horizon server.

[TOML]: https://github.com/toml-lang/toml

## Start the server

Start Horizon to test it out:

    hz serve --dev

You'll see a series of output messages as Horizon starts a RethinkDB server, ending with `Metadata synced with server, ready for queries.` Now, go to <http://localhost:8181>. You should see the message "App works!" scrolling across your screen.

Here's what `hz serve` actually does:

* Start the Horizon API server, a Node.js application.
* Starts an HTTP server which serves the Horizon client library, `horizon.js`.

Passing the `--dev` flag to `hz serve` puts it in development mode, which makes the following changes. (All of these can also be set individually with separate flags to `serve`.)

* A RethinkDB server is automatically started (`--start-rethinkdb`). This server is specifically for this Horizon application, and will create a `rethinkdb_data` folder in the working directory when started.
* Horizon is served in "insecure mode," without requiring SSL/TLS (`--secure no`).
* The permissions system is disabled (`--permissions no`).
* Tables and indexes will automatically be created if they don't exist (`--auto-create-collection` and `--auto-create-index`).
* Static files will be served from the `dist` directory (`--serve-static ./dist`).

You can find the complete list of [command line flags][server] for `hz serve` in the documentation for the [Horizon server][server].

In production (i.e., without the `--dev` flag), you'll use the `.hz/config.toml` file to set these and other options. See [Configuring Horizon][config-file] for details.

[server]: /docs/server
[config-file]: /docs/configuration

## Talk to Horizon

Load the `index.html` file in `example_app`. It's pretty short:

```html
<!doctype html>
<html>
  <head>
    <meta charset="UTF-8">
    <script src="/horizon/horizon.js"></script>
    <script>
      var horizon = Horizon();
      horizon.onReady(function() {
        document.querySelector('h1').innerHTML = 'App works!'
      });
      horizon.connect();
    </script>
  </head>
  <body>
   <marquee><h1></h1></marquee>
  </body>
</html>
```

The two `script` tags do the work here. The first loads the actual Horizon client library, `horizon.js`; the second is a (very tiny) Horizon application:

* `var horizon = Horizon()` instantiates a [Horizon][ho] object. This object only has a few methods on it, for handling connection-related events and for instantiating Horizon [Collections][co].
* [onReady][hc] is an event handler that's executed when the client makes a successful connection to the server.
* Our connection function simply fills in `"App works!"` into the `<h1>` tag in the document. Since this function only gets executed on a successful connection, it *does* verify that Horizon is working, but it's not leveraging RethinkDB for anything yet.
* Also, we're sorry for the `<marquee>` tag.

[ho]: /api/horizon
[co]: /api/collection
[hc]: /api/horizon/#onready

## Horizon Collections

The heart of Horizon is the `Collection` object, which lets you store, retrieve, and filter documents. Many `Collection` methods for reading and writing documents return [RxJS][rx] Observables.

[rx]: http://reactivex.io/rxjs/

```js
// Create a "messages" collection
const chat = horizon("messages");
```

To store documents in the collection, use [store][st].

```js
let message = {
  text: "What a beautiful horizon!",
  datetime: new Date(),
  author: "@dalanmiller"
}

chat.store(message);
```

To retrieve documents, use [fetch][fe].

[st]: /api/collection/#store
[fe]: /api/collection/#fetch

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

We use the RxJS [subscribe][sub] method to receive items from the collection, as well as to provide an error handler.

[sub]: http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-subscribe

## Removing documents

To remove documents from a collection, use either [remove][rem] or [removeAll][rema].

[rem]:  /api/collection/#remove
[rema]: /api/collection/#removeall

```js
// These two queries are equivalent and will remove the document with id: 1
chat.remove(1).subscribe((id) => { console.log(id) })
chat.remove({id: 1}).subscribe((id) => {console.log(id)})

// Will remove documents with ids 1, 2, and 3 from the collection
chat.removeAll([1, 2, 3])
```

As with the other functions, you can chain `subscribe` onto the remove functions to provide response and error handlers.

## Watching for changes

We can "listen" to an entire collection, query, or a single document by using [watch][]. This lets us build apps that update state immediately as data changes in the database.

[watch]: /api/collection/#watch

```js
// Watch all documents. If any of them change, call the handler function.
chat.watch().subscribe((docs) => { console.log(docs)  })

// Query all documents and sort them in ascending order by datetime,
//  then if any of them change, the handler function is called.
chat.order("datetime").watch().subscribe((docs) => { console.log(docs)  })

// Watch a single document in the collection.
chat.find({author: "@dalanmiller"}).watch().subscribe((doc) => { console.log(doc) })
```

By default, the Observable returned from `watch` receives the entire collection of documents when a single one changes. This makes it easy to use frameworks such as [Vue][vue] or [React][react], allowing you to simply replace your existing copy of the collection with the new array returned by Horizon.

[vue]: https://vuejs.org/
[react]: https://facebook.github.io/react/

```js
let chats = [];

// Query chats with `.order`, which by default is in ascending order
chat.order("datetime").watch().subscribe(

  // Returns the entire array
  (newChats) => {

    // Here we replace the old value of `chats` with the new
    //  array. Frameworks such as React will re-render based
    //  on the new values inserted into the array.
    chats = newChats;
  },

  (err) => {
    console.log(err);
  })
```

To learn more about how Horizon works with React, check out the [complete Horizon & React example][apps].

[vue]: https://vuejs.org/
[react]: https://facebook.github.io/react/
[apps]: /docs/examples

## Putting it all together

Now that we have the basics covered, let's pretend we're building a simple chat application where the messages are displayed in ascending order.

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

chat.watch().subscribe(chats => {
    renderChats(allChats)
  },

  // When error occurs on server
  error => console.log(error),
)
```

You can also get notifications when the client connects and disconnects from the server:

```js
// Triggers when client successfully connects to server
horizon.onReady().subscribe(() => console.log("Connected to Horizon server"))

// Triggers when disconnected from server
horizon.onDisconnected().subscribe(() => console.log("Disconnected from Horizon server"))
```

From here, you could take any framework and add these functions to create a realtime chat application
without writing a single line of backend code!

## Integrating Horizon with an existing application

While Horizon serves static files from the `dist` folder by default, you can use any method to serve your files. You have two options to include the Horizon client library:

* Use `horizon.js` served by the Horizon server
* Install `@horizon/client` as a dependency in your project

We recommend the first option, as that will prevent any possibly mismatches between the client library version and the Horizon server. However, if you're using [Webpack][webpack] or a similar build setup, or requesting the `.js` library at load time isn't desirable, just add the client library as an NPM dependency (`npm install @horizon/client`).

In your application, you'll need to include the Horizon client file, and specify the Horizon port number when initializing the connection.

[webpack]: https://webpack.github.io/

```html
<script src="localhost:8181/horizon/horizon.js"></script>

// Specify the host property for initializing the Horizon connection
const horizon = Horizon({host: 'localhost:8181'});
```

## Further reading

* Read about [Authentication][auth], including easy integration with OAuth providers such as Twitter and Github.
* Read about Horizon's [Permissions][perm] and schema enforcement.
* Read the [Collection][co] and [Horizon][ho] API documentation.
* The [Horizon sample apps][apps] can show you how to integrate Horizon with popular frameworks such as React and Angular.

[auth]: /docs/auth
[perm]: /docs/permissions
