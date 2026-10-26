# Rentify Backend — Property Rental Marketplace REST API

[![Tests: 200+ Passing](https://img.shields.io/badge/Tests-200%2B%20Passing-brightgreen.svg)](#testing--code-quality)
[![OpenAPI 3.0](https://img.shields.io/badge/OpenAPI-3.0_Spec-6BA539.svg?logo=openapiinitiative&logoColor=white)](docs/API_SURFACE.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

[![Java 17](https://img.shields.io/badge/Java-17-orange.svg?logo=openjdk&logoColor=white)](https://openjdk.org/projects/jdk/17/)
[![Spring Boot 3.4.3](https://img.shields.io/badge/Spring%20Boot-3.4.3-brightgreen.svg?logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Spring Security 6](https://img.shields.io/badge/Spring_Security-6.x-6DB33F.svg?logo=springsecurity&logoColor=white)](https://spring.io/projects/spring-security)
[![PostgreSQL 16](https://img.shields.io/badge/PostgreSQL-16-336791.svg?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Redis 7](https://img.shields.io/badge/Redis-7-DC382D.svg?logo=redis&logoColor=white)](https://redis.io/)
[![Hibernate / JPA](https://img.shields.io/badge/Hibernate-ORM_JPA-59666C.svg?logo=hibernate&logoColor=white)](https://hibernate.org/)
[![JWT Auth](https://img.shields.io/badge/JWT-Stateless_Auth-000000.svg?logo=jsonwebtokens&logoColor=white)](https://jwt.io/)
[![Google OAuth 2.0](https://img.shields.io/badge/Google_OAuth-2.0-4285F4.svg?logo=google&logoColor=white)](https://developers.google.com/identity)
[![Cloudinary](https://img.shields.io/badge/Cloudinary-Media_CDN-3448C5.svg?logo=cloudinary&logoColor=white)](https://cloudinary.com/)
[![MapStruct](https://img.shields.io/badge/MapStruct-1.5.5-E05D44.svg)](https://mapstruct.org/)
[![Testcontainers](https://img.shields.io/badge/Testcontainers-Integration_Tests-0B1C28.svg?logo=testcontainers&logoColor=white)](https://testcontainers.com/)
[![Docker](https://img.shields.io/badge/Docker-Multi--Stage-2496ED.svg?logo=docker&logoColor=white)](https://www.docker.com/)
[![Swagger UI](https://img.shields.io/badge/Swagger-OpenAPI_3-85EA2D.svg?logo=swagger&logoColor=black)](#swagger--openapi-documentation)

---

## Overview

**Rentify Backend** is a production-grade, highly scalable RESTful API powering a modern rental marketplace and property accommodation platform. It handles the complete lifecycle of property listings, geo-spatial discovery and faceted search, conflict-free booking reservations, internal wallet transactions, direct host-guest messaging, verified guest reviews, and monetization through promotion and subscription tiers.

Engineered with **Clean Architecture**, **Domain-Driven Design (DDD)** principles, and strict enterprise security standards, Rentify Backend delivers high-throughput request processing, strict data consistency, and seamless integration with modern web and mobile frontends.

<p align="center">
  <img src="docs/screenshots/api-overview.png" alt="Rentify Backend Architecture & API Overview" width="100%" />
</p>

---

## 📚 Engineering Documentation

The project includes production-grade engineering specifications, C4 architecture diagrams, threat models, and architectural decision records:

| Document | Purpose & Highlights |
|---|---|
| [**Architecture & System Design**](docs/ARCHITECTURE.md) | C4 Context & Container diagrams, Clean Architecture layers, Booking State Machine, Dual-Auth flows |
| [**Database Schema & Data Architecture**](docs/DATABASE_SCHEMA.md) | Mermaid ER diagram, table dictionaries, indexing strategies, audit policies, anti-N+1 rules |
| [**Concurrency & Transaction Locking**](docs/CONCURRENCY_AND_LOCKING.md) | Double-booking collision detection algorithm, wallet double-spending prevention (`FOR UPDATE`) |
| [**Threat Model & Security Architecture**](docs/THREAT_MODEL.md) | STRIDE framework analysis, trust boundaries, CSRF handshakes, IDOR prevention, RBAC matrix |
| [**Failures, Resilience & Recovery**](docs/FAILURES_AND_RECOVERY.md) | RFC 7807 ProblemDetail error format, HikariCP failover, Redis and Cloudinary outage fallbacks |
| [**Testing Strategy & Quality Assurance**](docs/TESTING_STRATEGY.md) | Testing pyramid, hermetic PostgreSQL 16 Testcontainers integration tests, mock policies |
| [**Deployment, Operations & DevOps**](docs/DEPLOYMENT_AND_DEVOPS.md) | Multi-stage Docker build, Docker Compose orchestration, environment variables, healthchecks |
| [**API Surface Specification**](docs/API_SURFACE.md) | Exhaustive REST endpoint catalog with request/response schemas and authorization requirements |
| [**Architecture Decision Records (ADRs)**](docs/adr/) | Formal decision logs (JWT with Redis blacklist, dual-mode auth, immutable wallet ledger, etc.) |

---

## Tech Stack

### Core Frameworks & Runtime
- **Java 17 (Eclipse Temurin)**: Long-term support Java runtime utilizing modern language features.
- **Spring Boot 3.4.3** (Spring Framework 6.x): Core enterprise application framework.
- **Spring Web MVC**: High-performance RESTful API controllers with standard HTTP semantics and OpenAPI annotations.
- **Spring Security 6 & JJWT (0.11.5)**: Stateless authentication architecture with HMAC-SHA256 tokens and Google OAuth2 ID token verification.
- **Spring Data JPA & Hibernate**: ORM layer with `open-in-view=false`, entity graph fetching, and query optimization.
- **Jakarta Bean Validation**: Strict runtime validation of all request DTO payloads.

### Persistence, Cache & In-Memory Layer
- **PostgreSQL 16**: Relational storage with ACID compliance, indexed multi-column filters, and transactional integrity.
- **Redis 7 (Alpine)**: Distributed in-memory data store powering instant JWT token revocation lists (blacklisting), session state, and fast cache lookups.
- **Auditing & Entity Lifecycle**: Base `@MappedSuperclass` models (`AuditableEntity`, `CreatedAtEntity`) for automated timestamping and lifecycle tracking.

### Security & Session Management
- **Dual-Mode Authentication Strategy**: Configurable auth strategy supporting secure HTTP-only cookies (`SameSite=Lax`, `Secure`, `HttpOnly`) for modern SPAs and standard `Authorization: Bearer <token>` headers for mobile clients.
- **CSRF Token Handshake**: Cookie-to-header cross-site request forgery prevention mechanism (`X-CSRF-Token`) for cookie-based authentication.
- **Granular RBAC**: Role-Based Access Control (`ROLE_GUEST`, `ROLE_HOST`, `ROLE_ADMIN`) securing listing operations, booking confirmations, and wallet transactions.
- **BCrypt Password Hashing**: Adaptive hashing algorithm securing user credentials.

### Media & Cloud Integrations
- **Cloudinary Java SDK (1.39.0)**: Cloud-native multipart image uploading, format transformations, secure deletion, and global CDN delivery for property galleries and user avatars.
- **Google OAuth 2.0 Client**: Token verifier for social single sign-on (SSO).

### Mapping & Documentation
- **MapStruct 1.5.5**: Zero-overhead, compile-time type-safe DTO-to-entity mappers.
- **Project Lombok**: Boilerplate reduction for data models and dependency injection.
- **SpringDoc OpenAPI 3.0 (v2.8.5)**: Automated OpenAPI 3.0 specification generation and interactive Swagger UI.

### Testing & DevOps
- **Testcontainers 1.21.4**: Hermetic PostgreSQL container testing ensuring exact production parity in integration test suites.
- **JUnit 5, Mockito & AssertJ**: 200+ unit and integration tests across services, mappers, and controllers.
- **Multi-Stage Docker**: Lightweight Eclipse Temurin JRE container builds.
- **Docker Compose**: Multi-service local environment orchestration (PostgreSQL, Backend API, Frontend SPA).

---

## Core Features

- **Multi-Strategy Authentication & Google SSO**: User onboarding via Email/Password or Google OAuth 2.0. Dynamic token delivery via HTTP-only secure cookies (with CSRF protection) or Bearer header tokens.
- **User Profile & Avatar Management**: Complete profile lifecycle with Cloudinary avatar uploads, secure password rotation, and safe account deactivation.
- **Faceted Property Search & Interactive Map Pins**: Multi-criteria search filters (city, district, price min/max, guest capacity, bedrooms, amenities, property type, available dates) paired with high-speed `/map-pins` endpoint for lightweight geo-coordinate clustering.
- **Host Property Lifecycle & Media Gallery**: Host property CRUD operations, lifecycle state management (`DRAFT`, `ACTIVE`, `INACTIVE`, `BLOCKED`), multi-photo uploads with Cloudinary CDN optimization, and ownership enforcement.
- **Smart Availability Engine & Collision Prevention**: Unified calendar engine merging real-time guest booking reservations with host manual availability blocks, with automated date-range collision detection preventing double-bookings.
- **End-to-End Booking Workflow**: Multi-state reservation state machine (`PENDING`, `CONFIRMED`, `REJECTED`, `CANCELLED`, `COMPLETED`), automatic price calculation (nightly rate, cleaning fee, service fee, total), and host approval/rejection workflows.
- **Internal Wallet & Transaction Ledger**: In-app wallet management with configurable currency (`UAH`), predefined top-up amounts, balance validation, booking payment execution, and paginated transaction audit history.
- **Monetization & Promotion Packages**: TOP property placement, subscription packages, and automated promotion expiration schedulers to boost listing visibility in search results.
- **Direct Host-Guest Messaging**: Context-aware messaging threads tied to properties and bookings, message history, unread counters, and instant communication.
- **Verified Reviews & Multi-Criteria Ratings**: Post-stay feedback loop allowing guests to review completed bookings with rating categories (cleanliness, accuracy, communication, location, check-in, value) and dynamic property rating aggregates.
- **Favorites & Wishlists**: One-click bookmarking for properties with instant personal collection retrieval.
- **Location Taxonomy & Amenity Directories**: Autocomplete location suggestion service for cities, districts, metro stations, and residential complexes alongside category-grouped amenities.

---

## System Architecture & Patterns

Rentify Backend is designed as a modular, maintainable Clean Architecture service.

### Architecture Overview
- **Layered Architecture**: Strict separation of concerns across Presentation (Controllers & DTOs), Domain/Service Layer (Business rules, validations, state machines), and Data Access (Spring Data JPA Repositories & Entities).
- **Dual Authentication Strategy Pattern**: Seamless switching between stateless Bearer header authorization and browser-native HTTP-only cookies with CSRF token validation (`AuthCookieService`).
- **Conflict-Free Booking & Availability Matrix**: Transactional date-range validation algorithm that verifies zero collisions against both active reservations and manual calendar blocks within transactional boundaries.
- **Compile-Time DTO Serialization (MapStruct)**: High-performance DTO mapping generated at compile time, eliminating runtime reflection overhead and preventing accidental internal entity exposure.
- **Database Query Optimization & Anti-N+1 Protection**: Disabled `open-in-view` pattern (`spring.jpa.open-in-view=false`) forcing explicit transaction-scoped data fetching via `@EntityGraph` and custom JPQL queries.
- **Auditing & Entity Lifecycle Tracking**: Base classes (`AuditableEntity`, `CreatedAtEntity`) ensuring transparent timestamping and auditing across all persistent entities.
- **Hermetic Integration Testing with Testcontainers**: Automated spin-up of containerized PostgreSQL instances (`AbstractIntegrationTest`) ensuring test isolation and zero reliance on mocked databases.

---

## Testing & Code Quality

The codebase undergoes rigorous automated testing and static analysis to guarantee stability, data integrity, and production readiness:

- **200+ Unit & Integration Tests**: Comprehensive JUnit 5 and Mockito test suite covering business logic, domain services, custom validation rules, and error handling.
- **Testcontainers PostgreSQL Integration Suite**: Real PostgreSQL container test suite validating authentication flows, cookie sessions, booking lifecycles, review submissions, property creation, search queries, and wallet operations.
- **Strict Jakarta Validation**: Strict constraint annotations (`@Valid`, `@NotNull`, `@NotBlank`, `@Positive`, `@Future`) on all incoming request payloads to eliminate invalid data at the boundary.
- **Multi-Stage Docker Validation**: Containerized build and packaging verification.

### Running Tests Locally

```bash
# Run all unit and integration tests (requires Docker Desktop running)
./mvnw clean test

# Run a specific integration test class
./mvnw test -Dtest=RegistrationIntegrationTest

# Run unit tests only
./mvnw test -Dtest=*UnitTest
```

---

## Repository Structure

```
rentify-backend/
├── .mvn/                               # Maven wrapper configuration
├── docs/
│   ├── API_SURFACE.md                  # Complete REST API & endpoint specification
│   └── screenshots/
│       └── api-overview.png            # Architecture & API overview diagram
├── src/
│   ├── main/
│   │   ├── java/com/rentify/core/
│   │   │   ├── config/                 # Security, OpenAPI, Cloudinary, CORS, Cookie configs
│   │   │   ├── controller/             # REST API controllers (Auth, Booking, Property, Wallet, etc.)
│   │   │   ├── dto/                    # Request & Response Data Transfer Objects
│   │   │   ├── entity/                 # JPA Domain Entities (User, Property, Booking, Payment, etc.)
│   │   │   ├── enums/                  # Domain Enums (BookingStatus, PropertyStatus, Roles, etc.)
│   │   │   ├── exception/              # Global exception handling & custom domain errors
│   │   │   ├── mapper/                 # MapStruct compile-time DTO mappers
│   │   │   ├── repository/             # Spring Data JPA repositories with custom JPQL queries
│   │   │   ├── scheduler/              # Scheduled tasks (promotion expiries, cleanup jobs)
│   │   │   ├── security/               # JWT utilities, Custom UserDetailsService, Auth filters
│   │   │   ├── service/                # Domain business logic interfaces & implementations
│   │   │   ├── validation/             # Custom Jakarta validators and annotations
│   │   │   └── RentifyBackendApplication.java
│   │   └── resources/
│   │       ├── application.properties  # Main application configuration
│   │       └── application-secret.properties # Local secret overrides (gitignored)
│   └── test/
│       └── java/com/rentify/core/
│           ├── integration/            # Testcontainers PostgreSQL full-stack integration tests
│           └── unit/                   # Isolated Mockito unit tests for service layer
├── Dockerfile                          # Multi-stage Eclipse Temurin JRE container build
├── docker-compose.yml                  # Local multi-service infrastructure (PostgreSQL, Backend, Frontend)
├── pom.xml                             # Maven dependencies & build configuration
└── README.md                           # Project documentation
```

---

## Running Locally

### Prerequisites
- **Java 17+** (JDK)
- **Maven 3.9+** (or use included `./mvnw`)
- **Docker & Docker Compose** (for PostgreSQL database and Testcontainers)
- **Cloudinary Account** (for property photo and avatar uploads)

---

### Option 1: Hybrid Development (Recommended)

Run PostgreSQL in Docker, and run Backend locally with hot-reloading:

```bash
# 1. Clone the repository
git clone https://github.com/polchduikt/rentify-backend.git
cd rentify-backend

# 2. Start PostgreSQL container
docker compose up -d db

# 3. Create .env file or application-secret.properties
cp .env.example .env

# 4. Start the Spring Boot application
./mvnw spring-boot:run
```

---

### Option 2: Full Docker Stack

Run PostgreSQL, Backend API, and Frontend SPA simultaneously in Docker:

```bash
docker compose up --build -d
```

To stop all containers:

```bash
docker compose down
```

---

### Environment Variables

Configure environment variables in `.env` or in `src/main/resources/application-secret.properties`:

```properties
# Database Configuration
DB_URL=jdbc:postgresql://localhost:5432/rentify
DB_USERNAME=postgres
DB_PASSWORD=your_secure_password

# Security & JWT
SECRET_KEY=your_base64_encoded_jwt_secret_key_min_256_bits
GOOGLE_CLIENT_ID=your_google_oauth_client_id

# Auth Strategy (cookie or bearer)
AUTH_STRATEGY=cookie
AUTH_COOKIE_NAME=rentify_access_token
AUTH_COOKIE_DOMAIN=
AUTH_COOKIE_SECURE=false
AUTH_COOKIE_SAME_SITE=Lax
CSRF_COOKIE_NAME=csrf_token
CSRF_HEADER_NAME=X-CSRF-Token
CSRF_COOKIE_SECURE=false
ALLOWED_ORIGINS=http://localhost:5173,http://localhost:3000

# Wallet & Currency
WALLET_CURRENCY=UAH
WALLET_TOP_UP_OPTIONS=300.00,500.00,1000.00

# Cloudinary CDN Integration
CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret
```

---

### Testcontainers Setup on Windows

To run Testcontainers integration tests smoothly on Windows:

1. In **Docker Desktop** -> **Settings** -> **General** -> Enable `Expose daemon on tcp://localhost:2375 without TLS`.
2. Run in PowerShell (Administrator):

```powershell
[System.Environment]::SetEnvironmentVariable("DOCKER_API_VERSION", "1.47", "Machine")
New-Item -Path "$env:USERPROFILE\.testcontainers.properties" -ItemType File -Force
Set-Content "$env:USERPROFILE\.testcontainers.properties" "docker.host=npipe:////./pipe/docker_engine_linux`ntestcontainers.reuse.enable=true"
```

3. Restart your IDE and run `./mvnw test`.

---

### Available Local Endpoints:
- **Backend REST API**: [http://localhost:8080/api/v1](http://localhost:8080/api/v1)
- **Swagger / OpenAPI Documentation**: [http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html)
- **OpenAPI JSON Specification**: [http://localhost:8080/v3/api-docs](http://localhost:8080/v3/api-docs)
- **PostgreSQL Database**: `localhost:5432`

---

## API Documentation & Surface

The complete, versioned REST API specification, request/response payload schemas, parameter types, and security policies are documented in the dedicated reference guide:

[Complete API Surface Specification (docs/API_SURFACE.md)](docs/API_SURFACE.md)

### Interactive OpenAPI / Swagger UI
The backend provides real-time interactive OpenAPI 3.0 documentation generated automatically via SpringDoc:

- **Swagger UI**: [http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html)
- **OpenAPI 3.0 JSON Schema**: [http://localhost:8080/v3/api-docs](http://localhost:8080/v3/api-docs)

To test secured endpoints directly in Swagger UI:
1. Authenticate via `POST /api/v1/sessions` (or `POST /api/v1/users`).
2. Copy the returned `token`.
3. Click the **Authorize** button at the top of the Swagger page and enter `Bearer <your_token>`.

---

## Status

Rentify Backend is actively maintained and continuously enhanced with new features, performance optimizations, and integrations.

---

## License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.


