---
layout: api
title: Collection.upsert()
---

# Method

{% apibody %}
Collection.upsert(object | list of objects)
{% endapibody %}

# Description

Insert one or more new documents into a Collection.

The `upsert` method can be called either with an object representing a single document, or a list of objects. Depending on whether the documents passed to `upsert` already exist in the collection as determined by their `id` values, the documents will either be inserted into the documents as new members of the collection (cf. [store][cs]) or replace the existing documents with the same `id` (cf. [replace][cr]).

```js
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

Also see: [Collection.replace][cr], [Collection.store][cs]

[cr]: /api/collection-replace/
[cs]: /api/collection-store/
