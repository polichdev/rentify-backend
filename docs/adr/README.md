# Architecture Decision Records (ADRs)

This directory contains records of the significant architectural decisions made during the evolution of **Rentify Backend**.

The format is based on Michael Nygard's Architecture Decision Record (ADR) template.

---

## Index of Decisions

| ADR | Title | Status | Date | Tags |
|:---:|---|:---:|:---:|---|
| [**ADR-001**](ADR-001-stateless-jwt-with-redis-blacklisting.md) | Stateless JWT Authentication with Distributed Redis Blacklisting | `Accepted` | 2026-09-15 | `Security`, `Auth`, `Redis`, `JWT` |
| [**ADR-002**](ADR-002-dual-mode-cookie-bearer-authentication.md) | Dual-Mode Authentication (Secure Cookies vs Bearer Tokens) | `Accepted` | 2026-09-18 | `Security`, `Auth`, `SPA`, `Mobile`, `CSRF` |
| [**ADR-003**](ADR-003-availability-engine-collision-matrix.md) | Availability Engine & Unified Collision Detection | `Accepted` | 2026-09-20 | `Booking`, `Concurrency`, `Algorithms` |
| [**ADR-004**](ADR-004-internal-wallet-ledger-immutability.md) | Internal Wallet Architecture & Immutable Transaction Ledger | `Accepted` | 2026-09-22 | `Financial`, `Wallet`, `Database`, `Ledger` |
| [**ADR-005**](ADR-005-open-in-view-disabled-and-entity-graph.md) | Disabling Open-Session-In-View and Adopting JPA EntityGraphs | `Accepted` | 2026-09-24 | `Database`, `JPA`, `Hibernate`, `Performance` |
| [**ADR-006**](ADR-006-compile-time-mapping-with-mapstruct.md) | Compile-Time DTO Mapping with MapStruct | `Accepted` | 2026-09-26 | `Architecture`, `Mapping`, `Performance` |
