---
name: Sign in with Alien ID
description: Authenticate a user as a verified unique human via Alien SSO (OAuth 2.0 + OIDC, PKCE, deep-link + poll) and read their subject claims.
api: https://sso.alien-api.com
operations:
- GET /oauth/authorize
- POST /oauth/poll
- POST /oauth/token
- GET /oauth/userinfo
- GET /oauth/jwks
---

# Sign in with Alien ID

Add "Sign in with Alien ID" so a user proves they are a unique human without
sharing name, email, or documents. Alien SSO is a standard OpenID Connect /
OAuth 2.0 provider — you can use `@alien-id/sso` (or any OIDC library) instead of
hand-rolling these calls.

## Prerequisites
- An Alien Provider (your app identity) configured in the developer dashboard: https://dev.alien.org/dashboard
- Base URL: `https://sso.alien-api.com` (public client — `token_endpoint_auth_methods: none`, PKCE required)

## Steps
1. Generate a PKCE `code_verifier` and `code_challenge` (method `S256`), plus `state` and `nonce`.
2. **Start authorization** — `GET /oauth/authorize` with `response_type=code`,
   `response_mode=json`, `scope=openid`, `code_challenge`, `code_challenge_method=S256`,
   `state`, `nonce`. Present the returned deep link to the user (opens the Alien app).
3. **Poll** — `POST /oauth/poll` with the code until the user approves. A `404`
   means the code is unknown/expired.
4. **Exchange** — `POST /oauth/token` (`grant_type=authorization_code`, `code`,
   `code_verifier`). A `409 invalid_grant` means the code was already redeemed
   (single-use). Optionally bind the token with DPoP (RFC 9449, EdDSA).
5. **Read the user** — `GET /oauth/userinfo` with `Authorization: Bearer <access_token>`.
   The `sub` is the stable, privacy-preserving Alien ID subject. A `401` means the
   access token expired — refresh with `grant_type=refresh_token`.
6. **Verify ID tokens** server-side against JWKS at `GET /oauth/jwks` (RS256).

## Conventions & errors
- Discovery: `https://sso.alien-api.com/.well-known/openid-configuration`
- Errors are OAuth 2.0 `{error, error_description}` (see `errors/alien-problem-types.yml`).
- Only the `openid` scope is offered — Alien asserts personhood, not personal data.
