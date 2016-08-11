---
layout: documentation
title: Permissions and schema enforcement
id: permissions
permalink: /docs/permissions/
---

* Table of Contents
{:toc}

# The whitelist

Horizon's permission system is based on a query whitelist. Any operation on a Horizon collection is disallowed by default, unless there is a rule that allows the operation.

A whitelist rule has three properties that define which operations it covers:

* A [user group][users]
* A query template describing the type of operation
* An optional validator function written in JavaScript that can be used to check the contents of the accessed documents, or to implement more complex permission checks

[users]: /docs/users

You can use the special `"default"` group to create rules that apply to all users, authenticated or not. Or use the `"authenticated"` group to cover authenticated users only.

<div class="infobox" markdown="1">
**A Horizon application allows _no_ access to collections by default, even for authenticated users!** You can use rules with the `anyRead` and `anyWrite` placeholders on each collection to grant access:

```toml
[groups.authenticated.rules.read]
template = "collection('messages').anyRead()"

[groups.authenticated.rules.write]
template = "collection('messages').anyWrite()"
```

These rules would allow users in the `authenticated` group complete read and write access to the "messages" collection. Much finer-grained control is possible; read on for more information.
</div>

For example the following rule allows authenticated users to read their own messages from the `messages` collection:

```toml
[groups.authenticated.rules.read_own_messages]
template = "collection('messages').findAll({owner: userId()})"
```

This rule allows users to store new messages, as long as the new message has a correctly set `owner` field, a `message` field of type `string`, and no additional fields (apart from the auto-generated document ID):

```toml
[groups.authenticated.rules.store_message]
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

You can define whitelist rules and indexes as part of a schema file using the [TOML][toml] configuration format.

The format for a whitelist rule specification is as follows:

```toml
[groups.GROUP_NAME.rules.RULE_NAME]
template = "QUERY_TEMPLATE"
# Optional:
validator = "VALIDATOR_FUNCTION"
```

* `GROUP_NAME` is the name of a [user group][users] to which the rule should apply.
* `RULE_NAME` is an arbitrary name to identify the rule.
* `QUERY_TEMPLATE` must be a string that described the Horizon query that the rule applies to. See the section on [Query templates](#templates) for details.
* `VALIDATOR_FUNCTION` can be set to a string containing a JavaScript function value. See the section on [Validator functions](#validator_functions) for details.

You can have an arbitrary number of rule specifications in your schema file.

The format for an index specification is as follows:

```toml
[collections.COLLECTION_NAME]
[[collections.COLLECTION_NAME.indexes]]
fields = [['field_name']]
```

* `COLLECTION_NAME` is the name of the collection with the index.
* `fields` is an array of one or more arrays, each array containing one field name. (This format is for future extensibility, when Horizon will support nested-field indexes.)
    * One field name creates a [simple index][rdb-si].
    * Two or more field names create a [compound index][rdb-ci].

[rdb-si]: https://www.rethinkdb.com/docs/secondary-indexes/javascript/#simple-indexes
[rdb-ci]: https://www.rethinkdb.com/docs/secondary-indexes/javascript/#compound-indexes

Here is an example for a full schema file including collection and index specifications and the example rules from above:

```toml
[collections.messages]
[[collections.messages.indexes]]
fields = [['owner']]

[groups.authenticated.rules.read_own_messages]
template = "collection('messages').findAll({owner: userId()})"

[groups.authenticated.rules.store_message]
template = "collection('messages').store({owner: userId(), message: any()})"
validator = """
  (context, oldValue, newValue) => {
    return typeof newValue.message === 'string';
  }
"""
```

Load a schema file into the Horizon cluster with `hz schema apply`:

```bash
# Load the default schema, .hz/schema.toml
$ hz schema apply

# Load the schema from new_schema.toml
$ hz schema apply new_schema.toml
```

`hz schema apply` replaces all existing rule, collection and index specifications. If loading the new schema causes collections to be deleted, the `hz schema apply` command will error. If you are sure that you want to delete those collections, you can specify the `--force` flag.

The changed schema and permissions become effective immediately.

You can extract the current schema from a Horizon cluster with `hz schema save`:

```bash
# Save the current schema to the default file (.hz/schema.toml)
$ hz schema save

# Save the current schema to current_schema.toml
$ hz schema save -o current_schema.toml
```

[toml]: https://github.com/toml-lang/toml

# Query templates {#templates}

Query templates allow Horizon to determine which whitelist rule applies to a given query or write operation. Templates are specified in terms of a Horizon query.

The starting point for any template is a collection from the `collection` function, similar to how you write `horizon(<collection name>)` in your Horizon application.

A basic rule with a query template looks like this:

```toml
[groups.default.rules.list_messages]
template = "collection('public_messages')"
```

The `list_messages` rule allows read operations on the `public_messages` collection, such as:

```js
horizon('public_messages').fetch()
horizon('public_messages').watch()
horizon('public_messages').findAll({type: "announcement"}).fetch()
horizon('public_messages').order("year").fetch()
horizon('public_messages').order("year").above({year: 2015}).fetch()
```

You can specify additional operations behind the collection:

```toml
# Specifically allow listing public messages ordered by year
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
[groups.authenticated.rules.lookup_messages]
template = "collection('messages').findAll({owner: any()})"
```

You can also specify a list of legal values by passing them to the `any()` call:

```toml
# Allow users to look up messages of type "shared" or "announcement"
[groups.authenticated.rules.lookup_public_messages]
template = "collection('messages').findAll({type: any('shared', 'announcement')})"
```

### `anyRead()` {#template-placeholders-anyread}

The `anyRead()` placeholder matches any read operation or chain of read operations that can be performed on a collection.

Horizon implicitly adds `anyRead()` to the end of any read template, unless the template ends in `fetch()` or `watch()`. This is useful because adding additional read operations can only further restrict the set of returned results, but never allow access to additional results.

To allow only a specific read query, just finish the template off with `fetch()` or `watch()`:

```toml
[groups.default.rules.list_messages_any]
template = "collection('public_messages').fetch()"
```

This restricts the allowed queries as follows:

```js
// This is still ok:
horizon('public_messages').fetch()

// These queries no longer match the template:
horizon('public_messages').watch()
horizon('public_messages').findAll({type: "announcement"}).fetch()
horizon('public_messages').order("year").fetch()
horizon('public_messages').order("year").above({year: 2015}).fetch()
```

### `anyWrite()` {#template-placeholders-anywrite}

The `anyWrite()` placeholder matches `store`, `replace`, `upsert`, `remove`, and `removeAll` operations.

```toml
# Allow users in the group "admin" to perform any write operation on `messages`:
[groups.admin.rules.write_messages]
template = "collection('messages').anyWrite()"
```

### `userId()` {#template-placeholders-userid}

The `userId()` placeholder matches the ID of the currently authenticated user (see [Authentication][auth]). If no user is currently authenticated, it will match the `null` value.

```toml
# Allow users to read their own messages
[groups.authenticated.rules.read_own_messages]
template = "collection('messages').findAll({owner: userId()})"
```

[auth]: /docs/auth

# Validator functions {#validator_functions}

Validator functions further restrict which operations a whitelist rule allows. In contrast to query templates which are static rules on the query structure, validator functions can be arbitrary JavaScript functions that take the data itself into account.

The main use cases for validator functions are:

* enforcing a data schema, including restrictions on value types or number ranges
* implementing advanced restrictions that cannot be expressed with a query template

A validator function receives either two or three arguments. For _read_ operations:

* `context` is the current user document, as described in [Users and groups][users], or `null` if no user is logged in
* `value` is the document being read

For _write_ operations:

* `context` is the current user document, as described in [Users and groups][users], or `null` if no user is logged in
* `oldValue` contains the existing document before any write options, or `null` if a new document is being inserted
* `newValue` contains the new document the application wants to write, or `null` if a document is being removed

The validator function is expected to return either `true` or `false`.

This whitelist rule allows store operations as long as the stored documents match a certain schema:

```toml
[groups.authenticated.rules.store_message]
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
[groups.authenticated.rules.store_message]
template = "collection('messages').replace({id: any(), counter: any()})"
# The counter can only be incremented by one step at a time
validator = """
  (context, oldValue, newValue) => {
    return newValue.counter == oldValue.counter + 1;
  }
"""
```

## Execution semantics {#validator_functions-semantics}

Validator functions are executed in a restricted context. They cannot perform any I/O and cannot access external data other than what is passed to the validator function through its arguments.

The validator function of any matching rule is called once for each document that an operation touches. In case of a read query, every result document is validated through the validator function. For write operations, the validator function is executed atomically on the latest version of each document before the document is written. (If there are many concurrent writes being made to the same document, it is possible for this step to fail, but an error will be thrown to the application in such cases.)

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
template = "collection('integers')"
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
template = "collection('integers')"
validator = """
  (context, value) => {
    return value.id % 2 == 0;
  }
"""
```

While there is no single rule that validates all results of the query, for each result there now is a matching rule for which the validator function passes.

# Making an admin auth token {#admin}

To log in as the admin user initially, your application will need to be bootstrapped using the [hz make-token](/cli/#make-token) command.

```sh
hz make-token admin
```

A JSON Web Token will be printed to the console. Copy that token, and create a Horizon object with it:

```js
var horizon = Horizon({
    authType: {
        token: "<token>",
        storeLocally: false
    }
});
horizon.connect();
```

(The `storeLocally` option controls whether the token should be preserved in the browser's local storage area; if you set it to `true`, you'll remain logged in as the admin from this browser.)

The `make-token` command can be used to create a token for any user, whether or not that user already exists in Horizon's user database. If you wished to manually create a token for a user with the ID value of '4C720BD1-2729-46BA-9213-ED84DEDE3120`:

```sh
hz make-token 4C720BD1-2729-46BA-9213-ED84DEDE3120
```
