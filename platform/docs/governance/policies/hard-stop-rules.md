---
description: >-
  The actions that always halt for explicit TCW confirmation before executing,
  and what to do when one appears mid-sequence.
icon: hand
---

# Hard-stop rules

A hard stop is an action that never executes on inference. It halts, is reported, and waits for TCW's explicit confirmation.

## The list

{% tabs %}
{% tab title="Legal & financial" %}
**Legal signatures**

* `pcg.contract.sign`
* Any `sign:*` tool
* PIV-9C document signing

**Financial**

* `pst.fiduciary.pay`
* `pst.token.mint`
* `pst.token.transfer`
* `pcg.invoice.issue` — first issuance

**Legal notices**

* `pcg.notice.unauthorized_use` — requires document binding
{% endtab %}

{% tab title="Identity & schema" %}
**Identity and auth changes**

* Auth0 role assignments
* Permission changes
* `app_metadata` writes via the Management API

**Schema changes**

* Any modification to the KV storage schema in `workers-oauth-provider`

**New entity onboarding**

* Issuing new Auth0 Client IDs
* Creating new entity scope manifests
{% endtab %}

{% tab title="Infrastructure" %}
**Infrastructure**

* Force-push to any branch
* `git reset --hard`
* Branch deletion
* CI/CD pipeline edits

**Zero Trust**

* Changes to device enrollment permissions
* MFA policy changes
* TLS inspection scope changes

**Token compromise response**

* Refresh token revocation
* Session invalidation for any principal other than the caller
{% endtab %}
{% endtabs %}

## When one appears mid-sequence

{% hint style="danger" %}
Halt the **entire** sequence — not just the blocking action. Report the blocking action and the reason, then wait for explicit instruction.
{% endhint %}

Continuing the non-blocked remainder is the tempting failure here. It is wrong for a specific reason: the steps after a hard stop were planned assuming it succeeded. Running them against a state where it did not leaves a partial result that nobody authorized and that the audit trail describes as complete.

The correct shape is to flag hard stops during the [planning step](https://app.gitbook.com/s/PQ2GmOf68qXiLqjUBtjb/architecture/handoff-chain), before execution begins — so confirmation is obtained up front rather than discovered halfway through.

## Absolute boundaries

These are not hard stops awaiting approval. They do not happen without explicit TCW instruction, at all:

* Storing secrets or PIV key material in plaintext
* Bypassing constructor validation in `workers-oauth-provider`
* Using the TypeScript `any` type without written justification
* Removing or downgrading any security control
* Enrolling a new principal with `entity_scope` pre-populated
* Executing a bond-related or legally binding action outside a document-binding-bound session
* Marking a deployment complete without verifying audit logs
* Pushing to `main` without passing `npm run check`
