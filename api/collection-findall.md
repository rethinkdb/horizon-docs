---
layout: api
title: Collection.findAll()
---

# Method

{% apibody %}
Collection.findAll(object[, object, ...])
{% endapibody %}

# Description

Retrieve multiple documents from a Collection.

The `findAll` method can be called with one or more key-value pairs to match against (e.g., `{email: "bob@example.com"}`. Every document that matches the pairs will be returned in a list. (If no documents match, an empty list, `[]`, will be returned.)

```
const messages = hz("messages");

// get all messages from Bob, Agatha and Dave
messages.findAll({from: "bob"}, {from: "agatha"}, {from: "dave"});

// get all messages from Jane and all messages with a high priority
messages.findAll({from: "jane"}, {priority: "high"});
```

Also see: [Collection.find][cf]

[cf]: /api/collection-find/
