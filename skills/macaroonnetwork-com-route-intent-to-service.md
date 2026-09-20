---
generated: '2026-09-19'
method: generated
name: Route a natural-language intent to the right service and read its evidence
description: Use the semantic router and the public catalogue to pick a service, then read its validation evidence, limitations, category and upstream data rights before buying anything.
api: openapi/macaroonnetwork-com-openapi.json
operations: [router_resolve_api_router_resolve_post, public_service_detail_api_public_services__service_id__get, public_service_evidence_api_public_services__service_id__evidence_get, public_catalog_taxonomy_api_public_taxonomy_get, public_sources_api_public_sources_get]
source: Grounded in openapi/macaroonnetwork-com-openapi.json (operationIds verified) and the live /api/public/* responses of 2026-09-19.
---

# Route an intent to a service and read its evidence

All operations here are free and anonymous (`https://api.macaroonnetwork.com`). Nothing in this skill spends money.

## Steps
1. **Resolve the intent** — `router_resolve_api_router_resolve_post` (`POST /api/router/resolve`) with `{"intent": "<what you need>", "constraints": {...optional...}}`. The `RouterResolveResponse` carries `request_id`, `selected_service`, `alternatives[]` and `selection_evidence` — keep `request_id` for tracing.
2. **Read the service** — `public_service_detail_api_public_services__service_id__get` (`GET /api/public/services/{service_id}`): `price` (USDC amount, network base, rail x402), `status`, `trust_status`, `input_schema`, `output_schema`, `example_request`, `source_ids`, `category_id`, `mcp_server_id`.
3. **Read the evidence** — `public_service_evidence_api_public_services__service_id__evidence_get` (`GET /api/public/services/{service_id}/evidence`): validation claims and status, limitations, licence gate. Do not treat a `candidate_unverified` or `metadata_only` claim as more than it says.
4. **Place it in the taxonomy** — `public_catalog_taxonomy_api_public_taxonomy_get` (`GET /api/public/taxonomy`) returns the 17-category ontology (`macaroon.catalog-ontology.v1`) with `mapped_product_count`, useful when the router's `alternatives[]` span categories.
5. **Check the upstream rights** — `public_sources_api_public_sources_get` (`GET /api/public/sources`) lists each upstream source with `rights_status`, `rights_evidence_url`, `attribution_required`, `provenance_required` and `last_verified_at`; `coverage.complete` is `false`, so a service whose `source_ids` are not listed has no reviewed rights record yet.

## Then
- To buy, hand `service_id` to the discover-and-buy skill (`POST /execute/{capability_id}`), or to the MCP router tool `macaroons_execute`.
- Pagination is limit-only (max 20) on the search endpoints; `GET /api/public/services` returns the whole catalogue. See `conventions/macaroonnetwork-com-conventions.yml`.

## Errors
- 422 on the router means `intent` was missing or too long; 404 means an unknown `service_id`. See `errors/macaroonnetwork-com-problem-types.yml`.
