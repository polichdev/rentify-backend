# ADR-001: Stateless JWT Authentication with Distributed Redis Blacklisting

| Metadata | Details |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-09-15 |
| **Deciders** | Engineering Architecture Team |
| **Tags** | Security, Auth, Redis, JWT |

---

## Context & Problem Statement
Rentify requires a high-throughput, horizontally scalable authentication mechanism across web browsers and mobile clients. 
Traditional stateful server sessions stored in PostgreSQL introduce database read overhead on every authenticated request and complicate horizontal scaling behind load balancers. Conversely, purely stateless JWTs lack an immediate revocation mechanism upon user logout, account suspension, or password rotation.

## Considered Options
1. **Server-Side Session Store in PostgreSQL**: High consistency, but creates continuous DB write/read pressure and high latency on high-frequency API endpoints.
2. **Purely Stateless JWT**: High scalability and zero DB reads, but impossible to revoke compromised or logged-out tokens before their expiration timestamp.
3. **Stateless JWT with Ephemeral Redis Blacklisting (Selected)**: Tokens are signed statelessly via HMAC-SHA256. Upon logout or token revocation, the token's unique ID (`jti`) is written to Redis with a TTL equal to the token's remaining lifespan.

## Decision Outcome
We adopted **Option 3: Stateless JWT with Redis Blacklisting**.
- **Execution**: On every request, `JwtAuthenticationFilter` validates the cryptographic signature. Only valid signatures perform an $O(1)$ fast check against Redis `EXISTS blacklist:{jti}`.
- **TTL Pruning**: Redis automatically prunes expired blacklist entries via native key eviction policies, keeping RAM usage bounded.

## Consequences
### Positive
- Negligible database load: 99% of authentication work happens in-memory with sub-millisecond Redis lookups.
- Immediate logout and session revocation without needing to cycle master secret keys.
- Horizontal scalability: Any number of backend instances can validate tokens using the shared Redis cluster.

### Negative / Trade-offs
- Introduced a hard runtime dependency on Redis availability for full revocation guarantees.
