# Changelog

Every release of apogee-socialite, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

## 0.2.2 (2026-11-28)

### Fixes

- A Redis connection shared by several threads (the cache, session and queue-dashboard stores, used by a multi-threaded server or queue worker) no longer mixes up replies or stalls: every command is one locked round trip (grav-redis 0.2.2). Uses the new releases of apogee-package, apogee-session.

## 0.2.1 (2026-11-27)

### Fixes

- Database connections that the server dropped while idle (MySQL's `wait_timeout`, a restart, a proxy's idle limit) reconnect on their next use instead of failing with "server has gone away" (grav-db 0.2.1). Uses the new releases of apogee-package.

## 0.2.0 (2026-11-25)

Needs grav 1.12.0 or newer.

- `SocialitePackage`: the module connects itself to an apogee-framework app (apogee-package):
  its settings, wiring moved here from the framework.

### Dependencies

- Uses apogee-package 0.1.0, apogee-http 0.3.0, apogee-session 0.2.2, apogee-http-client 0.2.0, apogee-routing 0.3.0, apogee-testing 0.1.1 (from their GitHub tags).

## 0.1.1 (2026-11-21)

- Laid out by `grav fmt` and tidied as grav 1.11's lints suggest (`isEmpty()`
  over `size() == 0`, compound assignments); no change in behaviour.

## 0.1.0 (2026-11-21)

- `OAuthProvider`: the OAuth 2 flow, with a state checked in constant time,
  PKCE (S256), refresh tokens, tokens the app already has, stateless use, and
  client auth in the body or as Basic.
- Providers: GitHub (with private emails), Google, Microsoft, GitLab,
  Bitbucket, Discord, Facebook (with appsecret_proof), LinkedIn, Slack, and any
  OpenID Connect issuer by discovery.
- `Socialite`: providers by name from configuration, `extend`, and `fake`
  for an app's tests.
