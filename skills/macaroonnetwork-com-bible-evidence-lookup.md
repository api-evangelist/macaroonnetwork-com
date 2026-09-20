---
generated: '2026-09-19'
method: generated
name: Retrieve source-labelled scripture evidence for free
description: Look up exact Berean Standard Bible or World English Bible verses through the read-only GPT Action endpoints (or the free Bible Evidence MCP), with translation labels, corpus version and a content hash — no payment, no key.
api: openapi/macaroonnetwork-com-openapi.json
operations: [public_gpt_faith_scripture_lookup_api_public_gpt_faith_scripture_lookup_post, public_gpt_faith_services_list_api_public_gpt_faith_services_list_post, public_gpt_faith_service_evidence_api_public_gpt_faith_services__service_id__evidence_get]
source: Grounded in openapi/macaroonnetwork-com-openapi.json (operationIds verified), https://macaroonnetwork.com/bible-api and the live tools/list of https://api.macaroonnetwork.com/mcp-faith-evidence-v1/mcp/.
---

# Retrieve source-labelled scripture evidence for free

Free, anonymous, read-only. The corpus is the 66-book Protestant canon in BSB (`source_id` for Berean Standard Bible) and WEB (World English Bible); inputs use USFM-style book identifiers; runtime requests do not call third-party scripture APIs.

## Steps
1. **Look up a passage** — `public_gpt_faith_scripture_lookup_api_public_gpt_faith_scripture_lookup_post` (`POST /api/public/gpt/faith/scripture-lookup`) with `GPTScriptureLookupRequest` `{"source_id", "book_id", "chapter", "verse_start", "verse_end"}` (one to five same-chapter verses). Every response labels translation, corpus version, attribution, retrieval time and a deterministic content hash — keep the hash if you cite the text.
2. **List the paid Christian services** — `public_gpt_faith_services_list_api_public_gpt_faith_services_list_post` (`POST /api/public/gpt/faith/services/list`) returns the sale-ready christian-* capabilities (context, comparison, cross-references, reading plans, topic guides, devotional and confession reflections) with prices.
3. **Read a service's evidence before recommending it** — `public_gpt_faith_service_evidence_api_public_gpt_faith_services__service_id__evidence_get` (`GET /api/public/gpt/faith/services/{service_id}/evidence`) gives validation claims and stated limitations; the provider's own copy is explicit that these products offer "no absolution" and that "Payment buys the structured service, not divine favour or a sacrament".

## MCP alternative
- Connect an MCP client to `https://api.macaroonnetwork.com/mcp-faith-evidence-v1/mcp/` (Streamable HTTP, no key). `faith_scripture_lookup` takes the same five fields; `faith_passage_context_preview`, `faith_passage_compare_preview`, `faith_cross_references_preview`, `faith_scripture_search_preview` and `faith_topic_guide_preview` return bounded free previews of the paid twins. See `mcp/macaroonnetwork-com-bible-evidence-mcp.yml`.

## Errors
- 422 for a malformed book id or a verse range that crosses a chapter. 5 free calls per day per agent-or-IP apply to the paid christian-* listings, not to these read-only endpoints. See `errors/macaroonnetwork-com-problem-types.yml`.
