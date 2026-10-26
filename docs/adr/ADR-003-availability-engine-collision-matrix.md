# ADR-003: Availability Engine & Unified Collision Detection

| Metadata | Details |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-09-20 |
| **Deciders** | Engineering Architecture Team |
| **Tags** | Booking, Concurrency, Algorithms |

---

## Context & Problem Statement
In an accommodation marketplace, double-bookings are critical business failures. The system has two distinct mechanisms that can occupy property dates:
1. **Confirmed / Pending Bookings**: Created by guests reserving dates.
2. **Manual Availability Blocks**: Created by hosts to mark maintenance, personal stays, or off-platform bookings.

A single query against only the `bookings` table would miss host calendar blockouts, leading to reservation disputes.

## Decision Outcome
We implemented a **Unified Collision Matrix Engine** that enforces overlap validation across both domains within an atomic database transaction.

### Overlap Condition
Two intervals $[A_{\text{start}}, A_{\text{end}}]$ and $[B_{\text{start}}, B_{\text{end}}]$ overlap if and only if:
$$A_{\text{start}} < B_{\text{end}} \land A_{\text{end}} > B_{\text{start}}$$

- Any booking creation, host approval, or manual block creation executes this check across both `BookingRepository` and `AvailabilityBlockRepository`.
- If any count $> 0$, the service aborts with `BookingConflictException` (HTTP 409).
- Status filtering: Bookings marked `CANCELLED` or `REJECTED` are explicitly excluded from collision detection.

## Consequences
### Positive
- 100% mathematical guarantee against overbooking and calendar collisions.
- Unified calendar visualization endpoint combining active reservations and blocks in a single payload.

### Negative / Trade-offs
- Slight indexing requirement on `(property_id, start_date, end_date)` across two tables.
