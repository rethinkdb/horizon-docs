---
layout: api
title: Collection.store()
---

# Method

{% apibody %}
Collection.store(object | list of objects)
{% endapibody %}

# Description

Insert one or more new documents into a Collection.

The `store` method can be called either with an object representing a single document, or a list of objects. The objects must have unique `id` values, and must have `id` values that do not already exist in the collection or an error will be raised.

```js
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
        text: "Would Superman lose in a fight against Wonder Woman?"
    }
]);
```

Also see: [Collection.replace][cr], [Collection.upsert][cu]

[cr]: /api/collection-replace/
[cu]: /api/collection-upsert/
