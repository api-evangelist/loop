---
name: Ingest and audit a payable invoice with Loop
description: Upload a payable-invoice record to Loop, wait for ingestion to complete, then read the audited payable invoice and its review.
api: openapi/loop-openapi-original.json
operations: [CreatePayableInvoiceRecordArtifact_create, ArtifactIngestionState_get, PayableInvoices_list, PayableInvoices_get, PayableInvoiceReview_get]
---

# Ingest and audit a payable invoice

Use the Loop API (`https://api.loop.com/v1`) to submit a carrier/vendor invoice and retrieve Loop's audit result.

## Auth
Send `Authorization: Bearer <apiKey>` on every request (key prefix `lk_live_`, issued by your Loop contact). Confirm the key with `GET /v1/ping` (expect HTTP 200).

## Steps
1. **Create the payable-invoice artifact** — `POST /v1/artifacts/payable-invoice-record` (`CreatePayableInvoiceRecordArtifact_create`). Reference the involved organizations by QID (`payorOrganizationQid`, `payeeOrganizationQid`, `vendorOrganizationQid`). Loop de-duplicates by md5 hash, so re-submitting the same document returns the existing artifact rather than a duplicate. The response returns the artifact `qid`.
2. **Poll ingestion state** — `GET /v1/artifact-ingestion-state/{qid}` (`ArtifactIngestionState_get`) until it reports complete. A `404` means the artifact QID is not found.
3. **Find the resulting payable invoice** — `GET /v1/payable-invoices` (`PayableInvoices_list`) using cursor pagination (`first`, `after`; read `pageInfo.endCursor`), or `GET /v1/payable-invoices/{qid}` (`PayableInvoices_get`) once you know the invoice QID.
4. **Read the audit review** — `GET /v1/payable-invoices/{qid}/review` (`PayableInvoiceReview_get`) to get the audit outcome (accepted charges, adjustments, exceptions).

## Rules
- Identifiers are QIDs: `qid::<entity-type>:<uuid>` (e.g. `qid::payable_invoice:<uuid>`), optionally pinned to a revision with `@<n>`.
- Errors return an HTTP status + description (not RFC 9457). Handle `400` (validation), `404` (not found), `429` (rate-limited bulk uploads — back off and retry).
- Subscribe to Loop webhooks (Svix) instead of tight polling where possible; verify `svix-signature`.
