---
layout: api
title: Collection.replace()
---

# Method

{% apibody %}
Collection.replace(object | list of objects)
{% endapibody %}

# Description

Replace one or more existing documents within a Collection.

The `replace` method can be called either with an object representing a single document, or a list of objects. The objects must have `id` values that already exist in the collection or an error will be raised.

```js
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

Also see: [Collection.store][cs], [Collection.upsert][cu]

[cs]: /api/collection-store/
[cu]: /api/collection-upsert/
