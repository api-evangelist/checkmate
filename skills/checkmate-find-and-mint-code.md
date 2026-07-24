---
name: Find a merchant and mint a working discount code
description: >-
  Resolve a store to its Checkmate merchant ID, then request the best
  merchant-backed discount code for it via the OpenStock API. The killer
  agentic-commerce flow: turn a brand name into a ready-to-use checkout code.
api: openapi/checkmate-openstock-openapi-original.json
operations:
- 'POST /v1/merchants/search'
- 'POST /v1/merchants/codes'
---

# Find a merchant and mint a working discount code

Use this skill when a shopper or agent names a store ("find me a deal at Acme")
and you need a real, merchant-backed discount code to use at checkout.

## Auth
All requests need `Authorization: Bearer <api_key>` (partner-issued — request
developer access at https://openstock.sh). Only `GET /health` is unauthenticated.

## Steps

1. **Resolve the merchant.** `POST /v1/merchants/search` with a batched
   `queries` array (up to 50). Query by name or domain. Each result has an
   `items` array; a name query with `limit > 1` may return multiple candidates,
   and a miss returns `id: null`. Take the `id` (Checkmate merchant ID) of the
   best match.

2. **Request codes.** `POST /v1/merchants/codes` with a batched `queries` array
   (up to 20). Every query MUST include an `idempotency_key` (a missing key
   returns `400 VALIDATION_ERROR`) — this makes minting safe to retry. Results
   return an `items` array of `OfferCode` objects; a merchant with no code
   returns an empty `items` array.

3. **Pick the best code.** Rank `OfferCode` items by `probability` (estimated
   chance it works at checkout) and `value_amount` / `value_type`
   (`percent` | `fixed` | `free_shipping`). Respect `conditions[]`
   (e.g. `minimum_subtotal`) and `single_use`. Use `redirect_url` (the
   Checkmate affiliate-attributed deep link) to send the shopper to checkout.

## Rules
- Idempotency: always send a stable `idempotency_key` per codes query; reuse it on retry.
- Rate limits are per-partner; on `429` back off and retry.
- Errors use `{ "error": { "code", "message" } }` (not RFC 9457) — see errors/checkmate-problem-types.yml.
- Never fabricate a code; only surface codes returned by the API.
