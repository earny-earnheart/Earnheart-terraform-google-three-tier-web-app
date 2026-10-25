---
description: >-
  How Pantheon Consulting Group's administrative platform is put together — the
  four enforced infrastructure layers, the five entity namespaces, and the rules
  that govern every tool call.
icon: sitemap
---

# Platform

Pantheon Consulting Group (PCG) operates a single administrative platform shared by five entities. Every request that crosses it passes through four enforced layers — network, identity, authorization, and tooling — and is denied unless each layer explicitly permits it.

This space covers how those layers fit together and how work is routed between entities. For the operator-facing dashboard, see [Operations](https://app.gitbook.com/o/2smdwazqwLM4RyBf4nnz/s/GblCrmQ98Es67eiWmCBP/). For the policies that gate consequential actions, see [Governance](https://app.gitbook.com/o/2smdwazqwLM4RyBf4nnz/s/CLVU3zDPYq3NtGA5U56K/).

## Start here

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><i class="fa-layer-group">:layer-group:</i></td><td><strong>The four layers</strong></td><td>Network, identity, OAuth gateway, and MCP server — and what each one enforces.</td><td><a href="architecture/four-layers.md">four-layers.md</a></td></tr><tr><td><i class="fa-diagram-project">:diagram-project:</i></td><td><strong>Entity scopes</strong></td><td>The five namespaces, their tool prefixes, and what <code>shared.*</code> covers.</td><td><a href="architecture/entity-scopes.md">entity-scopes.md</a></td></tr><tr><td><i class="fa-right-left">:right-left:</i></td><td><strong>Handoff chain</strong></td><td>How a task decomposes across entities and returns for approval.</td><td><a href="architecture/handoff-chain.md">handoff-chain.md</a></td></tr><tr><td><i class="fa-code">:code:</i></td><td><strong>Tool invocation</strong></td><td>The five preflight checks every JSON-RPC call must pass.</td><td><a href="reference/tool-invocation.md">tool-invocation.md</a></td></tr></tbody></table>

## The entities

| Code | Entity                             | Tool prefix |
| ---- | ---------------------------------- | ----------- |
| PCG  | Pantheon Consulting Group (parent) | `pcg.*`     |
| PST  | ProofStream                        | `pst.*`     |
| EET  | Earnheart Express Trust            | `eet.*`     |
| CMF  | Earnheart Media                    | `cmf.*`     |
| TRO  | Fleet Management Operations        | `tro.*`     |

PCG is the parent entity. Tasks originate at PCG scope, decompose into entity-specific work, and return to PCG for final approval.

{% hint style="info" %}
Scope isolation is absolute. A tool call in one entity namespace cannot read or write data in another. Cross-scope access requires `shared.*` tooling **and** multi-scope claims on the caller's JWT.
{% endhint %}
