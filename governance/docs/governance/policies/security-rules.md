---
description: Nine constraints that hold across every layer, with no exception path.
icon: lock
---

# Security rules

These are not defaults to be overridden under time pressure. Each one exists because the alternative has a known failure mode.

{% stepper %}
{% step %}
## Never store, log, or transmit secrets

Access tokens, refresh tokens, client secrets, YubiKey PINs, and PIV key material never appear in responses, files, comments, or handoff records.
{% endstep %}

{% step %}
## Validate `aud` strictly

A JWT issued for `https://mcp.pantheon-platform.us/api` is not valid for any other resource server — even inside the same Auth0 tenant. Accepting a token issued for a different audience turns any service that can obtain one into a caller here.
{% endstep %}

{% step %}
## Reject dev-tenant tokens in production

Production trusts only `iss = https://pantheon-platform.us.auth0.com/`. Staging tokens are rejected.
{% endstep %}

{% step %}
## No HS256

The MCP server uses RS256. If any integration proposes symmetric JWT signing, refuse and escalate — a shared secret makes every verifier a signer.
{% endstep %}

{% step %}
## Props stay encrypted

The `props` object in `workers-oauth-provider` is AES-GCM encrypted. Never reconstruct or expose decrypted props outside an authorized handler.
{% endstep %}

{% step %}
## Scope isolation is absolute

A tool call in one entity namespace cannot read or write another's data. Cross-scope access requires `shared.*` tooling **and** multi-scope JWT claims.
{% endstep %}

{% step %}
## The audit trail is mandatory

Every invocation produces an `audit_id`. If an audit record cannot be confirmed, treat the operation as **unverified**.
{% endstep %}

{% step %}
## Do not bypass PKCE

Authorization code + PKCE only. Never propose the implicit flow as a workaround.
{% endstep %}

{% step %}
## Zero Trust is a prerequisite

Do not invoke MCP tools from any device that is not WARP-connected with identity-bound enrollment.
{% endstep %}
{% endstepper %}

## Code contribution rules

For `workers-oauth-provider` specifically:

| Rule         | Detail                                                                                                                     |
| ------------ | -------------------------------------------------------------------------------------------------------------------------- |
| Single file  | All implementation lives in `src/oauth-provider.ts`. Do not split it.                                                      |
| Checks       | Run `npm run check` before declaring any code task complete.                                                               |
| Tests        | Add tests for every new behavior using `MockKV`, `MockExecutionContext`, and `createMockRequest()`.                        |
| Versioning   | Any change that alters KV storage format is a **minor** version bump, not a patch.                                         |
| Dependencies | Do not add runtime dependencies without TCW confirmation.                                                                  |
| Disclosure   | State security implications explicitly in any PR touching token storage, PKCE, redirect URI validation, or scope handling. |

MCP specification compliance governs MCP-related OAuth behavior: [modelcontextprotocol.io/specification/2025-11-25/basic/authorization](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization).

{% hint style="warning" %}
A KV storage format change is a minor bump because deployed instances read records written by the previous version. Shipping it as a patch invites an automatic upgrade into a format the running code cannot parse.
{% endhint %}
