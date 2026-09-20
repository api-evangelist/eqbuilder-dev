---
name: pay-and-benchmark-x402
description: Pay for a validated benchmark with x402 (USDC on Base) and collect the certificate
api: openapi/eqbuilder-dev-openapi.yml
base_url: https://eqbuilder.dev/api
operations:
- simulate_quote_get_api_simulate_get
- get_simulate_fee_api_fee_simulate__wallet_address__get
- run_simulation_api_simulate_post
- get_certificate_api_certificate__tx_hash__get
- public_leaderboard_api_leaderboard_get
- get_wallet_statement_api_wallet__wallet_address__statement_get
generated: '2026-09-19'
method: generated
grounding: operationIds grep-verified against the harvested spec; rules from llms.txt, /api/pricing and conventions/eqbuilder-dev-conventions.yml
---

# Pay for a validated benchmark with x402 (USDC on Base) and collect the certificate

Preconditions: an explicitly authorized burner wallet holding USDC on Base (no ETH needed — EIP-3009 is gasless) and an operator payment policy. Installation alone never authorizes spending.

1. `GET /api/simulate` (`simulate_quote_get_api_simulate_get`) — read the live HTTP 402 x402 v2 quote for free. Optionally `GET /api/fee/simulate/{wallet_address}` (`get_simulate_fee_api_fee_simulate__wallet_address__get`) to preview the fixed tier fee.
2. `POST /api/simulate` (`run_simulation_api_simulate_post`) with `{"target_profile", "proposed_text", "response_delay_seconds", "wallet_address"}` and no payment header; it returns 402 with `accepts[]`. Select the entry with `network: eip155:8453` (Base USDC). For deep diagnostics add `"tier": "deep"` (separate fixed price).
3. Sign that entry's EIP-3009 `TransferWithAuthorization` as EIP-712 typed data (domain from `extra.name/version`, chainId 8453, verifyingContract = `asset`; `to = payTo`, `value = maxAmountRequired`, fresh 32-byte nonce, `validBefore` a few minutes ahead). Build the canonical x402 v2 payload from the full 402 body.
4. Retry the EXACT same request ONCE with the `PAYMENT-SIGNATURE` header (add `Idempotency-Key`; the operation declares it). Read `PAYMENT-RESPONSE` on success. The SDKs (`pip install "eqbuilder[payments]"`, `npm install eqbuilder viem`) do steps 1–4 in `score()` / `autoScore()`.
5. Errors: `402` again = payment rejected or underpaid before any money moved; `409` = the transaction signature was already used — never replay a settled proof; `429 paid_capacity_busy` = wait `Retry-After` and retry the SAME proof/key; a timeout AFTER sending payment = keep the receipt and check settlement before creating a new authorization.
6. Fees are charged on pass and fail alike and are not refundable. On a PASS, fetch `GET /api/certificate/{tx_hash}` (`get_certificate_api_certificate__tx_hash__get`); check rank with `GET /api/leaderboard?profile=<id>` (`public_leaderboard_api_leaderboard_get`).
7. Audit spend for free: `GET /api/wallet/{wallet_address}/statement?auth_tx_hash=<any settled tx hash of this wallet>` (`get_wallet_statement_api_wallet__wallet_address__statement_get`); foreign hashes return 403.

Conventions: errors/eqbuilder-dev-problem-types.yml · idempotency + reversibility: conventions/eqbuilder-dev-conventions.yml · prices: plans/eqbuilder-dev-plans-pricing.yml.
