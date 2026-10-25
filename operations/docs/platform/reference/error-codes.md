---
description: >-
  JSON-RPC error codes returned by the MCP server, and the required response to
  each.
icon: triangle-exclamation
---

# Error codes

Each code has one correct response. None of them is "retry with a different token."

| Code     | Meaning                  | Required response                                                    |
| -------- | ------------------------ | -------------------------------------------------------------------- |
| `-32000` | JWT invalid              | Stop and report. Do **not** retry with a different token.            |
| `-32001` | Scope denied             | Stop the sequence. Report the exact scope gap. Do not reroute.       |
| `-32002` | Role insufficient        | Report, and ask TCW to verify the role assignment in Auth0.          |
| `-32003` | AAL too low              | Halt. Inform TCW that AAL2 re-authentication is required.            |
| `-32004` | Rate limit exceeded      | Wait for `Retry-After`. Report if it persists.                       |
| `-32005` | Document binding missing | Halt. Re-authentication bound to a specific Document ID is required. |
| `-32030` | Upstream error           | Capture the detail, retry **once**, escalate if it persists.         |

{% hint style="warning" %}
`-32030` is the only code with a sanctioned retry, and it gets exactly one. Everything else is a decision that belongs to a human — retrying an authorization failure just produces a second audit record of the same denial.
{% endhint %}

## Why retries are restricted

`-32000` through `-32005` are all authorization outcomes: the platform evaluated the request against identity, scope, role, assurance level, or document binding, and declined. Re-issuing the same call cannot change that evaluation. What changes it is a role assignment in Auth0, a step-up authentication, or a document-bound session — all of which require [TCW](../architecture/entity-scopes.md#principals).

See [Tool invocation](tool-invocation.md) for the preflight checks that prevent most of these codes from ever being returned.
