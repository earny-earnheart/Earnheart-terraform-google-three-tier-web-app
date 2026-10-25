---
description: >-
  JSON-RPC 2.0 over the MCP server, and the five checks every call clears before
  it is sent.
icon: code
---

# Tool invocation

All MCP calls use JSON-RPC 2.0. Tool names follow `<scope>.<resource>.<action>` — for example `pcg.document.register` or `pst.fiduciary.pay`.

## Preflight checks

Confirm all five before invoking anything. Each maps to a specific denial code if it fails, so checking first turns a rejected call into a clear question.

{% stepper %}
{% step %}
## Scope covers the prefix

The calling principal's `entity_scope` must cover the tool's prefix. A `pst.*` tool needs PST in scope — PCG membership does not imply it.
{% endstep %}

{% step %}
## AAL meets the requirement

`pcg/aal` must meet the tool's `required_aal`. AAL2 is required for all `sign:*` tools and every tool marked `document_binding_required`.
{% endstep %}

{% step %}
## Role is sufficient

`pcg/roles` must include at least one of the tool's `required_role` values.
{% endstep %}

{% step %}
## Document binding matches

For `document_binding_required` tools, `pcg/document_binding` must match the target Document ID. A session bound to one document cannot act on another.
{% endstep %}

{% step %}
## Rate limit headroom exists

Check headroom before any bulk or looped call. Rate limits are enforced per scope.
{% endstep %}
{% endstepper %}

## After the call

Capture `audit_id` from the response metadata, then surface the outcome, `document_id`, and `entity_scope` to TCW.

{% hint style="danger" %}
The audit trail is mandatory. Every invocation produces an `audit_id`. If an audit record cannot be confirmed, treat the operation as **unverified** — regardless of what the response body said.
{% endhint %}

## On denial

| Response              | What to do                                                     |
| --------------------- | -------------------------------------------------------------- |
| `-32001` scope denied | Stop the sequence. Report the exact scope gap. Do not reroute. |
| `-32003` AAL too low  | Halt. AAL2 re-authentication is required.                      |

Neither is retried with elevated context. See [Error codes](error-codes.md) for the full set.
