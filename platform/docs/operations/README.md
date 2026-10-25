---
description: >-
  The PCG Operations Dashboard — one live feed across every connector, and an
  approval queue in front of every write.
icon: gauge-high
---

# Operations

The PCG Operations Dashboard is a Cloudflare Worker that aggregates activity from PCG's connectors — GitHub, HubSpot, QuickBooks, and Stripe — into a single live feed, and gates every write action behind a human-in-the-loop approval queue.

Reads are direct. Writes are proposals: nothing executes until a human approves it.

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><i class="fa-arrows-split-up-and-left">:arrows-split-up-and-left:</i></td><td><strong>How it works</strong></td><td>The approval lifecycle from proposal to execution.</td><td><a href="dashboard/how-it-works.md">how-it-works.md</a></td></tr><tr><td><i class="fa-brackets-curly">:brackets-curly:</i></td><td><strong>API reference</strong></td><td>Every endpoint the worker exposes.</td><td><a href="dashboard/api-reference.md">api-reference.md</a></td></tr><tr><td><i class="fa-plug">:plug:</i></td><td><strong>Connectors</strong></td><td>The four connectors, and how live and mock mode are decided.</td><td><a href="dashboard/connectors.md">connectors.md</a></td></tr><tr><td><i class="fa-rocket">:rocket:</i></td><td><strong>Deploying</strong></td><td>KV namespace, connector secrets, and the deploy step.</td><td><a href="deployment/deploying.md">deploying.md</a></td></tr></tbody></table>

## The human-in-the-loop boundary

Every connector runs in **mock mode** until its secrets are set, so the dashboard is fully demoable without live credentials. No action ever executes without an explicit `approve` call — that is the boundary the whole design exists to hold.

{% hint style="warning" %}
Connector `execute()` handlers are stubs. Wire up the specific write call per `actionType` before relying on this for real operations, and keep [hard-stop actions](https://app.gitbook.com/s/CLVU3zDPYq3NtGA5U56K/policies/hard-stop-rules) out of scope for auto-execution entirely.
{% endhint %}
