# apogee-socialite

OAuth 2 and OpenID Connect sign-in for [Apogee](https://github.com/venomous-maker)
apps: GitHub, Google, Microsoft (accounts and Entra ID), GitLab (and
self-hosted GitLab), Bitbucket, Discord, Facebook, LinkedIn, Slack, and any
OpenID Connect issuer by discovery (Okta, Auth0, Keycloak, Authentik,
Zitadel…). Built on [apogee-http-client](https://github.com/venomous-maker/apogee-http-client)
and [apogee-session](https://github.com/venomous-maker/apogee-session).

```grav
import "std"
import "apogee-socialite"
use apogee.*

Socialite.configure("github", env("GITHUB_CLIENT_ID"), env("GITHUB_CLIENT_SECRET"), "https://app.test/auth/github/callback")
Socialite.configure("microsoft", id, secret, redirect, {"tenant": "organizations"})
Socialite.configure("okta", id, secret, redirect, {"issuer": "https://dev-1.okta.com/oauth2/default"})

router.middleware(["session"]).group(fn(r: Router) {
    r.get("/auth/{provider}", fn(req: Request) -> Response {
        return Socialite.driver(str(req.route("provider"))).redirect(req)
    })
    r.get("/auth/{provider}/callback", fn(req: Request) -> Response {
        let who = Socialite.driver(str(req.route("provider"))).user(req)
        // who.id, who.email, who.emailVerified, who.name, who.nickname, who.avatar,
        // who.token, who.refreshToken, who.expiresIn, who.scopes, who.raw
        ...find or create the user, sign them in...
    })
})
```

## In an apogee-framework app

Importing the module is all it takes: `SocialitePackage` (apogee-package) connects it when
the app is created. Each provider with a client id under `services.<name>` is configured; the redirect defaults to `APP_URL/auth/<name>/callback`. Providers: github, google, microsoft, gitlab, bitbucket, discord, facebook, linkedin, slack, oidc.

| Setting | Default |
|---|---|
| `services.<name>.client_id`, `.client_secret`, `.redirect` | `<NAME>_CLIENT_ID`, `<NAME>_CLIENT_SECRET`, `<NAME>_REDIRECT_URI` |
| `services.microsoft.tenant` | `MICROSOFT_TENANT`, `common` |
| `services.gitlab.host` | `GITLAB_HOST`, `https://gitlab.com` |
| `services.oidc.issuer` | `OIDC_ISSUER` |
| `services.<name>.scopes` | extra scopes, comma-separated |

Your own configuration wins over these defaults.

## What it checks

- **State.** `redirect` puts a random state in the URL and the session.
  `user` takes it back out (it works once) and compares the two in constant
  time. A mismatch throws `InvalidStateException`.
- **PKCE (S256)** for the providers that take it (Google, Microsoft, GitLab,
  Discord, OpenID Connect issuers): the verifier stays in the session and only
  its challenge goes to the provider. `withPkce()` turns it on for others.
- **Errors.** A provider's refusal (`?error=access_denied`), a bad code, or a
  failed call throws `OAuthException`, with the provider's `error` code.

## Settings, per request

```grav
Socialite.driver("github").setScopes(["read:user"]).redirect(req)
Socialite.driver("google").addScopes(["https://www.googleapis.com/auth/calendar.readonly"])
(Socialite.driver("google") as GoogleProvider).offline().domain("example.com")   // a refresh token; one Workspace domain
Socialite.driver("slack").with({"team": "T0123"})                                  // extra query parameters
Socialite.driver("github").stateless().user(req)                                   // no session (no state check, no PKCE)
```

Other calls: `tokens(code)` trades a code for tokens, `refresh(refreshToken)`
gets a fresh access token, and `userFromToken(token)` reads who a token
belongs to (from a mobile sign-in, say).

## Configuration

`Socialite.configure(name, clientId, clientSecret, redirect, extra)` takes
these extras:

| Key | For |
|---|---|
| `driver` | which provider, when the name is another (`"work"` → `"microsoft"`) |
| `tenant` | Microsoft: `common`, `organizations`, `consumers` or a tenant id |
| `host` | GitLab: a self-hosted instance |
| `issuer` | any other name: an OpenID Connect issuer, found by discovery |
| `scopes` | space-separated scopes, instead of the provider's defaults |

`Socialite.extend(name, fn(c: SocialiteConfig) -> OAuthProvider)` adds a
provider of your own. A provider extends `OAuthProvider` and gives
`authorizeUrl`, `tokenUrl`, `fetchUser(token)` and `mapUser(json)`. Every
built-in provider's endpoints are public fields, so a self-hosted instance
can stand in.

## Testing an app

```grav
Socialite.fake("github", Socialite.makeUser("9", "faye@app.test", "Faye"))
client.get("/auth/github")                    // redirects straight to the callback with ?code=fake
client.get("/auth/github/callback?code=fake") // user() is the fake user
Socialite.reset()
```

## Tests

```sh
grav test tests/socialite.grav
grav test tests/package.grav
```

They run the whole sign-in through the router and a session against a local
identity provider. It checks what a real one does: the client secret (in the
body or as Basic), single-use codes, the PKCE verifier and bearer tokens.

## License

MIT
