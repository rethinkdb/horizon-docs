---
layout: documentation
title: Horizon authentication
id: auth
permalink: /docs/auth/
---

Horizon uses [JSON Web Tokens][jwt] for user authentication, an [open industry standard][rfc7519]. Depending on your application, you can choose one of three types of authentication handling by passing the `authType` option to the [Horizon object constructor][hoc].

[jwt]:     https://jwt.io
[rfc7519]: https://tools.ietf.org/html/rfc7519 "RFC 7519: JSON Web Token (JWT)"
[hoc]:     /docs-archive/client#horizon

* `unauthenticated`: share a single token among all users, and do not create entries in the Horizon user table. This essentially bypasses Horizon's authentication and permission system, and is best for applications that don't need to store any user data.
* `anonymous`: generate a unique token for each new user, and create an entry in the users table for the generated token. This allows authentication through the generated token, which is stored client-side in [localStorage][ls]. Your application will need to prompt for username and password.
* `token`: verify a user's identify via a third-party [OAuth][] service provider. As with `anonymous`, the returned JWT will be stored client-side.

[ls]:    https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
[OAuth]: http://oauth.net

# Using local authentication

## Unauthenticated

The first auth type is unauthenticated. One web token is shared by all unauthenticated users. To create a connection using the 'unauthenticated' method do:

```js
const horizon = Horizon({ authType: 'unauthenticated' });
```

This is the default authentication method and provides no means to separate user permissions or data in the Horizon application.

## Anonymous

The second auth type is anonymous. If anonymous authentication is enabled in the config, any user requesting anonymous authentication will be given a new JWT, with no other confirmation necessary. The server will create a user entry in the users table for this JWT, with no other way to authenticate as this user than by passing the JWT back. (This is done under the hood with the jwt being stored in localStorage and passed back on subsequent requests automatically).

```js
const horizon = Horizon({ authType: 'anonymous' });
```

This type of authentication is useful when you need to differentiate users but don't want to use a popular 3rd party to authenticate them. This is essentially the means of "Creating an account" or "Signing up" for people who use your website.

# Using OAuth

Your application will need a client ID and "secret" for each OAuth provider you want to connect with. The providers Horizon currently supports are:

* [Facebook](https://developers.facebook.com/apps/)
* [Github](https://github.com/settings/applications/new)
* [Google](https://console.developers.google.com/project)
* [Slack](https://api.slack.com/apps)
* [Twitch](https://www.twitch.tv/kraken/oauth2/clients/new)
* [Twitter](https://apps.twitter.com/app/new)

Each provider will let you register your application, and will give you the client data you need to give Horizon.

## Configuring the server

In order to use OAuth with Horizon, you'll need to configure a TLS certificate so you can serve assets with HTTPS, and either specify the `--key-file` and `--cert-file` options to `hz serve` or add them to the server's `.hz/config.toml` file. (See [The config.toml file][cf] for more details.) You can create a self-signed certificate with `hz create-cert`

[cf]: /docs/configuration

You'll need to enter `client_id` and `client_secret` values in the `.hz/config.toml` file for your server. Toward the bottom of the automatically generated file, you'll see commented-out sample OAuth settings. Uncomment the appropriate lines and replace the dummy values with the ones you've received from the application provider. Adding Github OAuth configuration data would look like this:

```toml
# [auth.facebook]
# id = "000000000000000"
# secret = "00000000000000000000000000000000"
#
# [auth.google]
# id = "00000000000-00000000000000000000000000000000.apps.googleusercontent.com"
# secret = "000000000000000000000000"
#
# [auth.twitter]
# id = "0000000000000000000000000"
# secret = "00000000000000000000000000000000000000000000000000"

[auth.github]
id = "your_client_id"
secret = "your_client_secret"
```

Verify the configuration by running `hz serve` and browsing to `https://localhost:8181/horizon/auth_methods`. (If you're not running on your local machine, change the server name and port as appropriate.) You'll see a list of currently active authentication options. For our example, you should see `github` included in the available auth methods object:

```js
{
  github: "/horizon/github"
}
```

If instead you only see empty brackets (e.g., `{ }`), ensure you've restarted the Horizon server, and that it's using the `.hz/config.toml` file you've edited.

## Configuring the client application

Use `authType: 'token'` when initializing Horizon in your client, and then use the `authEndpoint()` command to retrieve the endpoint for your OAuth identity provider and redirect the user to that URL.

```js
const horizon = Horizon({ authType: 'token' });
if (!horizon.hasAuthToken()) {
  horizon.authEndpoint('github').toPromise()
    .then((endpoint) => {
      window.location.pathname = endpoint;
    })
} else {
  // We have a token already, do authenticated Horizon stuff here
}
```

After logging in with Github, the user will be redirected back to the root page of your application, with the JSON Web Token in the `horizon_token` query parameter. The Horizon client will automatically save the JWT from the redirected URL, storing it client-side in [localStorage][ls] (just as with `anonymous` tokens). If the token is lost, the user can simply re-authenticate with the provider. User information will be stored in the `Users` table.

If an error occurs somewhere during the authentication process, the browser will be redirected back to the root of page with an error message in the query parameters.

## Clearing tokens

To delete all authentication tokens from localStorage, use `clearAuthTokens()`.

```js
// Note the 'H'
Horizon.clearAuthTokens();
```

## Notes about Horizon's OAuth support

* Currently, no metadata from OAuth providers&mdash;for example, friend/following lists&mdash; can be requested. In the near future, Horizon will support authentication scopes for selected providers to request acess to these details, and returned metadata will be stored in the `Users` table.
* The redirection URL will be configurable in a future release.
* [Passport][pp] integration is not currently on Horizon's roadmap; this decision was made to avoid tightly coupling Horizon with [Express][ex]. (A [Github issue][gi] may be opened to discuss this in the future.)

[pp]: http://passportjs.org
[ex]: http://expressjs.com
[gi]: https://github.com/rethinkdb/horizon/issues/new
