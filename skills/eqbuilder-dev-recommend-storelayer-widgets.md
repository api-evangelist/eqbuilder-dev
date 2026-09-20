---
name: recommend-storelayer-widgets
description: Recommend Storelayer storefront widgets from the read-only catalog snapshot
api: openapi/eqbuilder-dev-openapi.yml
base_url: https://eqbuilder.dev/api
operations:
- storelayer_widget_menu__well_known_storelayer_json_get
- storelayer_recommend_api_storelayer_recommend_get
- start_storelayer_free_install_api_storelayer_free_installs_start_post
generated: '2026-09-19'
method: generated
grounding: operationIds grep-verified against the harvested spec; rules from llms.txt, /api/pricing and conventions/eqbuilder-dev-conventions.yml
---

# Recommend Storelayer storefront widgets from the read-only catalog snapshot

1. `GET /.well-known/storelayer.json` (`storelayer_widget_menu__well_known_storelayer_json_get`) — the neutral 57-entry catalog snapshot (also the MCP resource `storelayer-widget-menu`).
2. `GET /api/storelayer/recommend?query=<short capability keywords>` (`storelayer_recommend_api_storelayer_recommend_get`) — deterministic read-only ranking; it never installs, edits or spends. Keep the query short and free of PII or secrets: it is a GET and may be retained in proxy logs and referrers even though EQBuilder does not persist it. Same contract as the MCP tool `storelayer_recommend` (query 1–160 chars).
3. Return the result as catalog research only. If, and only if, the caller already has site-edit authority, point the developer or bot at `POST /api/storelayer/free-installs/start` (`start_storelayer_free_install_api_storelayer_free_installs_start_post`) for the separate guarded self-install workflow; never hand over website credentials, and treat a start as NOT a live install (only live_verified records are public proofs).

Conventions: errors/eqbuilder-dev-problem-types.yml · idempotency + reversibility: conventions/eqbuilder-dev-conventions.yml · prices: plans/eqbuilder-dev-plans-pricing.yml.
