---
layout: api
title: Collection.watch()
---

# Method

{% apibody %}
Collection.watch({options})
{% endapibody %}

# Description

Convert a query into a [changefeed][feed]. This returns an Observable which receives real-time updates of the query's result set.

The `watch` command takes one option, `rawChanges`. If set `true`, your application will receive the actual change documents from the RethinkDB changefeed, rather than having Horizon update the result set for you based on the change documents. (Read the RethinkDB [changefeed documentation][feed] for more details on change documents.)

```js
// get a handle to the channels table in a multi channel chat application
const channels = hz("channels");

// receive all active channels, listing them every time a channel is
// added, deleted or changed
channels.watch().forEach(allChannels => {
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
channels.watch({rawChanges: true}).forEach(allChannels => {
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
channels.above({users: 10}, "closed").watch().forEach(allChannels => {
    console.log('Popular channels: ', allChannels);
});

// maintain an updating "top 10" channel list
channels.order("users", "descending").limit(10).watch()..forEach(allChannels => {
    console.log('Popular channels: ', allChannels);
});
```

Also see: [Collection.fetch][cf].

[feed]: https://rethinkdb.com/docs/changefeeds/javascript/
[cf]:   /api/collection-fetch/
