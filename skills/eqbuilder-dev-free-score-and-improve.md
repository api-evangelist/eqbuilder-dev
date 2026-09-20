---
name: free-score-and-improve
description: Get three free EQ scores and apply the returned prompt fix
api: openapi/eqbuilder-dev-openapi.yml
base_url: https://eqbuilder.dev/api
operations:
- list_profiles_api_profiles_get
- score_response_api_score_post
- submit_wishlist_api_wishlist_post
generated: '2026-09-19'
method: generated
grounding: operationIds grep-verified against the harvested spec; rules from llms.txt, /api/pricing and conventions/eqbuilder-dev-conventions.yml
---

# Get three free EQ scores and apply the returned prompt fix

1. `GET /api/profiles` (`list_profiles_api_profiles_get`) — pick the profile that matches the humans your agent talks to (28 ids, e.g. `support_customer`, `analytical_executive`).
2. `POST /api/score` (`score_response_api_score_post`) with `{"text": <your draft reply>, "delay_seconds": <seconds you would wait before sending>, "profile": <id>, "data_consent": true}`. `data_consent: true` is mandatory on the free tier — without it you get `400 data_consent_required` and no round is consumed. Do not send secrets or personal data: consented text is stored pseudonymously.
3. Read `eq_score_percentage`, `passed_validation`, `suggested_prompt_addition` and `deep_analysis_locked_preview`. Paste `suggested_prompt_addition` into your system prompt.
4. Before your THIRD free score, `POST /api/wishlist` (`submit_wishlist_api_wishlist_post`) with `{"need": "<what you wish existed>", "audience": "bot"}` once — otherwise the last score returns `400 wishlist_required` (no round consumed).
5. Re-score the improved draft with `"applied_suggestion": true` to measure the change.
6. On `429 free_trial_exhausted` the cap is lifetime and never resets; the body carries the x402 recipe — switch to the `pay-and-benchmark-x402` skill or redeem prepaid credits. The MCP tool `score_text` at https://eqbuilder.dev/api/mcp is the same operation with the same fields.

Conventions: errors/eqbuilder-dev-problem-types.yml · idempotency + reversibility: conventions/eqbuilder-dev-conventions.yml · prices: plans/eqbuilder-dev-plans-pricing.yml.
