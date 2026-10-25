---
description: The identifier convention every PCG record artifact follows.
icon: id-card
---

# Document IDs

All PCG record artifacts use a single identifier format:

```
[ENTITY]-[CATEGORY]-[YEAR]-[SEQUENCE]-[VERSION]
```

| Segment    | Meaning                                         | Example          |
| ---------- | ----------------------------------------------- | ---------------- |
| `ENTITY`   | Entity code — `PCG`, `PST`, `EET`, `CMF`, `TRO` | `PCG`            |
| `CATEGORY` | Artifact category                               | `MCP-AUTH0-SPEC` |
| `YEAR`     | Four-digit year of issuance                     | `2026`           |
| `SEQUENCE` | Zero-padded sequence within category and year   | `001`            |
| `VERSION`  | Version of the artifact                         | `v1.0`           |

Two real examples: `PCG-MCP-AUTH0-SPEC-001` (the Auth0 specification that Auth0 changes are validated against) and `PCG-TCW-AI-2026-001-v1.0` (the governing instrument for this platform).

## Registering an ID

Register every new Document ID via `pcg.document.register` **before** issuing the artifact it names.

{% hint style="danger" %}
Never reuse a Document ID. Document binding ties an authenticated session to a specific ID — a reused identifier means a session bound to one artifact can act on a different one, which is exactly what the binding exists to prevent.
{% endhint %}

## Where Document IDs are used

* **Handoff records** carry `context_document_id`, linking delegated work back to its originating record. See [Handoff chain](../architecture/handoff-chain.md).
* **Document-bound tools** check `pcg/document_binding` against the target ID as one of the [preflight checks](tool-invocation.md).
* **`shared.document.resolve`** resolves an ID to its record from any entity scope.

Governing law: Wyoming.
