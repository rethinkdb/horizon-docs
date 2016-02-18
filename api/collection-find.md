---
layout: api
title: Collection.find()
---

# Method

{% apibody %}
Collection.find(integer | object)
{% endapibody %}

# Description

Retrieve a single document from a Collection.

The `find` method may be called with either a key-value pair to match against (e.g., `{name: "agatha"}` or an integer (an `id` value to look up).

```
const messages = hz("messages");

// get the first message from Bob
messages.find({from: "bob"});

// get the message with ID 101
messages.find({id: 1});

// because the id field contains integers, we can use a shorthand
messages.find(101);
```

Also see: [Collection.findAll][cfa]

[cfa]: /api/collection-findall/
