---
layout: documentation
title: Current limitations
id: limitations
permalink: /docs/limitations/
---

## Requirements

Horizon should be compatible with most popular browsers, although it hasn't been extensively tested with older versions. If you expect to run Horizon on browsers that do not support JavaScript ES6 promises, you will need to supply a polyfill to support them, such as [es6-promise][ep].

[ep]: https://github.com/stefanpenner/es6-promise

Currently, Horizon has issues with Mobile Safari. This will be addressed in an upcoming release.

There may be issues with Internet Explorer (see [issue #566][566]).

[566]: https://github.com/rethinkdb/horizon/issues/566

The Horizon server requires Node.js version 4.4 or higher, and the corresponding version of npm.

Setting up a Horizon server under Windows has not been extensively tested.

## Limitations

* Horizon operations are non-atomic (even with single documents).
* Horizon will not automatically retry writes that have version conflicts; while these writes will fail, it is up to the application to retry.
* User records for "anonymous" users need to be removed from the `users` collection manually after their JSON Web Tokens expire.
* The Horizon client cannot automatically reconnect to the Horizon server if the connection is lost.
* If the client manually reconnects to the server, active subscriptions will not automatically resume. They must be set up again by the client application.

## Planned features

* [GraphQL][gql] support (see [issue #125][125])
* Pagination (see [issue #31][31])
* Local user/password authentication (see [issue #176][176])
* Reconnection logic (see [issue #9][9])

[gql]: http://graphql.org
[125]: https://github.com/rethinkdb/horizon/issues/125
[31]:  https://github.com/rethinkdb/horizon/issues/31
[176]: https://github.com/rethinkdb/horizon/issues/176
[9]:   https://github.com/rethinkdb/horizon/issues/9