---
name: Track invested Notes, loans, and payments
description: Retrieve an investor's owned Notes, invested loans, and loan payment history from the Prosper Investor API for portfolio reconciliation.
api: https://developers.prosper.com/docs/investor/
operations:
  - GET /notes/
  - GET /loans/
  - GET /loans/payments/
---

# Track invested Notes, loans, and payments

Reconcile a Prosper investor portfolio.

## Auth

Obtain an OAuth 2.0 bearer token (see the search-and-invest skill) and send
`Authorization: bearer <access_token>` with `Accept: application/json`.

## Steps

1. **List Notes** — `GET /notes/` (page with `offset`/`limit`, max `limit=500`).
   Each note has `loan_note_id`, `note_ownership_amount`, `note_status` /
   `note_status_description`, `principal_balance_pro_rata_share`,
   `interest_paid_pro_rata_share`, and `prosper_rating`. Fetch a single note with
   `GET /notes/{loan_note_id}`.
2. **List loans** — `GET /loans/` for invested loans (`loan_number`,
   `principal_balance`, `loan_status`, `next_payment_due_date`,
   `next_payment_due_amount`). Fetch one with `GET /loans/{loan_id}`.
3. **List payments** — `GET /loans/payments/` for payment transactions
   (`transaction_id`, `payment_amount`, `principal_amount`, `interest_amount`,
   `payment_status`, `transaction_effective_date`, `resulting_principal_balance`).
4. **Reconcile** — join notes → loans → payments on `loan_number`; sum
   `principal_amount` + `interest_amount` per loan to track returns.

## Rules

- List responses wrap results in a `result` array with `result_count` and
  `total_count`; page until you have all rows.
- These are read-only operations; safe to retry on transient HTTP 5xx with backoff.
- Back off on HTTP 429.
