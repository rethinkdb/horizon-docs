---
layout: api
title: Collection.fetch()
---

# Method

{% apibody %}
Collection.fetch({options})
{% endapibody %}

# Description

Return the entire contents of a [Collection][c].

[c]: /api/collection

With no options, `fetch` returns the contents of a Collection as an array. With the `asCursor` option set to `true`, an Observable will be returned instead. (Unlike [watch][cw], this Observable will _not_ be updated with changes; it represents a snapshot of the Collection as it existed when `fetch` was called.)

```js
const users = hz("users");

var userList = users.fetch();

console.log(userList);
```

The result will be something like:

```json
[
    {id: 1, name: 'bob', email: 'bob@example.com'},
    {id: 2, name: 'agatha', email: 'agatha@example.com'},
    {id: 3, name: 'skyla', email: 'skyla@example.com'}
]
```

The same query can be rewritten with an Observable:

```js
const users = hz("users");

users.fetch({asCursor: true}).forEach(
    result => console.log(result);
    err => console.error(err),
    () => console.log('Query finished')
);
```

Also see: [Collection.watch][cw].

[cw]: /api/collection-watch/
