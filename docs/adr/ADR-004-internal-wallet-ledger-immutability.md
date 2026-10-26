# ADR-004: Internal Wallet Architecture & Immutable Transaction Ledger

| Metadata | Details |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-09-22 |
| **Deciders** | Engineering Architecture Team |
| **Tags** | Financial, Wallet, Database, Ledger |

---

## Context & Problem Statement
Rentify allows users to hold internal credits (UAH) to pay for bookings, promote listings, or receive host earnings. 
A naive approach that only updates a `wallet_balance` column in the `users` table lacks an audit trail, makes financial dispute resolution impossible, and is vulnerable to untraceable balance alterations.

## Decision Outcome
We adopted an **Append-Only Immutable Ledger Pattern**:
1. **Running Balance**: The `users.wallet_balance` field represents the quick-access running balance for instantaneous balance checks.
2. **Immutable Transaction Log**: Every mutation MUST append a corresponding record to `wallet_transactions`.
3. **No Direct Mutations**: Updates to `wallet_balance` without a transaction record are strictly forbidden in code.
4. **Append-Only Enforcement**: Records in `wallet_transactions` cannot be modified or deleted (`UPDATE` and `DELETE` queries are disabled at repository and database level). Reversals are performed via new `REFUND` transactions.

## Consequences
### Positive
- Complete auditability: The running balance can be cryptographically and mathematically verified at any time by summing `wallet_transactions`.
- Compliance with financial logging and dispute resolution best practices.
- Zero lost financial history.

### Negative / Trade-offs
- Database table size grows monotonically, requiring indexing on `(user_id, created_at)`.
