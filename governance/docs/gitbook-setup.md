# GitBook setup — remaining steps

The GitBook site, spaces, and sections are built. What's left is Git Sync, which GitBook only exposes through its UI.

## Status

| Step                                                               | State                                |
| ------------------------------------------------------------------ | ------------------------------------ |
| Docs authored and structured (3 spaces, 14 pages)                  | Done                                 |
| GitBook MCP server connected                                       | Done                                 |
| Site created — **Pantheon-Platform Docs** (`site_ClQeb`, Ultimate) | Done (pre-existing)                  |
| Spaces created — Platform, Operations, Governance                  | Done                                 |
| Sections created and linked                                        | Done                                 |
| Cross-space link sentinels resolved                                | Done                                 |
| **Wire Git Sync per space**                                        | **Yours — UI only**                  |
| **Delete the 5 leftover template sections**                        | **Yours — UI only**                  |
| Site published                                                     | Done                                 |
| Branding                                                           | Not applied — needs a primary colour |

## What exists in GitBook

Organization **Pantheon-Platform** (`2smdwazqwLM4RyBf4nnz`), site **Pantheon-Platform Docs** (`site_ClQeb`).

| Section    | Path          | Space ID               | Repo directory    |
| ---------- | ------------- | ---------------------- | ----------------- |
| Platform   | `/platform`   | `PQ2GmOf68qXiLqjUBtjb` | `docs/platform`   |
| Operations | `/operations` | `GblCrmQ98Es67eiWmCBP` | `docs/operations` |
| Governance | `/governance` | `CLVU3zDPYq3NtGA5U56K` | `docs/governance` |

All three spaces are currently **empty**. They fill up when Git Sync runs.

## Step 1 — Wire Git Sync (UI only)

GitBook's API cannot configure Git Sync — repository selection, branch, project directory, and initial sync direction are all UI operations. About 30 seconds per space.

For each space below: open it → **Set up Git Sync** → **GitHub** → authorize GitBook if prompted → pick the repo → set branch and project directory → **Initialize**.

**Platform** — https://app.gitbook.com/o/2smdwazqwLM4RyBf4nnz/s/PQ2GmOf68qXiLqjUBtjb/

* Repository: `earny-earnheart/cloudflare-dashboard`
* Branch: `main`
* Project directory: `docs/platform`

**Operations** — https://app.gitbook.com/o/2smdwazqwLM4RyBf4nnz/s/GblCrmQ98Es67eiWmCBP/

* Repository: `earny-earnheart/cloudflare-dashboard`
* Branch: `main`
* Project directory: `docs/operations`

**Governance** — https://app.gitbook.com/o/2smdwazqwLM4RyBf4nnz/s/CLVU3zDPYq3NtGA5U56K/

* Repository: `earny-earnheart/cloudflare-dashboard`
* Branch: `main`
* Project directory: `docs/governance`

> **Initial sync direction must be GitHub → GitBook.** The repository is the source of truth and the GitBook spaces are empty. Choosing "GitBook → GitHub" would push three empty spaces over the docs in the repo, and the only way back is `git revert`.

If you wire this up before PR #2 is merged, use branch `claude/gitbook-setup-publish-j8bhkh` instead of `main`, then switch each space to `main` after merging.

## Step 2 — Delete the leftover template sections

The site was created at signup with GitBook's default template: **Home**, **Documentation**, **API Reference**, **Changelog**, **Help Center**. All five are untouched boilerplate, and GitBook's own guidance is that template content and Git Sync must not be mixed in one site.

Deletion is not available through the API, so remove them from the site's settings in the app: https://app.gitbook.com/o/2smdwazqwLM4RyBf4nnz/sites/site\_ClQeb

Deleted spaces are recoverable from the trash for 7 days.

## Step 3 — Branding

Not applied. Needs a primary colour (hex) at minimum; optionally logo URLs (light and dark), a favicon, and header/footer links.

## Note on the plan

The org is on `free_2024` with the trial ended and a downgrade scheduled for **2026-08-12**. Sections are an Ultimate-only feature. If the site drops to free, this three-section navigation degrades — decide on billing before that date.
