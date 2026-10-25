---
description: >-
  Four connectors, each live or mock depending on which secrets are set. No
  configuration flag decides it — the presence of credentials does.
icon: plug
---

# Connectors

The dashboard ships with four connectors. Each one implements the same three functions: `status()`, `fetchActivity()`, and `execute()`.

## Live vs. mock

There is no mode switch. A connector reports `live` only when **all** of its required secrets are present in the environment; otherwise it reports `mock`, returns sample activity, and makes no outbound call.

| Connector  | Required secrets                              | Live read                               |
| ---------- | --------------------------------------------- | --------------------------------------- |
| GitHub     | `GITHUB_TOKEN`, `GITHUB_OWNER`, `GITHUB_REPO` | 10 most recently updated issues and PRs |
| HubSpot    | `HUBSPOT_TOKEN`                               | —                                       |
| QuickBooks | `QUICKBOOKS_TOKEN`, `QUICKBOOKS_REALM_ID`     | —                                       |
| Stripe     | `STRIPE_SECRET_KEY`                           | 10 most recent charges                  |

GitHub needs all three secrets — a token without `GITHUB_OWNER` and `GITHUB_REPO` stays in mock mode. The same applies to QuickBooks, which needs both its token and its realm ID.

{% hint style="success" %}
This is what makes the dashboard demoable with zero credentials. Every connector falls back to sample data independently, so you can set up one connector at a time and the other three keep working.
{% endhint %}

## Reads

`GET /api/feed` calls every connector in parallel and normalizes the results into a common `FeedItem` shape:

```ts
interface FeedItem {
  id: string;
  source: 'github' | 'hubspot' | 'quickbooks' | 'stripe';
  type: string;
  title: string;
  detail?: string;
  url?: string;
  timestamp: string;
}
```

Items are sorted by `timestamp` descending and truncated to 30. A connector that throws is isolated — its error lands in the response's `errors` array and the other three still return.

## Writes

{% hint style="warning" %}
`execute()` is a stub in every connector. In mock mode it logs the action and returns without making a call. In live mode it **throws** — `No live handler implemented for <source> actionType "<actionType>"`.
{% endhint %}

Approving an action against a live connector today produces a `failed` approval, not a side effect. Before relying on the dashboard for real operations, implement the specific write call for each `actionType` you intend to support.

When you do, two constraints from the platform's governance apply directly:

* **Keep hard-stop actions out of auto-execution entirely.** See [hard-stop rules](https://app.gitbook.com/s/CLVU3zDPYq3NtGA5U56K/policies/hard-stop-rules).
* **Never log or return the secret.** The connector holds live credentials; `resultDetail` is stored in KV and returned over the API. See [security rules](https://app.gitbook.com/s/CLVU3zDPYq3NtGA5U56K/policies/security-rules).

## Adding a connector

A connector is a module exporting `status`, `fetchActivity`, and `execute`. Register it in `src/connectors/index.ts` and add its source to the `ConnectorSource` union in `src/types.ts` — the `SOURCES` array and the route validation both read from there.
