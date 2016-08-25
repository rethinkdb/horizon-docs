---
layout: documentation
title: Horizon authentication
id: auth
permalink: /docs/auth/
---

Horizon uses [JSON Web Tokens][jwt] for user authentication, an [open industry standard][rfc7519]. Depending on your application, you can choose one of three types of authentication handling by passing the `authType` option to the [Horizon object constructor][hoc].

[jwt]:     https://jwt.io
[rfc7519]: https://tools.ietf.org/html/rfc7519 "RFC 7519: JSON Web Token (JWT)"
[hoc]:     /api/horizon/#constructor

* `unauthenticated`: do not generate a web token, and do not create entries in the Horizon user table. This essentially bypasses Horizon's authentication system, and is best for applications that don't need to store any user data.
* `anonymous`: generate a unique token for each new user, and create an entry in the users table for the generated token. This allows authentication through the generated token, which is stored client-side in [localStorage][ls].
* `token`: verify a user's identify via a third-party [OAuth][] service provider. As with `anonymous`, the returned JWT will be stored client-side.

[ls]:    https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
[OAuth]: http://oauth.net

You may also pass an already-created JWT to `authType`; this is useful for bootstrapping the admin user, or for integrating with non-browser-based authentication methods. Read "Making an admin auth token" in [Permissions and schema enforcement][admin] for more details.

[admin]: /docs/permissions/#admin

# Using local authentication

<div class="infobox" markdown="1">
Note that Horizon does _not_ have any "baked-in" local user/password authentication. The only fully-supported authentication method for individual user accounts is OAuth through a third-party provider.
</div>

## Unauthenticated

The first auth type is unauthenticated. This creates no web token, and Horizon does no user management whatsoever. To create a connection using the 'unauthenticated' method do:

```js
const horizon = Horizon({ authType: 'unauthenticated' });
```

This is the default authentication method and provides no means to separate user permissions or data in the Horizon application.

## Anonymous

The second auth type is anonymous. If anonymous authentication is enabled in the config, any user requesting anonymous authentication will be given a new JWT, with no other confirmation necessary. The server will create a user entry in the users table for this token, with no other way to authenticate as this user than by passing the token back. (This is done "under the hood," with the JWT stored in localStorage and passed back automatically on subsequent requests.)

```js
const horizon = Horizon({ authType: 'anonymous' });
```

In effect, this authentication type creates a "temporary user" for use with the current session. This allows user information to be saved while that session is active, but the user has no way of reauthenticating with the same account if the token is lost. (Note that the temporary user ID is stored in the Horizon database, and must be cleaned up manually.)

# Using OAuth {#oauth}

Your application will need a client ID and "secret" for each OAuth provider you want to connect with. The providers Horizon currently supports are:

* [Facebook](https://developers.facebook.com/apps/)
* [Github](https://github.com/settings/applications/new)
* [Google](https://console.developers.google.com/project)
* [Slack](https://api.slack.com/apps)
* [Twitch](https://www.twitch.tv/kraken/oauth2/clients/new)
* [Twitter](https://apps.twitter.com/app/new)
* [Auth0](https://auth0.com)

Each provider will let you register your application, and will give you the client data you need to give Horizon.

## Configuring the server

In order to use OAuth with Horizon, you'll need to configure a TLS certificate so you can serve assets with HTTPS, and either specify the `--key-file` and `--cert-file` options to `hz serve` or add them to the server's `.hz/config.toml` file. (See [The config.toml file][cf] for more details.) You can create a self-signed certificate with `hz create-cert`.

[cf]: /docs/configuration

You'll need to enter `id` and `secret` values in the `.hz/secrets.toml` file for your server (along with `host` and `redirect_url` values for Auth0). Toward the bottom of the automatically generated file, you'll see commented-out sample settings. Uncomment the appropriate lines and replace the dummy values with the ones you've received from the application provider. Adding Github OAuth configuration data would look like this:

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

**Note:** If your application embeds Horizon rather than using `hz serve`, you'll need to pass the OAuth endpoint to that object. Read "Configuring OAuth providers" in [Embedding Horizon][eh] for details.

[eh]: /docs/embed

## Configuring the client application

Use `authType: 'token'` when initializing Horizon in your client, and then use the `authEndpoint()` command to retrieve the endpoint for your OAuth identity provider and redirect the user to that URL.

```js
const horizon = Horizon({ authType: 'token' });
if (!horizon.hasAuthToken()) {
  horizon.authEndpoint('github').subscribe((endpoint) => {
    window.location.replace(endpoint);
  });
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

## Accessing session data

You can check whether a user is currently authenticated using the [Horizon.hasAuthToken][ha] method, and access their information with [Horizon.currentUser][cu]. For more information, read about [Users and groups][ug] and [Permissions and schema enforcement][perm].

[ha]:   /api/horizon/#hasauthtoken
[cu]:   /api/horizon/#currentuser
[ug]:   /docs/users
[perm]: /docs/permissions

## Notes about Horizon's OAuth support

* The authorization callback URL defaults to `https://<hostname>/horizon/<servicename>`. For Github, for instance, your callback would be `https://yourapp.com/horizon/github`. This can be changed with the `path` option to the `add_auth_provider` method. (See [Embedding Horizon][eh] for details on this option.)
* Currently, no metadata from OAuth providers&mdash;for example, friend/following lists&mdash; can be requested. In the near future, Horizon will support authentication scopes for selected providers to request access to these details, and returned metadata will be stored in the `Users` table.
* The redirection URL will be configurable in a future release.
* [Passport][pp] integration is not currently on Horizon's roadmap; this decision was made to avoid tightly coupling Horizon with [Express][ex]. (A [Github issue][gi] may be opened to discuss this in the future.)

[pp]: http://passportjs.org
[ex]: http://expressjs.com
[gi]: https://github.com/rethinkdb/horizon/issues/new
[eh]: /docs/embed
