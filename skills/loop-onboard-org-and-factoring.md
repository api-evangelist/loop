---
name: Onboard an organization and set up a factoring relationship
description: Create (or upsert) an organization in the Loop network and establish a carrier-to-factor factoring relationship.
api: openapi/loop-openapi-original.json
operations: [Organizations_create, Organizations_get, FactoringRelationships_upsert, FactoringRelationships_get]
---

# Onboard an organization and set up factoring

Use the Loop API (`https://api.loop.com/v1`) to add a party to the network and link a carrier to its factor.

## Auth
`Authorization: Bearer <apiKey>`.

## Steps
1. **Create the organization** — `POST /v1/organizations` (`Organizations_create`). This call is idempotent on uniqueness-enforced tags: if a tag whose type has `enforceUniqueness=true` already resolves to an existing organization, that organization is returned with `200` instead of creating a duplicate. A `409` means the tags resolve to multiple existing organizations — reconcile before retrying.
2. **Confirm it** — `GET /v1/organizations/{qid}` (`Organizations_get`), or look it up by tag with `GET /v1/organizations/tagged/{tagType}/{tagTypeValue}` (`Organizations_getByTag`).
3. **Upsert the factoring relationship** — `POST /v1/factoring-relationships` (`FactoringRelationships_upsert`) with `carrierOrganizationQid` and `factorOrganizationQid`. A carrier can have only one effective factoring relationship at a time: upserting with a different factor terminates the current one and creates a new one; upserting the same factor merges into the existing relationship.
4. **Verify** — `GET /v1/factoring-relationships/{qid}` (`FactoringRelationships_get`) or list with `GET /v1/factoring-relationships` (`FactoringRelationships_list`).

## Rules
- Ids are QIDs (`qid::organization:<uuid>`, `qid::factoring_relationship:<uuid>`).
- Use the upsert semantics rather than pre-checking existence — that is the idempotency contract.
- To onboard an organization together with its payment detail in one call, use the separate **Loop Onboarding API** (`https://onboarding.api.loop.com/v1`, `Organizations_create`).
