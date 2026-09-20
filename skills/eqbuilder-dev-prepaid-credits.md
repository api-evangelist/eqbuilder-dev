---
name: prepaid-credits
description: Buy prepaid basic-score credits (x402 bundle or card pack) and redeem them with X-BUNDLE-TOKEN
api: openapi/eqbuilder-dev-openapi.yml
base_url: https://eqbuilder.dev/api
operations:
- buy_bundle_api_bundle_post
- card_packs_api_card_packs_get
- card_checkout_api_card_checkout_post
- card_claim_api_card_claim_post
- card_refill_api_card_refill_post
- bundle_balance_api_bundle_balance_get
- run_simulation_api_simulate_post
- embedded_eq_check_api_embed_check_post
generated: '2026-09-19'
method: generated
grounding: operationIds grep-verified against the harvested spec; rules from llms.txt, /api/pricing and conventions/eqbuilder-dev-conventions.yml
---

# Buy prepaid basic-score credits (x402 bundle or card pack) and redeem them with X-BUNDLE-TOKEN

Two rails mint the same basic-tier credits:

A. Wallet (x402): `POST /api/bundle` (`buy_bundle_api_bundle_post`) — 1,000 basic credits at a published 15% discount; pay the 402 quote exactly as in `pay-and-benchmark-x402`. The response contains a SECRET `bundle_token` shown exactly once — store it server-side.

B. Card (a human, no crypto): `GET /api/card/packs` (`card_packs_api_card_packs_get`) lists starter $5 / builder $20 / fleet $50. `POST /api/card/checkout` (`card_checkout_api_card_checkout_post`) with `{"wallet_address", "pack"}` returns a hosted checkout URL plus a `claim_secret` (save it). After payment `POST /api/card/claim` (`card_claim_api_card_claim_post`) with `{"checkout_id", "claim_secret"}` mints the `bundle_token`. Monthly renewal is opt-in only (`auto_refill: true` at checkout); `POST /api/card/refill` (`card_refill_api_card_refill_post`) reconciles a settled renewal and is idempotent — it never starts a charge. Cancel renewals from the Whop account controls; settled credits remain.

Redeem: send `X-BUNDLE-TOKEN` on `POST /api/simulate` (`run_simulation_api_simulate_post`, basic tier only, no per-call payment) or, from a company-owned host proxy, on `POST /api/embed/check` (`embedded_eq_check_api_embed_check_post`) with an `Idempotency-Key` shaped `embed-<13-digit-ms>-<suffix>`; exact-key replays within 24 h consume no extra credit. Check the balance with `GET /api/bundle/balance` (`bundle_balance_api_bundle_balance_get`) — a missing header is `400 Missing X-BUNDLE-TOKEN header`.

Conventions: errors/eqbuilder-dev-problem-types.yml · idempotency + reversibility: conventions/eqbuilder-dev-conventions.yml · prices: plans/eqbuilder-dev-plans-pricing.yml.
