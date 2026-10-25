---
description: Every endpoint the PCG Operations Dashboard worker exposes.
icon: brackets-curly
---

# API reference

All responses are JSON except `GET /`, which serves the dashboard HTML. CORS is open (`Access-Control-Allow-Origin: *`) and `OPTIONS` is handled on every route.

| Method | Path                         | Purpose                                               |
| ------ | ---------------------------- | ----------------------------------------------------- |
| `GET`  | `/`                          | Single-page dashboard — live feed and approvals inbox |
| `GET`  | `/api/health`                | Liveness check                                        |
| `GET`  | `/api/connectors`            | Per-connector status                                  |
| `GET`  | `/api/feed`                  | Aggregated read-only activity                         |
| `GET`  | `/api/approvals`             | List proposed actions and their status                |
| `POST` | `/api/actions`               | Propose a write action                                |
| `POST` | `/api/approvals/:id/approve` | Approve — **this executes**                           |
| `POST` | `/api/approvals/:id/reject`  | Reject — nothing executes                             |

## GET /api/health

```json
{ "ok": true, "time": "2026-07-29T05:57:58.000Z" }
```

## GET /api/connectors

Returns one status per connector. `mode` is `live`, `mock`, or `error`.

```json
{
  "connectors": [
    { "source": "github", "mode": "mock", "detail": "Set GITHUB_TOKEN, GITHUB_OWNER, GITHUB_REPO to go live" },
    { "source": "hubspot", "mode": "live" }
  ]
}
```

## GET /api/feed

Fetches all four connectors in parallel, merges the results, sorts newest-first, and returns at most **30** items. A connector that throws does not fail the request — its error is collected in `errors` and the remaining connectors still return.

```json
{
  "items": [
    {
      "id": "github-2381",
      "source": "github",
      "type": "pull_request",
      "title": "#42 Add approval queue",
      "detail": "open",
      "url": "https://github.com/…",
      "timestamp": "2026-07-29T05:00:00.000Z"
    }
  ],
  "errors": [{ "source": "stripe", "error": "Error: Stripe API error 401" }]
}
```

{% hint style="info" %}
A non-empty `errors` array with a `200` status is the normal shape for a partial failure. Check it — an empty feed with one entry in `errors` is a broken connector, not a quiet period.
{% endhint %}

## POST /api/actions

Creates a `pending` approval. Nothing executes.

**Body**

| Field        | Type                                              | Required | Notes                                  |
| ------------ | ------------------------------------------------- | -------- | -------------------------------------- |
| `source`     | `github` \| `hubspot` \| `quickbooks` \| `stripe` | yes      | Must be one of the four                |
| `actionType` | string                                            | yes      | Non-empty; dispatched by the connector |
| `summary`    | string                                            | yes      | Non-empty; what a reviewer reads       |
| `payload`    | object                                            | no       | Defaults to `{}`                       |
| `riskTier`   | `standard` \| `hard_stop`                         | no       | Defaults to `standard`                 |

{% tabs %}
{% tab title="Request" %}
```sh
curl -X POST https://<worker-host>/api/actions \
  -H 'Content-Type: application/json' \
  -d '{
    "source": "github",
    "actionType": "issue.comment",
    "summary": "Reply to #42 confirming the KV namespace id",
    "payload": { "issue": 42, "body": "Namespace created." }
  }'
```
{% endtab %}

{% tab title="201 Created" %}
```json
{
  "approval": {
    "id": "6f1c…",
    "source": "github",
    "actionType": "issue.comment",
    "summary": "Reply to #42 confirming the KV namespace id",
    "payload": { "issue": 42, "body": "Namespace created." },
    "riskTier": "standard",
    "status": "pending",
    "createdAt": "2026-07-29T05:57:58.000Z"
  }
}
```
{% endtab %}

{% tab title="400 Bad Request" %}
```json
{ "error": "source, actionType, and summary are required" }
```

Returned when `source` is not one of the four valid values, when `actionType` or `summary` is missing or empty, or when `riskTier` is present but not `standard` or `hard_stop`.
{% endtab %}
{% endtabs %}

## POST /api/approvals/:id/approve

{% hint style="danger" %}
This is the execution boundary. Approving dispatches to the connector's `execute()` immediately — there is no second confirmation.
{% endhint %}

**Body** — optional. `{ "decidedBy": "TCW" }`. Omitted or non-string values record `decidedBy: "unknown"`.

```sh
curl -X POST https://<worker-host>/api/approvals/6f1c…/approve \
  -H 'Content-Type: application/json' \
  -d '{"decidedBy": "TCW"}'
```

Returns the updated approval — `executed` with `resultDetail`, or `failed` with the error text. Execution failures still return `200`: the decision succeeded even when the downstream call did not.

Returns `404` (`Approval not found or already decided`) if the ID is unknown or the approval is no longer `pending`.

## POST /api/approvals/:id/reject

Identical shape, terminal `rejected` status. No connector is contacted.

## Errors

| Status | Body                                                          | Cause                                    |
| ------ | ------------------------------------------------------------- | ---------------------------------------- |
| `400`  | `{ "error": "source, actionType, and summary are required" }` | Failed validation on `POST /api/actions` |
| `404`  | `{ "error": "Approval not found or already decided" }`        | Unknown ID, or already decided           |
| `404`  | `{ "error": "Not found" }`                                    | No route matched                         |
| `500`  | `{ "error": "<message>" }`                                    | Unhandled error                          |
