# cloudflare-dashboard

cloudflare gateway and workers

## PCG Operations Dashboard

A Cloudflare Worker that aggregates activity from PCG's connectors (GitHub, HubSpot, QuickBooks, Stripe) into a single live feed, and gates any write action behind a human-in-the-loop approval queue.

### How it works

* `GET /` — single-page dashboard (live activity feed + approvals inbox).
* `GET /api/feed` — aggregated read-only activity across all connectors.
* `GET /api/connectors` — per-connector status (`live` if credentials are configured, otherwise `mock`).
* `POST /api/actions` — propose a write action (`source`, `actionType`, `summary`, optional `payload`/`riskTier`). Creates a `pending` approval; nothing executes yet.
* `GET /api/approvals` — list proposed actions and their status.
* `POST /api/approvals/:id/approve` / `POST /api/approvals/:id/reject` — a human decides. Only `approve` triggers the connector's write call.

Every connector runs in **mock mode** until its secrets are set, so the dashboard is fully demoable without live credentials. No action ever executes without an explicit `approve` call — this is the human-in-the-loop boundary. Connector `execute()` handlers are stubs; wire up the specific write call per `actionType` before relying on this for real operations, and keep hard-stop actions (per `CLAUDE.md`) out of scope for auto-execution entirely.

### Local development

```sh
npm install
npm run dev      # wrangler dev, serves on http://localhost:8787
npm run check    # tsc --noEmit
```

### Deploying

1.  Create a KV namespace for the approvals queue and put its id in `wrangler.toml`:

    ```sh
    npx wrangler kv namespace create APPROVALS_KV
    npx wrangler kv namespace create APPROVALS_KV --preview
    ```
2.  (Optional) Set connector secrets to go live — any connector left unset stays in mock mode:

    ```sh
    npx wrangler secret put GITHUB_TOKEN
    npx wrangler secret put GITHUB_OWNER
    npx wrangler secret put GITHUB_REPO
    npx wrangler secret put HUBSPOT_TOKEN
    npx wrangler secret put QUICKBOOKS_TOKEN
    npx wrangler secret put QUICKBOOKS_REALM_ID
    npx wrangler secret put STRIPE_SECRET_KEY
    ```
3.  Deploy:

    ```sh
    npm run deploy
    ```
