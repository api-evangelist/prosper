---
name: Search listings and place an investment order
description: Authenticate to Prosper, check available cash, search active loan listings with filters, and place an order of bids to purchase Notes.
api: https://developers.prosper.com/docs/investor/
operations:
  - GET /accounts/prosper/
  - GET /listingsvc/v2/listings/
  - POST /orders/
  - GET /orders/{order_id}
---

# Search listings and place an investment order

Use the Prosper Investor API to invest available cash into active loan listings.

## Auth

1. Obtain an OAuth 2.0 token: `POST https://api.prosper.com/v1/security/oauth/token`
   with `grant_type=password`, `client_id`, `client_secret`, `username`, `password`.
   Use `grant_type=refresh_token` to refresh. Send `Authorization: bearer <access_token>`
   and `Accept: application/json` on every request. Use the sandbox host
   `https://api.sandbox.prosper.com/v1` for testing.

## Steps

1. **Check funds** — `GET /accounts/prosper/`. Read `available_cash_balance` to
   know how much you can invest.
2. **Search listings** — `GET /listingsvc/v2/listings/` with `biddable=true` and
   filters (e.g. `prosper_rating=AA,A`, `borrower_rate_min`, `fico_score_min`,
   `listing_term=36`) plus `sort_by=lender_yield desc`, `limit`, `offset`.
   Collect target `listing_number` values.
3. **Place order** — `POST /orders/` with a `bid_requests` array of
   `{listing_id, bid_amount}` (max 100 bids per order). Keep total bid amount
   at or under `available_cash_balance`.
4. **Confirm** — `GET /orders/{order_id}`. Wait for `order_status=COMPLETED`,
   then inspect each bid's `bid_status` (INVESTED/EXPIRED) and `bid_result`
   (BID_SUCCEEDED / PARTIAL_BID_SUCCEEDED / INSUFFICIENT_FUNDS / LISTING_NOT_BIDDABLE).

## Rules

- Idempotency: there is no Idempotency-Key header. Before retrying a failed
  `POST /orders/`, poll `GET /orders/` to confirm the order did not already land,
  so you do not double-bid.
- Handle `INSUFFICIENT_FUNDS` by re-reading the account balance; handle
  `LISTING_NOT_BIDDABLE` by re-searching (the listing filled or expired).
- Respect rate-limiting best practices; back off on HTTP 429.
