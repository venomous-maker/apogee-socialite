# Changelog

Every release of apogee-socialite, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

Needs grav 1.12.0 or newer.

- `SocialitePackage`: the module connects itself to an apogee-framework app (apogee-package):
  its settings, wiring moved here from the framework.

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
