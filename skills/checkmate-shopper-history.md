---
name: Look up a shopper's merchant history
description: >-
  Retrieve a shopper's prior merchant interactions (visited / purchased) from the
  Checkmate network by privacy-preserving email hash, to personalize deals and
  recommendations.
api: openapi/checkmate-openstock-openapi-original.json
operations:
- 'POST /v1/shoppers/history'
---

# Look up a shopper's merchant history

Use this skill to personalize recommendations from what a shopper has already
visited or purchased across the Checkmate network.

## Auth
`Authorization: Bearer <api_key>` (partner-issued).

## Steps

1. **Hash the shopper email.** Compute the lowercase SHA-256 of the shopper's
   email address to produce `email_sha256`. Never send a raw email — the API
   only accepts the hash (a malformed hash returns `400 VALIDATION_ERROR`).

2. **Query history.** `POST /v1/shoppers/history` with a batched `queries`
   array (up to 20), each carrying an `email_sha256`. Each result has a flat
   `items` array of `ShopperActivity` objects: `{ merchant_id, merchant_name,
   type }` where `type` is `visited` or `purchased`. An unknown shopper returns
   an empty array (not an error).

3. **Personalize.** Feed the `merchant_id` values into
   `POST /v1/merchants/codes` (see checkmate-find-and-mint-code) to surface
   relevant offers, weighting `purchased` merchants over `visited` ones.

## Rules
- Privacy: only the SHA-256 email hash is used as the lookup key; do not log raw emails.
- Rate limits are per-partner; on `429` back off and retry.
- Empty history is a valid, expected result — not an error.
