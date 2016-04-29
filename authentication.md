---
layout: documentation
title: Horizon authentication
---

Horizon uses [JSON Web Tokens][jwt] for user authentication, an [open industry standard][rfc7519]. Depending on your application, you can choose one of three types of authentication handling by passing the `authType` option to the [Horizon object constructor][hoc].

[jwt]:     https://jwt.io
[rfc7519]: https://tools.ietf.org/html/rfc7519 "RFC 7519: JSON Web Token (JWT)"
[hoc]:     /horizon/#constructor

* `unauthenticated`: share a single token among all users, and do not create entries in the Horizon user table. This essentially bypasses Horizon's authentication and permission system, and is best for applications that don't need to store any user data.
* `anonymous`: generate a unique token for each new user, and create an entry in the users table for the generated token. This allows authentication through the generated token, which is stored client-side in [localStorage][ls]. Your application will need to prompt for username and password.
* `token`: verify a user's identify via a third-party [OAuth][] service provider. As with `anonymous`, the returned JWT will be stored client-side.

[ls]:    https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
[OAuth]: http://oauth.net

The first two types 