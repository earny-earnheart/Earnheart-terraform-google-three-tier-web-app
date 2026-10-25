---
description: >-
  Promotion checklists for Auth0, Cloudflare Zero Trust, and the MCP server.
  Each one validates somewhere safe before touching production.
icon: list-check
---

# Deployment rules

Every path to production runs through a lower environment or a smaller blast radius first. None of them ends at "it deployed" — each ends at a verification step.

## Auth0

{% stepper %}
{% step %}
## Validate against the spec

Check the change against `PCG-MCP-AUTH0-SPEC-001`.
{% endstep %}

{% step %}
## Apply to dev

Apply to `pantheon-platform-dev` first.
{% endstep %}

{% step %}
## Verify in logs

Confirm the expected behavior in the dev tenant's logs before going further.
{% endstep %}

{% step %}
## Obtain confirmation

Get TCW's confirmation before applying to production. Role assignments and permission changes are [hard stops](hard-stop-rules.md) in their own right.
{% endstep %}
{% endstepper %}

## Zero Trust

Apply to the **pilot ring first** and verify in Gateway logs before expanding.

Every policy is documented with:

| Field          | What it captures                             |
| -------------- | -------------------------------------------- |
| Intent         | What the policy is for                       |
| Owner          | Who is accountable for it                    |
| Risk           | What it mitigates, and what it introduces    |
| Selector logic | How the policy targets devices or identities |
| Affected group | Who it applies to                            |
| Rollback path  | How to undo it                               |

{% hint style="danger" %}
Device enrollment permissions, MFA policy, and TLS inspection scope are hard stops. A Zero Trust change that locks out enrollment can also lock out the path used to fix it — which is why the rollback path is documented before the change is applied, not after.
{% endhint %}

## MCP server

Before promotion to production, confirm all of the following:

* JWT middleware validates `iss`, `aud`, and enforces a JWKS TTL of 10 minutes
* All five entity manifest files are present
* Redis is reachable
* The Postgres `audit_log` is reachable
* Scope denial integration tests pass
* AAL denial integration tests pass

{% hint style="warning" %}
The two denial tests are the ones that matter most and the easiest to skip — they assert that the platform correctly says _no_. A server that passes every happy-path test and fails these is one that grants access it should refuse.
{% endhint %}

## Completion

{% hint style="danger" %}
Do not mark a deployment complete without verifying audit logs. A deploy that produced no audit records is unverified, not successful.
{% endhint %}

See [Error codes](https://app.gitbook.com/s/PQ2GmOf68qXiLqjUBtjb/reference/error-codes) for the denial responses the integration tests assert against.
