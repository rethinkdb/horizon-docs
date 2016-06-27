---
layout: documentation
title: Frequently asked questions
id: faq
permalink: /faq/
active: faq
---

## Why Horizon?

While technologies like [RethinkDB](http://www.rethinkdb.com) and
[WebSockets](https://en.wikipedia.org/wiki/WebSocket) make it possible to build
engaging realtime apps, empirically there is still too much friction for most
developers. Building realtime apps now requires understanding and manually
orchestrating multiple systems across the software stack, understanding
distributed stream processing, and learning how to deploy and scale realtime systems. The
learning curve is quite steep, and most of the initial work involves boilerplate
code that is far removed from the primary task of building a realtime app.

Horizon sets out to solve this problem. You can start building
realtime apps using your favorite front-end framework and Horizon's
APIs without having to write any backend code.

Since Horizon stores data in RethinkDB, once the app gets sufficiently
complex to need custom business logic on the backend, you can
incrementally add backend code at any time in the development cycle of
your app.

## How can I install Horizon? What are the prerequisites? ##

Horizon is built on top of RethinkDB and Node.js. You can install Horizon via `npm`:

    npm install -g horizon

For more details, read the [Installation guide][ig].

[ig]: /install

## How do I get started with Horizon? ##

You can create a new Horizon app and start it as follows:

    hz init my_app
    hz serve my_app --dev

For more details, see the [Getting Started][gs] guide.

[gs]: /docs/getting-started

## What does the code look like? ##

Here is an example of what you'd write on the front-end for a simple todo list application:

```js
// Connect to horizon
const horizon = Horizon();
const todoCollection = horizon("todo-items");

const todoApp = document.querySelector('#app')

// Function called when a user adds a todo item in the UI
const todoCollection.watch().subscribe( todos => {
  const todoHTML = todos.map(todo =>
    `<div class="todo" id="${todo.id}">
       <input type="checkbox" ${todo.done ? 'checked' : ''}>
       ${todo.text} -- ${todo.date}
     </div>`);
  todoApp.innerHTML = todoHTML.join('');
});
```

For more details, see the [Getting Started][gs] guide.

## How is Horizon different from Firebase? ##

There are a few major differences:

- Horizon is open-source. You can run it on your laptop, deploy it to
  the cloud, or deploy it to any infrastructure you want.
- Horizon will allow you to build complex enterprise apps, not just
  basic applications with limited functionality. Since Horizon stores
  data in RethinkDB, once your app grows beyond the basic Horizon API,
  you can start adding backend code of arbitrary complexity that has
  complete access to a fully-featured database.
- Since Horizon is built on RethinkDB, we'll be able to expose services
  that are much more sophisticated than simple document sync
  (e.g. realtime analytics, streams on joined tables, etc.)

## How is Horizon different from Meteor? ##

Horizon has philosophical and technical differences with Meteor that
will result in vastly different developer experiences.

Horizon is a small layer on top of RethinkDB with a narrow,
purpose-built API designed to make it very easy to get started
building realtime apps without backend code. Horizon isn't prescriptive
-- you can use any front-end framework without any
magic/customizations, and once your app outgrows the Horizon API you
can use any backend technology (e.g. Node.js, Python, Ruby) and any
backend framework (e.g. Rails, Express, Koa, etc.)

By contrast, Meteor is a much more prescriptive framework. Horizon is a
reasonably small component that has a very clean separation with the
database and the frontend, while Meteor is a much more monolithic
experience.

Another major difference is architectural -- Meteor uses a LiveQuery
component built by tailing MongoDB's oplog. This approach is
fundamentally limited -- it's impossible to do many operations
efficiently, and even the basic functionality is extremely difficult
to scale.

Horizon is built on RethinkDB, so the LiveQuery functionality is in the
database. This allows for much more sophisticated streaming operations,
and scalability is dramatically simpler because the database has all
the necessary information to allow for a scalable feeds implementation.

## What is the relationship between Horizon and RethinkDB? ##

Horizon is built by the RethinkDB team and uses RethinkDB as the
underlying database. We built Horizon because many RethinkDB users
asked for a simpler way to prototype, expand, and deploy realtime web
apps. Since the RethinkDB team has accumulated a lot of experience
helping many different teams build and scale realtime applications, we
designed Horizon to make this process dramatically easier and more
accessible. Our goal with RethinkDB and Horizon is to make it easier
to build JavaScript apps, and to make all apps realtime by default.

## Are there plans to support other databases? ##

While Horizon is designed to be database-independent, it is built from
the ground up based on a push architecture. RethinkDB has three
necessary properties for Horizon to function as a scalable realtime
backend: a realtime push architecture, the ability to open tens of
thousands of realtime streams per node, and the ability to scale
database nodes linearly. As other databases emerge with these
properties, we'll extend Horizon to support them.

## Do I need to be a full-stack developer to use Horizon? ##

Horizon takes care of all the necessary web server and database
challenges, such as setting up web sockets, handling connections, and
creating necessary endpoints. If you're a front-end developer, you can
use Horizon to build modern realtime apps without writing any backend
code.

## Is Horizon limited to simple applications? ##

Horizon exposes a powerful API that allows writing sophisticated
realtime applications. However, we carefully designed Horizon to
provide an upgrade path in case your application outgrows the Horizon
API. If you need to add backend functionality, instead of using the
standalone Horizon server you can require Horizon in Node.js as a
library, and mix and match Horizon services along with the rest of the
Node.js ecosystem. Since Horizon is built on top of RethinkDB, your
app will have full access to the underlying database, and you can
write arbitrarily complex code without being limited by Horizon's base
APIs and tools.

## How does Horizon scale? ##

Horizon relies on RethinkDB's clustering functionality for
scalability. You can spin up as many Horizon servers as necessary to
handle the web traffic, and shard the RethinkDB cluster to scale the
realtime data layer. Horizon and RethinkDB are designed to support
millions of realtime feeds and the combined push architecture makes
scaling your apps simple and painless.

## Can I use Horizon on mobile devices? ##

Horizon ships with a JavaScript client library that can be used for
building web and mobile apps with frameworks like React, Angular, or
React Native. We're also working on specific platform client libraries
that support the Horizon protocol. Client libraries for iOS and
Android are coming soon.

## How can I deploy Horizon apps? ##

Since Horizon is open-source, you can deploy Horizon apps in the cloud
(e.g. on Amazon AWS), a custom VPS, or on your own
infrastructure. We're also working on a Horizon Cloud service that
makes it possible to deploy and scale Horizon apps in a click of a
button.

## Does Horizon support GraphQL? ##

Currently Horizon exposes a JavaScript API and does not support
GraphQL. However, we're working on native [GraphQL
support](https://github.com/rethinkdb/horizon/issues/125) in Horizon,
and will make it available soon!

## How is Horizon licensed? ##

Both the Horizon server and the client library are released under the
MIT license.
