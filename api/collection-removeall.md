---
layout: api
title: Collection.removeAll()
---

# Method

{% apibody %}
Collection.removeAll([integer, integer, ...] | [object, object, ...])
{% endapibody %}

# Description

Delete multiple documents from a Collection.

The `removeAll` method must be called with an array of objects to be deleted, or integers representing ID values to remove. The objects must have `id` keys. You can mix integers and objects within the array.

```
const messages = hz("messages");

// get all messages from Bob and Agatha...
var messageList = messages.findAll({from: "bob"}, {from: "agatha"});

// ...and delete them
messages.removeAll(messageList);

// delete messages with the IDs 101, 103 and 109
messages.removeAll([101, 103, 109]);
```

Also see: [Collection.remove][cr].

[cr]: /api/collection-remove/
