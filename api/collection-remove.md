---
layout: api
title: Collection.remove()
---

# Method

{% apibody %}
Collection.remove(integer | object)
{% endapibody %}

# Description

Delete a single document from a Collection.

The `remove` method may be called with either an object to be deleted or an integer representing a numeric ID value. In the object case, the object must include an `id` key.

```
const messages = hz("messages");

// get the message with an ID of 101
var messageObject = messages.find(101);

// delete that object
messages.remove(messageObject);

// it may also be deleted by passing an object just with an ID key
messages.remove({id: 101});

// because the id field contains integers, we can use a shorthand
messages.remove(101);
```

Also see: [Collection.removeAll][cra]

[cra]: /api/collection-removeall/
