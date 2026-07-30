---
name: Resolve Loop business exceptions on an entity
description: List the business exceptions raised against a target entity, inspect each one, and resolve them.
api: openapi/loop-openapi-original.json
operations: [BusinessExceptions_listForTargetEntityQid, BusinessExceptions_get, BusinessExceptions_resolve]
---

# Resolve business exceptions

Loop raises **business exceptions** against entities (e.g. a payable invoice) that need human/agent attention. Use the Loop API (`https://api.loop.com/v1`) to clear them.

## Auth
`Authorization: Bearer <apiKey>`.

## Steps
1. **List exceptions for the entity** — `GET /v1/business-exceptions/target-entity-qid/{targetEntityQid}` (`BusinessExceptions_listForTargetEntityQid`). Pass the target entity's QID (e.g. `qid::payable_invoice:<uuid>`); a `400` means the QID is invalid.
2. **Inspect an exception** — `GET /v1/business-exceptions/{qid}` (`BusinessExceptions_get`) to read its type and details.
3. **Resolve it** — `PUT /v1/business-exceptions/{qid}/resolve` (`BusinessExceptions_resolve`) with the resolution input. If the exception is already resolved you get `422`; treat that as already-done, not an error.

## Rules
- All ids are QIDs (`qid::business_exception:<uuid>`, `qid::payable_invoice:<uuid>`).
- Resolving is idempotent-ish: a second resolve returns `422 already resolved`.
- Loop can also notify you of new exceptions via webhooks (Svix) — verify the `svix-signature` header before acting.
