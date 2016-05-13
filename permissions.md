---
layout: documentation
title: Permissions and schema enforcement
---

Horizon's permission system is based on a query whitelist. Any operation on a Horizon collection is disallowed by default, unless there is a rule that allows the operation.

A whitelist entry has three properties that define which operations it covers:
* A user group (TODO: reference), or the `"default"` group if it should apply to all users
* A query template describing the type of operation
* Optionally: One or multiple JavaScript functions that can be used to validate the contents of the accessed documents, or to implement more complex permission checks

A given document can be read or written to by a user, if there is at least one rule on the whitelist for which
* the user is a member of the specified group,
* the query template matches the read/write operation,
* and *all* specified validation functions return a `true` value.

Note that permissions are not enforced when running the Horizon server in development mode (TODO: reference). This is so that you can easily change and prototype the queries in your application without having to deal with maintaining corresponding entries in the white list.

# Configuring rules {#configuring}

## Through a configuration file

TODO: Syntax & Example
TODO: How to load the configuration into horizon

## Using ReQL

TODO: How to access & Example
TODO: When they become effective
TODO: How to export the rules

# Query templates {#templates}

TODO: Matching semantics and a few examples

## Placeholders {#template-placeholders}

There are three special placeholders you can use in a query template.

### `any()` {#template-placeholders-any}

TODO: Description and examples

### `any_read()` {#template-placeholders-any_read}

TODO: Description and example

### `any_write()` {#template-placeholders-any_write}

TODO: Description and example

# Validation functions {#validation_functions}

TODO: Overview. Can be used for: * schema enforcement, * advanced security rules
TODO: Example rules for both use cases
TODO: Formal properties: Arguments & return type (for reads & writes). Execution context. Matching semantics (all need to return `true`).

## The `context` object (#validation_functions-context)
