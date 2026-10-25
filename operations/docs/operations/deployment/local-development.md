---
description: Run the worker locally with wrangler, with every connector in mock mode.
icon: laptop-code
---

# Local development

No credentials are needed to run the dashboard locally. Every connector starts in mock mode and serves sample activity.

```sh
npm install
npm run dev      # wrangler dev, serves on http://localhost:8787
npm run check    # tsc --noEmit
```

Open `http://localhost:8787` for the dashboard, or hit the API directly:

```sh
curl http://localhost:8787/api/health
curl http://localhost:8787/api/connectors
curl http://localhost:8787/api/feed
```

{% hint style="danger" %}
`npm run check` must pass before pushing to `main`. It is `tsc --noEmit` — it catches the type errors that a Worker would otherwise surface at request time in production.
{% endhint %}

## Trying the approval flow

Propose an action, list it, then decide it — all against mock connectors, so nothing leaves your machine.

```sh
# Propose
curl -sX POST http://localhost:8787/api/actions \
  -H 'Content-Type: application/json' \
  -d '{"source":"github","actionType":"issue.comment","summary":"Test proposal"}'

# List
curl -s http://localhost:8787/api/approvals

# Approve (substitute the id from the propose response)
curl -sX POST http://localhost:8787/api/approvals/<id>/approve \
  -H 'Content-Type: application/json' \
  -d '{"decidedBy":"TCW"}'
```

In mock mode the approval settles as `executed` with a `resultDetail` noting that no API call was made.

## Local KV

`wrangler dev` provides local KV storage for the `APPROVALS_KV` binding, so approvals persist across requests within a session without a real namespace. The placeholder IDs in `wrangler.toml` are only needed for a real deploy — see [Deploying](deploying.md).
