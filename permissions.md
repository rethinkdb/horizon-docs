---
layout: documentation
title: Permissions and schema enforcement
id: permissions
permalink: /docs/permissions
---

Horizon's permission system is based on a query whitelist. Any operation on a Horizon collection is disallowed by default, unless there is a rule that allows the operation.

A whitelist rule has three properties that define which operations it covers:

* A [user group][users], or the `"default"` group if it should apply to all users (TODO: Marc says there might also be an `"unauthenticated"` group. We should mention it here and probably use it in all the examples below as well, so they work without setting up authentication first.)
* A query template describing the type of operation
* Optionally: A validator function written in JavaScript that can be used to check the contents of the accessed documents, or to implement more complex permission checks

[users]: /docs/users

For example the following rule allows users in the `"default"` group to read their own messages from the `messages` collection:

```toml
[groups.default.rules.read_own_messages]
template = "collection('messages').findAll({owner: userId()})"
```

This rule allows users to store new messages, as long as the new message has a correctly set `owner` field, a `message` field of type `string`, and no additional fields (apart from the auto-generated document ID):

```toml
[groups.default.rules.store_message]
template = "collection('messages').store({owner: userId(), message: any()})"
# The template allows any value for the `message` field. We restrict it to strings by
# using a validator function:
validator = """
  // In case of a `store`, `oldValue` is always `null`.
  // `newValue` is the document that's getting stored, and `context` provides additional
  // details about the current user.
  (context, oldValue, newValue) => {
    return typeof newValue.message === 'string';
  }
"""
```

A given document can be read or written to by a user, if there is at least one rule on the whitelist for which

* the user is a member of the specified group,
* the query template matches the read or write operation,
* and the specified validator function (if any) returns a `true` value.

Note that permissions are not enforced when running the Horizon server in [development mode][dev-mode]. This is so that you can easily change and prototype the queries in your application without having to deal with maintaining corresponding entries in the whitelist.

[dev-mode]: /docs/server#development-mode

# Configuring rules {#configuring}

You can define whitelist rules as part of a schema file using the [TOML][toml] configuration format.

The format for a whitelist rule specification is as follows:

```toml
[groups.GROUP_NAME.rules.RULE_NAME]
template = "QUERY_TEMPLATE"
# Optional:
validator = "VALIDATOR_FUNCTION"
```

The values of the fields are as follows:

* `GROUP_NAME` is the name of a [user group][users] to which the rule should apply.
* `RULE_NAME` is an arbitrary name to identify the rule.
* `QUERY_TEMPLATE` must be a string that described the Horizon query that the rule applies to. See the section on [Query templates](#templates) for details.
* `VALIDATOR_FUNCTION` can be set to a string containing a JavaScript function value. See the section on [Validator functions](#validator_functions) for details.

You can have an arbitrary number of rule specifications in your schema file.

Here is an example for a full schema file including collection and index specifications and the example rules from above:

```toml
[collections.messages]
indexes = [
  "id",
  "owner"
]

[groups.default.rules.read_own_messages]
template = "collection('messages').findAll({owner: userId()})"

[groups.default.rules.store_message]
template = "collection('messages').store({owner: userId(), message: any()})"
validator = """
  (context, oldValue, newValue) => {
    return typeof newValue.message === 'string';
  }
"""
```

You can extract the current schema from a Horizon cluster with the `hz get-schema` command:

```bash
# Export the current schema into `current_schema.toml`
$ hz get-schema -o current_schema.toml
```

The `hz set-schema` command loads the new schema into the Horizon cluster:

```bash
# Import the schema from `new_schema.toml`
$ hz set-schema new_schema.toml
```

`hz set-schema` replaces all existing rule, collection and index specifications. If loading the new schema causes collections to be deleted, the `hz set-schema` command will error. If you are sure that you want to delete those collections, you can specify the `--force` flag.

The changed schema and permissions become effective immediately.

[toml]: https://github.com/toml-lang/toml

# Query templates {#templates}

Query templates allow Horizon to determine which whitelist rule applies to a given query or write operation. Templates are specified in terms of a Horizon query.

The starting point for any template is a collection from the `collection` function, similar to how you write `horizon(<collection name>)` in your Horizon application.

A basic rule with a query template looks like this:

```toml
[groups.default.rules.list_messages]
template = "collection('public_messages')"
```

The `list_messages` rule allows operations as follows:

```js
// This is ok:
horizon('public_messages')

// These are not ok:
horizon('public_messages').findAll({type: "announcement"})
horizon('public_messages').order("year")
horizon('public_messages').order("year").above({year: 2015})
```

You can specify additional operations behind the collection:

```toml
# Allow listing public messages ordered by year
[groups.default.rules.list_messages_by_year]
template = "collection('public_messages').order('year')"
```

Placeholders can be used to enable a rule to match a number of different, but related queries:

```toml
# Allow storing messages with arbitrary contents and year values
[groups.default.rules.list_messages_by_year]
template = "collection('messages').store({message: any(), year: any()})"
```

## Placeholders {#template-placeholders}

There are four special placeholders you can use in a query template.

### `any()` {#template-placeholders-any}

The `any()` placeholder matches any value.

```toml
# Allow users to look up messages from any user
[groups.default.rules.lookup_messages]
template = "collection('messages').findAll({owner: any()})"
```

You can also specify a list of legal values by passing them to the `any()` call:

```toml
# Allow users to look up messages of type "shared" or "announcement"
[groups.default.rules.lookup_public_messages]
template = "collection('messages').findAll({type: any('shared', 'announcement')})"
```

### `anyRead()` {#template-placeholders-anyread}

The `anyRead()` placeholder matches any read operation or chain of read operations that can be performed on a collection.

It is often useful to allow additional read operations behind a specified template, since those operations can only further restrict the set of returned results, but never allow access to additional results. We can adapt the example rule from above to allow additional operations:

```toml
[groups.default.rules.list_messages_any]
template = "collection('public_messages').anyRead()"
```

Now all of these are allowed:

```js
// These are all ok now:
horizon('public_messages')
horizon('public_messages').findAll({type: "announcement"})
horizon('public_messages').order("year")
horizon('public_messages').order("year").above({year: 2015})
```

`anyRead()` is not limited to whole collections. The following rule allows users to read their own messages, allowing them to add further restrictions or ordering constraints on the results:

```toml
[groups.default.rules.read_own_messages]
template = "collection('messages').findAll({owner: userId()}).anyRead()"
```

### `anyWrite()` {#template-placeholders-anywrite}

The `anyWrite()` placeholder matches `store`, `replace`, `upsert`, `remove`, and `removeAll` operations.

```toml
# Allow users in the group "admin" to perform any write operation on `messages`:
[groups.admin.rules.write_messages]
template = "collection('messages').anyWrite()"
```

### `userId()` {#template-placeholders-userid}

The `userId()` placeholder matches the ID of the currently authenticated user (see [Authentication][authentication]). If no user is currently authenticated, it will match the `null` value.

```toml
# Allow users to read their own messages
[groups.default.rules.read_own_messages]
template = "collection('messages').findAll({owner: userId()})"
```

[authentication]: /docs/authentication

# Validator functions {#validator_functions}

Validator functions further restrict which operations a whitelist rule allows. In contrast to query templates which are static rules on the query structure, validator functions can be arbitrary JavaScript functions that take the data itself into account.

The main use cases for validator functions are:

* enforcing a data schema, including restrictions on value types or number ranges
* implementing advanced restrictions that cannot be expressed with a query template

This whitelist rule allows store operations as long as the stored documents match a certain schema:

```toml
[groups.default.rules.store_message]
template = "collection('messages').store(any())"
# We require the message to be of the form {id: number, message: string} and have no
# extra fields.
validator = """
  (context, oldValue, newValue) => {
    return newValue.hasOwnProperty('id')
        && typeof newValue.id === 'number'
        && newValue.hasOwnProperty('message')
        && typeof newValue.message === 'string'
        && Object.keys(newValue).length == 2;
  }
"""
```

We can also use a validator to restrict the types of updates that can be performed on a document. Here we add a rule that allows users to increment a counter value:

```toml
[groups.default.rules.store_message]
template = "collection('messages').replace({id: any(), counter: any()})"
# The counter can only be incremented by one step at a time
validator = """
  (context, oldValue, newValue) => {
    return newValue.counter == oldValue.counter + 1;
  }
"""
```

A validator function for a read operation is called with two arguments, `context` and `value`. A validator function for a write operation is called with three arguments, `context`, `oldValue` and `newValue`. If a new document is inserted into the collection, `oldValue` is passed in as `null`. If a document gets removed, `newValue` will be `null`. The validator function is expected to return either `true` or `false`.

## Execution semantics {#validator_functions-semantics}

Validator functions are executed in a restricted context. They cannot perform any i/o and cannot access external data other than what is passed to the validator function through its arguments.

The validator function of any matching rule is called once for each document that an operation touches. In case of a read query, every result document is validated through the validator function. For write operations, the validator function is executed for each document before it gets stored, replaced or removed by the write.

If an operation encounters a document for which no matching rule with a passing validator function exists, the operation will error.

To understand how validator functions are executed, consider the following `integers` collection:

```js
horizon('integers').store([
    {id: 1},
    {id: 2},
    {id: 3},
    {id: 4}
  ])
```

We can use the following rule to enable reading only odd integers from this collection:

```toml
[groups.default.rules.read_odd]
template = "collection('integers').anyRead()"
validator = """
  (context, value) => {
    return value.id % 2 == 1;
  }
"""
```

The semantics of this rule are straight-forward when reading individual documents:

```js
// This will succeed
horizon('integers').find(1)

// This will error
horizon('integers').find(2)
```

Now consider the query

```js
horizon('integers')
```

This query will error as soon as it encounters an even integer.

Adding a second whitelist rule that allows reading even integers will make the query valid:

```toml
[groups.default.rules.read_even]
template = "collection('integers').anyRead()"
validator = """
  (context, value) => {
    return value.id % 2 == 0;
  }
"""
```

While there is no single rule that validates all results of the query, for each result there now is a matching rule for which the validator function passes.


## The `context` object {#validator_functions-context}

TODO: Describe the properties that are available in `context`
