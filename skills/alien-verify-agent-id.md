---
name: Verify an Alien Agent ID
description: Verify that an AI agent's Ed25519 Agent ID token is valid and linked to a verified human, before trusting an agent-initiated action.
api: https://sso.alien-api.com
operations:
- verify_agent_id_token (@alien-id/sso-agent-id)
- GET /oauth/jwks
---

# Verify an Alien Agent ID

Alien Agent ID lets an AI agent perform real actions that are cryptographically
signed and linked to a verified human (the agent's owner). Use this skill in a
backend service to verify an inbound agent token before acting on it.

## Prerequisites
- `npm install @alien-id/sso-agent-id`
- The agent presents an Agent ID token (Ed25519-signed).

## Steps
1. Extract the Agent ID token from the request (e.g. an `Authorization` header or
   the `agent-id-auth header` CLI output).
2. **Verify** the token with `@alien-id/sso-agent-id` — it checks the Ed25519
   signature and the token's binding to the owner's Alien ID.
3. Confirm the linked human subject (`sub`) is the expected owner and the token is
   unexpired.
4. Authorize the specific action per your own policy — an Agent ID proves *who is
   accountable*, not blanket permission.

## Owner / agent lifecycle (CLI)
The `alien-agent-id` CLI (`cli/alien-cli.yml`) bootstraps the other side:
`agent-id-core bootstrap` → `bind` (to owner Alien ID) → `setup-owner-session` →
`sign` / `export-proof`; `agent-id-git commit` signs commits.

## Conventions & errors
- Agent tokens are EdDSA (Ed25519); SSO ID tokens are RS256 (JWKS `GET /oauth/jwks`).
- See `authentication/alien-authentication.yml` and `conventions/alien-conventions.yml`.
