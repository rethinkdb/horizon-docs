---
layout: api
title: Collection.order()
---

# Method

{% apibody %}
Collection.order(field[, direction])
{% endapibody %}

# Description

Sort the results of the query by the values of a given field.

Fields passed to `order` may contain numbers, strings, or even arrays or objects; non-numeric values will be sorted lexicographically, and strings are sorted by UTF-8 codepoint. (Read about [Sorting order][so] and [ReQL data types][dt] in general.)

[so]: https://rethinkdb.com/docs/data-types/#sorting-order
[dt]: https://rethinkdb.com/docs/data-types/

The optional second argument must be a string indication sort direction, either `"ascending"` or `"descending"`. The default is `"ascending"`.

```
const messages = hz("messages");

// get all messages, ordered ascending by ID value
messages.order("id", "ascending");

// "ascending" is the default, so it can be left out
messages.order("id");

// get all messages ordered by time, most recent first
messages.order("time", "descending");
```

Also see: [Collection.limit][cl]

[cl]: /api/collection-limit/
