# ADR-005: Disabling Open-Session-In-View and Adopting JPA EntityGraphs

| Metadata | Details |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-09-24 |
| **Deciders** | Engineering Architecture Team |
| **Tags** | Database, JPA, Hibernate, Performance |

---

## Context & Problem Statement
Spring Boot ships with `spring.jpa.open-in-view=true` by default. This pattern keeps the Hibernate session and database connection open throughout the web request lifecycle, including the Jackson JSON serialization phase.
While convenient for rapid prototyping, OSIV causes critical production issues:
1. **N+1 Query Explosion**: Lazy associations accessed during JSON serialization trigger dozens of unintended SQL queries per HTTP request.
2. **Database Connection Pool Exhaustion**: Connections from the HikariCP pool remain tied to HTTP request threads that are performing slow I/O or client data writes.
3. **Architectural Bleed**: Database queries are executed in the presentation/view layer outside transactional boundaries.

## Decision Outcome
We explicitly configured:
```properties
spring.jpa.open-in-view=false
```
To fetch related entities cleanly without triggering `LazyInitializationException`:
1. All lazy loading is confined strictly to `@Transactional` service methods.
2. Required entity associations are fetched using Spring Data JPA 2.1 `@EntityGraph(attributePaths = {...})` or explicit JPQL `JOIN FETCH` queries.
3. Repositories provide specialized query methods for different retrieval projections (e.g., lightweight map-pins vs full property details).

## Consequences
### Positive
- Predictable and auditable SQL query counts per endpoint.
- Database connections are released back to the HikariCP pool immediately upon exiting the service layer.
- Clean architectural isolation: View layers and mappers never execute hidden SQL.

### Negative / Trade-offs
- Developers must explicitly declare `@EntityGraph` for any nested relationship needed in DTO mapping, or encounter a compile/test-time `LazyInitializationException` (which is easily caught by integration tests).
