---
layout: documentation
title: Frequently asked questions
id: faq
permalink: /faq
---

## How can I install Horizon? What are the prerequisites? ##

Horizon requires RethinkDB and Node.js. For more details, read the [Installation guide][ig].

[ig]: /install

## How do you start Horizon? ##

    npm install -g horizon
    hz init my-app
    hz serve my-app --dev

For more details, read the [Installation guide][ig] and the [Quickstart][qs].

[qs]: /quickstart

## What does the code look like? ##

Here is currently what you'd write on the front-end for a simple todo list application:

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

For more details, check out the [README][rc] for the Horizon client library.

[rc]: https://github.com/rethinkdb/horizon/tree/next/client#horizon-client-library

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

## How will Horizon be licensed? ##

We still have to figure out the exact license, but Horizon will be
fully open source (we'll probably use MIT or Apache).
