# ADR-006: Compile-Time DTO Mapping with MapStruct

| Metadata | Details |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-09-26 |
| **Deciders** | Engineering Architecture Team |
| **Tags** | Architecture, Mapping, Performance, MapStruct |

---

## Context & Problem Statement
Mapping between JPA entities and REST API DTOs is critical to prevent exposing internal database structure and sensitive properties (e.g. password hashes, internal status flags) to API consumers.
Common approaches:
1. **Manual Builders / Setters**: Labor intensive, boilerplate heavy, error prone.
2. **Reflection-Based Mappers (ModelMapper / Dozer)**: Slow at runtime, difficult to debug, prone to unexpected mapping failures when types diverge, and masks breaking schema changes until runtime.

## Decision Outcome
We selected **MapStruct 1.5.5** with Java annotation processing:
- Mappers are declared as interfaces annotated with `@Mapper(componentModel = "spring")`.
- MapStruct generates plain Java mapping code at compile-time (`target/generated-sources`).
- Compile-time safety: Any missing field or incompatible type mismatch results in a compilation error rather than a runtime exception.

## Consequences
### Positive
- **Maximum Performance**: Executes as fast as hand-written code (direct getter/setter calls), with zero reflection overhead.
- **Fail-Fast Compilation**: Schema or DTO drift breaks the build immediately before tests run.
- **Spring Integration**: Generated mappers are automatically registered as injectable Spring `@Component` beans.

### Negative / Trade-offs
- Requires configured annotation processors in `pom.xml` (`maven-compiler-plugin`).
