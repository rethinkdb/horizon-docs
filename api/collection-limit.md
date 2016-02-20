---
layout: api
title: Collection.limit()
---

# Method

{% apibody %}
Collection.limit(integer)
{% endapibody %}

# Description

Limit the results of the query to a maximum number of returned documents.

```
const users = hz("users");

// get the 10 most prolific posters
users.order("postCount", "descending").limit(10);
```

Also see: [Collection.order][co]

[co]: /api/collection-order/
