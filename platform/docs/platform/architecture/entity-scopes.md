---
description: >-
  Five entity namespaces on one MCP server, plus the shared tools available
  across all of them.
icon: diagram-project
---

# Entity scopes

A single MCP server hosts five entity namespaces. Which namespaces a principal can reach is determined entirely by the `entity_scope` claim on their JWT.

## The namespaces

| Prefix  | Entity                      | Routing                                   |
| ------- | --------------------------- | ----------------------------------------- |
| `pcg.*` | Pantheon Consulting Group   | Invoked directly                          |
| `pst.*` | ProofStream                 | Delegated to the ProofStream MCP gateway  |
| `eet.*` | Earnheart Express Trust     | Delegated to the EET gateway              |
| `cmf.*` | Earnheart Media             | Delegated to the Earnheart Media gateway  |
| `tro.*` | Fleet Management Operations | Delegated to the Fleet Management gateway |

## Shared tools

`shared.*` tools are available across every scope:

| Tool                        | Purpose                                                |
| --------------------------- | ------------------------------------------------------ |
| `shared.whoami`             | Resolve the calling principal and its effective claims |
| `shared.audit.search`       | Query the audit log                                    |
| `shared.document.resolve`   | Resolve a Document ID to its record                    |
| `shared.entity.list_scopes` | List the scopes available to the caller                |

These are the only sanctioned path for anything that spans entities.

{% hint style="danger" %}
Scope isolation is absolute. A tool call in one namespace cannot read or write another namespace's data. Reaching across a boundary requires `shared.*` tooling **and** a JWT carrying multi-scope claims — there is no other route, and none should be improvised.
{% endhint %}

## Principals

Two classes of principal use the platform.

{% columns %}
{% column %}
### TCW

Terrell Clarence Washington, President & Treasurer of PCG. Holds `root_admin` with full `entity_scope` across all five entities at AAL2 (YubiKey WebAuthn).

TCW is the authorizing human for every consequential decision. No hard-stop action executes without his explicit confirmation.
{% endcolumn %}

{% column %}
### Operational staff

VPs, secretaries, fleet dispatchers, and program staff. Each is assisted only within the permissions their Auth0 roles and entity scope already permit.

Their permissions are never escalated to complete a task — a scope gap is reported, not routed around.
{% endcolumn %}
{% endcolumns %}

## Denials

When `entity_scope` does not cover a tool's prefix, the server returns `-32001`.

{% hint style="warning" %}
On a scope denial, stop and report the exact scope gap. Do not retry with elevated context, and do not reroute the call through a different namespace to get the same effect — that defeats the boundary the denial exists to enforce.
{% endhint %}

See [Error codes](../reference/error-codes.md) for the full set of failure responses.
