---
description: Create the KV namespace, set the connector secrets you want live, and deploy.
icon: rocket
---

# Deploying

{% stepper %}
{% step %}
## Create the KV namespace

The approvals queue needs a real KV namespace. Create both the production and preview namespaces, then put their IDs into `wrangler.toml`:

```sh
npx wrangler kv namespace create APPROVALS_KV
npx wrangler kv namespace create APPROVALS_KV --preview
```

`wrangler.toml` ships with placeholders — replace both:

```toml
[[kv_namespaces]]
binding = "APPROVALS_KV"
id = "REPLACE_WITH_KV_NAMESPACE_ID"
preview_id = "REPLACE_WITH_PREVIEW_KV_NAMESPACE_ID"
```
{% endstep %}

{% step %}
## Set connector secrets (optional)

Any connector left unset stays in mock mode. Set only the ones you want live:

```sh
npx wrangler secret put GITHUB_TOKEN
npx wrangler secret put GITHUB_OWNER
npx wrangler secret put GITHUB_REPO
npx wrangler secret put HUBSPOT_TOKEN
npx wrangler secret put QUICKBOOKS_TOKEN
npx wrangler secret put QUICKBOOKS_REALM_ID
npx wrangler secret put STRIPE_SECRET_KEY
```

Secrets are set with `wrangler secret put`, never written into `wrangler.toml`.
{% endstep %}

{% step %}
## Check, then deploy

```sh
npm run check
npm run deploy
```
{% endstep %}

{% step %}
## Verify

```sh
curl https://<worker-host>/api/health
curl https://<worker-host>/api/connectors
```

`/api/connectors` confirms which connectors came up `live` — this is the fastest way to catch a secret that did not take.
{% endstep %}
{% endstepper %}

{% hint style="danger" %}
Do not put credentials in `wrangler.toml`. It is committed to the repository; secrets set via `wrangler secret put` are not.
{% endhint %}

## Before going live

Going live changes what an approval does. In mock mode, approving is inert. With secrets set, approving dispatches a real call.

* `execute()` handlers are stubs — see [Connectors](../dashboard/connectors.md). Implement the specific write per `actionType` before approving anything against a live connector.
* Keep [hard-stop actions](https://app.gitbook.com/s/CLVU3zDPYq3NtGA5U56K/policies/hard-stop-rules) out of auto-execution entirely.
* Confirm the audit trail before calling a deployment complete, per the [deployment rules](https://app.gitbook.com/s/CLVU3zDPYq3NtGA5U56K/policies/deployment-rules).

## Configuration reference

| Setting              | Where                 | Notes                              |
| -------------------- | --------------------- | ---------------------------------- |
| `name`               | `wrangler.toml`       | `pcg-dashboard`                    |
| `main`               | `wrangler.toml`       | `src/index.ts`                     |
| `compatibility_date` | `wrangler.toml`       | `2026-06-01`                       |
| `APPROVALS_KV`       | `wrangler.toml`       | KV binding for the approvals queue |
| Connector secrets    | `wrangler secret put` | Absent secrets mean mock mode      |
