# Contributing to Rentify Backend

Thank you for your interest in contributing to Rentify Backend! We welcome pull requests from everyone.

---

## 1. Development Environment Prerequisites

- **Java Development Kit (JDK)**: Java 17 LTS (Eclipse Temurin recommended)
- **Container Engine**: Docker Desktop or Podman (required for Testcontainers and local PostgreSQL/Redis)
- **Maven**: Maven 3.9+ (or use the included `./mvnw` wrapper)

---

## 2. Git Workflow & Branching Strategy

We follow the standard Git Feature Branch Workflow:

1. **Fork or Branch**: Create a descriptive branch from `main`:
   ```bash
   git checkout -b feat/availability-date-range-cache
   # or
   git checkout -b fix/wallet-pessimistic-lock-deadlock
   ```
2. **Branch Naming Conventions**:
   - `feat/*`: New features or API endpoints.
   - `fix/*`: Bug fixes or security patches.
   - `perf/*`: Performance and query optimizations.
   - `docs/*`: Documentation additions or revisions.
   - `refactor/*`: Code improvements without behavior changes.

---

## 3. Commit Message Conventions (Conventional Commits)

Commit messages must adhere strictly to the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```text
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

### Examples:
- `feat(booking): add date collision check against host manual availability blocks`
- `fix(security): sanitize return headers during dual-mode auth handshake`
- `docs(adr): record ADR-005 on open-in-view disabling`

---

## 4. Coding & Architecture Standards

1. **Zero Entity Leaks**: Never return JPA `@Entity` instances directly from REST controllers. Always map via compile-time MapStruct mappers into dedicated DTOs.
2. **Open-In-View Rule**: Do not rely on lazy loading outside `@Transactional` service boundaries (`open-in-view=false`). Always specify `@EntityGraph` for multi-level associations.
3. **Transaction Boundaries**: Ensure write operations declare `@Transactional`. Read-only queries should declare `@Transactional(readOnly = true)`.
4. **Validation**: All controller request payloads must be decorated with `@Valid` and use Jakarta Validation constraints (`@NotNull`, `@NotBlank`, `@Min`, `@Pattern`).

---

## 5. Verification & Testing Standards

Before opening a pull request, ensure all unit and integration tests pass locally:

```bash
# 1. Run unit test suite
./mvnw test

# 2. Run Testcontainers integration test suite
./mvnw verify

# 3. Verify Docker container builds
docker build -t rentify-backend:test .
```

---

## 6. Pull Request Process

1. Fill out all sections of [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md).
2. Reference any related issues (e.g. `Fixes #42`).
3. Ensure CI checks (Maven build, checkstyle, Testcontainers suite) report green.
4. Obtain at least one code review approval from a maintainer.
