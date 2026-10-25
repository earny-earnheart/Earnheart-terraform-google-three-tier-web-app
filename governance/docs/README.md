# Pantheon Platform Docs — source

Source content for the Pantheon Consulting Group documentation site.

Each top-level folder in this directory is a separate GitBook space, wired to GitBook via Git Sync. Edits flow in both directions once sync is configured — commits here appear in GitBook, and edits made in the GitBook editor come back as commits.

| Directory     | Space      | Git Sync project directory |
| ------------- | ---------- | -------------------------- |
| `platform/`   | Platform   | `docs/platform`            |
| `operations/` | Operations | `docs/operations`          |
| `governance/` | Governance | `docs/governance`          |

Each space directory holds its own `README.md` (the space homepage) and `SUMMARY.md` (the table of contents). Do not rename these — GitBook resolves both by convention.

See [`GITBOOK-SETUP.md`](gitbook-setup.md) for the remaining one-time setup steps, and `cross-space-links.yaml` for the cross-space link sentinels that resolve after the spaces are created.

Files at this level — this README, `GITBOOK-SETUP.md`, `cross-space-links.yaml` — sit outside every space directory and are never synced to GitBook.
