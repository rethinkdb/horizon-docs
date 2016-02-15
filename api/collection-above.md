---
layout: api
title: Collection.above()
---

# Method

{% apibody %}
Collection.above(integer | object[, "closed" | "open"])
{% endapibody %}

# Description

Restrict the range of results returned to values that sort above a given value.

The `above` method may be called with either a key-value pair to match against (e.g., `{name: "agatha"}` or an integer (an `id` value to look up). The second optional parameter must be the string `"closed"` or `"open"`, indicating that the specified value will be included (closed) or excluded (open) from the result set. The default is excluded (open): `above(10)` will return documents with `id` values higher than (but not equal to) 10.

Values in key-value pairs may be numbers, strings, or even arrays or objects; non-numeric values will be sorted lexicographically, and strings are sorted by UTF-8 codepoint. (Read about [Sorting order][so] and [ReQL data types][dt] in general.)

[so]: https://rethinkdb.com/docs/data-types/#sorting-order
[dt]: https://rethinkdb.com/docs/data-types/

The `above` method is often used in conjunction with [order][cor], but it may appear after any Horizon method with the exception of [find][cfi] and [limit][cli]. (However, `limit` may appear after `above`.)

```
// get all messages with an ID over 100, sorted
messages.order("id").above({id: 100});

// the same as above, but using the integer shorthand for ID
messages.order("id").above(100);

// get all messages with an ID between 101 and 200, sorted
messages.order("id").below(200).above(100);

// get all users with a reputation score of 50 or over, unsorted
users.above({reputation: 50}, "closed");
```

Also see: [Collection.below][cbe]

[cor]: /api/collection-order/
[cfi]: /api/collection-find/
[cli]: /api/collection-limit/
[cbe]: /api/collection-below/
