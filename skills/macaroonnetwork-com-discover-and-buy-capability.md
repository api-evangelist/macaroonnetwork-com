---
generated: '2026-09-19'
method: generated
name: Discover a capability and buy one execution over x402
description: Find a Macaroon Network capability by intent, read its price and acceptance predicate, pay exactly once in USDC on Base via x402 v2, and reconcile with the receipt.
api: openapi/macaroonnetwork-com-openapi.json
operations: [search_listings_listings_search_get, get_listing_listings__capability_id__get, execute_payment_required_execute__capability_id__get, execute_capability_execute__capability_id__post, get_receipt_api_receipts__receipt_id__get]
source: >-
  Grounded in openapi/macaroonnetwork-com-openapi.json (operationIds verified), https://macaroonnetwork.com/auth.md
  and the curl walkthrough on https://macaroonnetwork.com/listings/vat-validate-v1.
---

# Discover a capability and buy one execution over x402

Base URL `https://api.macaroonnetwork.com`. No account, API key or OAuth exists — payment is the only gate. See `authentication/macaroonnetwork-com-authentication.yml`.

## Before you spend
- Every call costs real USDC on Base mainnet; there is no testnet or sandbox. Set a spend cap in your wallet layer first.
- Settlement is predicate-gated: money settles only if the acceptance predicate you send passes against the real payload. A failed predicate refunds automatically. A passed predicate cannot be reversed. See `conventions/macaroonnetwork-com-conventions.yml` (reversibility).
- There is no idempotency key. If a paid request times out, check the receipt before paying again.

## Steps
1. **Search by intent** — `search_listings_listings_search_get` (`GET /listings/search?intent=<text>&limit=5`). Read `capability_id`, `price_sats`/`payment.amount_atomic` (USDC, 6 decimals), `sample_predicate`, `predicate_hash` and `input_schema` from each hit.
2. **Pin the listing** — `get_listing_listings__capability_id__get` (`GET /listings/{capability_id}`). Confirm `status` is `active`, read `policy.limitations` and `policy.free_tier` (if enabled, the first calls are free per self-declared `X-Macaroon-Agent-Id` and are consumed only on predicate pass).
3. **Preflight the price** — `execute_payment_required_execute__capability_id__get` (`GET /execute/{capability_id}`) returns HTTP 402 with the `PAYMENT-REQUIRED` header and never executes anything. Decode the base64 JSON: check `accepts[0].network` is `eip155:8453`, `amount` matches the listing, and `maxTimeoutSeconds` (60).
4. **Execute unpaid to get the exact challenge** — `execute_capability_execute__capability_id__post` (`POST /execute/{capability_id}`) with body `{"input": {...per input_schema, no extra fields...}, "predicate": {...}}`. Expect 402 + `PAYMENT-REQUIRED` for this exact request. A 422 means the body failed `input_schema` (fix `detail[].loc`).
5. **Pay and retry** — sign the requirement with your x402 v2 wallet and retry the identical POST with `PAYMENT-SIGNATURE: <base64 signed payload>` (the listing page also shows `X-Macaroon-Payment-Rail: x402` and `X-Macaroon-X402-Network: base`). Success is 200 with `{"payload", "predicate_passed", "receipt"}` and a `PAYMENT-RESPONSE` header.
6. **Reconcile** — `get_receipt_api_receipts__receipt_id__get` (`GET /api/receipts/{receipt_id}`) returns `settlement_state`, `predicate_result`, `amount`, `payment_reference` and `result_hash`. Use it after any timeout before re-sending payment.

## Errors
- 402 = pay (not an auth failure); 422 = body/schema; 404 = unknown capability id (use the exact id, never the `{capability_id}` template). See `errors/macaroonnetwork-com-problem-types.yml`.
- `predicate_passed: false` on a 200 is a real result (e.g. an invalid VAT number) and is not charged.

## Notes
- The same id is the A2A skill id and the MCP `macaroons_execute` `capability_id`; the MCP router at `https://api.macaroonnetwork.com/mcp` performs steps 1, 4 and 5 for you and returns the challenge for your own wallet to sign.
- Treat `policy.license_gate` and `validation.status` exactly as returned; the spec's own guidance is "unknown evidence is not permission".
