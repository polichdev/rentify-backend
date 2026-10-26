# Rentify Backend — System Architecture & Design Specification

| Field | Value |
|---|---|
| **System** | Rentify Marketplace Backend REST API |
| **Runtime & Platform** | Java 17 LTS / Spring Boot 3.4.3 / Spring Security 6 |
| **Persistence** | PostgreSQL 16 (ACID Relational), Redis 7 (In-Memory Blacklist & Cache) |
| **Architecture Pattern** | Clean Layered Architecture, Domain-Driven Design (DDD) Modularity |
| **Document Classification** | Public Engineering Architecture Specification |

---

## 1. Executive Summary & Core Mission

**Rentify** is a production-grade accommodation booking and real estate rental marketplace backend. It facilitates property discovery, faceted search with geo-clustering, collision-free calendar reservations, host-guest communication, an immutable internal wallet transaction ledger, post-stay verified reviews, and monetization through promotional listing tiers.

The backend is engineered with strict enterprise design rules:
- **Zero-leaks Domain Decoupling**: Database entities are strictly encapsulated within transactional service boundaries. DTOs are mapped at compile-time via MapStruct without runtime reflection overhead.
- **Stateless High-Throughput Auth**: JWT-driven sessions with distributed Redis token blacklisting and dual-mode delivery (HTTP-only SameSite cookies with CSRF validation for SPAs, Bearer tokens for mobile).
- **Zero Double-Booking Guarantee**: Transactional calendar collision engine combining active reservations and host availability blocks with strict date-range collision detection.
- **Anti-N+1 Protection**: `spring.jpa.open-in-view=false` is enforced globally; all association fetching is explicit via JPA 2.1 `@EntityGraph` and JOIN FETCH queries.

---

## 2. High-Level Architecture (C4 Model)

### 2.1 System Context (C4 Level 1)

```mermaid
flowchart TB
    subgraph Users["End Users"]
        GUEST["Guest Client<br/>(Web SPA / Mobile App)"]
        HOST["Host Client<br/>(Web SPA / Mobile App)"]
        ADMIN["Platform Admin<br/>(Admin Dashboard)"]
    end

    subgraph RentifySystem["Rentify Platform"]
        API["Rentify Backend REST API<br/>(Spring Boot 3.4.3 / Java 17)"]
    end

    subgraph ExternalServices["External Cloud & Identity Providers"]
        GOOGLE["Google Identity Services<br/>(OAuth 2.0 / OpenID Connect)"]
        CLOUDINARY["Cloudinary Media CDN<br/>(Image Transformation & Hosting)"]
    end

    subgraph Storage["Persistence & Caching Tier"]
        PG["PostgreSQL 16 Database<br/>(Relational Storage, GiST/B-tree Indexes)"]
        REDIS["Redis 7 In-Memory Store<br/>(Token Blacklist, Sessions, Ephemeral Cache)"]
    end

    GUEST -->|"HTTPS / JSON REST API"| API
    HOST -->|"HTTPS / JSON REST API"| API
    ADMIN -->|"HTTPS / JSON REST API"| API

    API -->|"OAuth Token Verification"| GOOGLE
    API -->|"Multipart Upload & CDN Purge"| CLOUDINARY
    API -->|"JDBC / HikariCP Pool"| PG
    API -->|"Lettuce Reactive / Sync Driver"| REDIS
```

### 2.2 Container Diagram (C4 Level 2)

```mermaid
flowchart TB
    subgraph Clients["Clients Tier"]
        WEB["React / Vue SPA Client"]
        MOBILE["Flutter / React Native Mobile Client"]
    end

    subgraph BackendApp["Spring Boot Application Container (:8080)"]
        direction TB
        GATEWAY["Security Filter Chain<br/>(JwtAuthenticationFilter + CsrfFilter)"]
        DISPATCHER["Spring Web MVC Dispatcher"]
        
        subgraph Subsystems["Core Domain Modules"]
            AUTH_MOD["Auth & Session Module"]
            PROP_MOD["Property & Search Module"]
            BOOK_MOD["Booking & Availability Module"]
            WALLET_MOD["Wallet & Transaction Module"]
            CHAT_MOD["Messaging Module"]
            REVIEW_MOD["Review & Rating Module"]
        end

        MAPPER["MapStruct Mappers<br/>(Compile-Time DTO <-> Entity)"]
        DATA_ACCESS["Spring Data JPA Repositories<br/>(Hibernate ORM, open-in-view=false)"]
    end

    subgraph Datastores["Data Tier"]
        POSTGRES[("PostgreSQL 16<br/>Port :5432")]
        REDIS_CACHE[("Redis 7<br/>Port :6379")]
    end

    WEB -->|"HTTPS Cookies + X-CSRF-Token"| GATEWAY
    MOBILE -->|"HTTPS Authorization: Bearer"| GATEWAY
    GATEWAY --> DISPATCHER
    DISPATCHER --> Subsystems
    Subsystems <--> MAPPER
    Subsystems --> DATA_ACCESS
    AUTH_MOD <-->|"Token Blacklist & Expiry"| REDIS_CACHE
    DATA_ACCESS <-->|"HikariCP Pool"| POSTGRES
```

---

## 3. Layered Clean Architecture & Data Flow

Rentify Backend adheres to strict unidirectional dependency layering:

```
[ HTTP Request ]
       │
       ▼
┌────────────────────────────────────────────────────────┐
│ Presentation Layer (Controllers & Advisers)            │
│ - Accepts validated DTOs (@Valid, Jakarta Validation)  │
│ - GlobalExceptionHandler (RFC 7807 ProblemDetail)      │
└───────────────────────┬────────────────────────────────┘
                        │ DTOs
                        ▼
┌────────────────────────────────────────────────────────┐
│ Domain & Service Layer (Business Logic & Workflow)     │
│ - State machines & business invariants                 │
│ - Transaction boundaries (@Transactional)              │
│ - MapStruct bidirectional translation                  │
└───────────────────────┬────────────────────────────────┘
                        │ Entities
                        ▼
┌────────────────────────────────────────────────────────┐
│ Data Access Layer (Repositories & Specifications)      │
│ - Spring Data JPA Repositories                         │
│ - JPA Criteria Specifications (Faceted dynamic query)  │
│ - @EntityGraph / JPQL explicit fetches                 │
└───────────────────────┬────────────────────────────────┘
                        │ SQL
                        ▼
┌────────────────────────────────────────────────────────┐
│ Infrastructure & External Adapters                     │
│ - PostgreSQL 16 (HikariCP)                             │
│ - Redis 7 (Lettuce)                                    │
│ - Cloudinary API SDK                                   │
└────────────────────────────────────────────────────────┘
```

---

## 4. Subsystem Architectures

### 4.1 Dual-Mode Authentication & Session Subsystem

Rentify Backend supports dual consumer profiles: single-page applications (vulnerable to XSS if tokens are in LocalStorage) and mobile native applications (which lack cookie handling conventions).

```mermaid
sequenceDiagram
    autonumber
    actor Client as SPA / Mobile Client
    participant AuthCtrl as SessionController
    participant AuthSvc as AuthenticationService
    participant CookieSvc as AuthCookieService
    participant Redis as Redis Cache
    participant DB as PostgreSQL

    Client->>AuthCtrl: POST /api/v1/sessions (email, password)
    AuthCtrl->>AuthSvc: authenticate(request)
    AuthSvc->>DB: findByEmail(email)
    DB-->>AuthSvc: UserEntity (BCrypt check)
    AuthSvc->>AuthSvc: Generate signed JJWT (HS256)
    
    alt Strategy == COOKIE (SPA)
        AuthSvc->>CookieSvc: createAuthCookie(jwt) + createCsrfCookie()
        CookieSvc-->>AuthCtrl: Set-Cookie: jwt_token (HttpOnly, Secure, SameSite=Lax)<br/>Set-Cookie: XSRF-TOKEN
        AuthCtrl-->>Client: 200 OK (User info, token in Set-Cookie)
    else Strategy == BEARER (Mobile)
        AuthCtrl-->>Client: 200 OK { token: "eyJhbGciOi...", user: {...} }
    end

    Note over Client,Redis: Logout Flow
    Client->>AuthCtrl: DELETE /api/v1/sessions/current
    AuthCtrl->>Redis: SETEX blacklist:{jti} remainingTTL "revoked"
    AuthCtrl-->>Client: 204 No Content
```

### 4.2 Booking State Machine & Availability Engine

Bookings navigate a deterministic lifecycle enforced at the service level:

```mermaid
stateDiagram-v2
    [*] --> PENDING: Guest submits booking request
    PENDING --> CONFIRMED: Host approves / Auto-confirm
    PENDING --> REJECTED: Host declines request
    PENDING --> CANCELLED: Guest cancels before confirmation
    CONFIRMED --> CANCELLED: Guest or Host cancels (subject to policy)
    CONFIRMED --> COMPLETED: Stay date ends (Checkout + 1 day)
    REJECTED --> [*]
    CANCELLED --> [*]
    COMPLETED --> [*]: Unlocks Verified Review creation
```

#### Collision Prevention Algorithm
When calculating availability or approving a reservation, the calendar engine executes an atomic range check across two tables:
1. `Booking`: Status is not `CANCELLED` and not `REJECTED`.
2. `AvailabilityBlock`: Host manual blackouts or maintenance periods.

$$\text{Collision Condition: } (\text{RequestedStart} < \text{ExistingEnd}) \land (\text{RequestedEnd} > \text{ExistingStart})$$

```mermaid
flowchart TD
    START([Guest Booking Request]) --> VALIDATE[Validate Check-In < Check-Out]
    VALIDATE --> QUERY_BOOKINGS[Query Active Bookings for Property in Range]
    QUERY_BOOKINGS --> CHECK_BOOKINGS{Any Booking Collision?}
    CHECK_BOOKINGS -- Yes --> REJECT_COLLISION[Throw BookingConflictException 409]
    CHECK_BOOKINGS -- No --> QUERY_BLOCKS[Query AvailabilityBlock for Property in Range]
    QUERY_BLOCKS --> CHECK_BLOCKS{Any Block Collision?}
    CHECK_BLOCKS -- Yes --> REJECT_COLLISION
    CHECK_BLOCKS -- No --> CALC_PRICE[Calculate Nights x Price + Cleaning + Service Fee]
    CALC_PRICE --> SAVE_PENDING[Save Booking Status = PENDING]
    SAVE_PENDING --> EMIT_EVENT([Notify Host via Messaging / Notification])
```

### 4.3 Internal Wallet & Ledger Immutability

To guarantee zero money leakage and audit compliance:
1. User wallets hold a running balance (`balance`).
2. Every mutation (top-up, booking hold, payout, refund) is backed by an append-only `WalletTransaction` row.
3. Balance mutations use database-level transactional locks (`@Transactional`) to prevent race conditions during simultaneous actions.

```mermaid
flowchart LR
    WALLET[User Wallet Balance]
    TOPUP[Top-up Operation] -->|+ Amount| WALLET
    BOOKING[Booking Payment] -->|- Amount| WALLET
    REFUND[Booking Cancellation] -->|+ Refund Amount| WALLET
    
    TOPUP -.->|Appends Entry| TX_LOG[(WalletTransaction Ledger)]
    BOOKING -.->|Appends Entry| TX_LOG
    REFUND -.->|Appends Entry| TX_LOG
```

---

## 5. Persistence & Query Optimization Strategy

### 5.1 Open-In-View Disabled (`open-in-view=false`)
By default, Spring Boot keeps the Hibernate session open throughout the HTTP request rendering phase, which can mask N+1 query problems and lead to connection pool starvation. 

In Rentify Backend, `spring.jpa.open-in-view` is strictly disabled:
- All database access occurs within `@Transactional` service boundaries.
- Controllers never touch lazy proxies directly.
- All relationships needed for API responses are loaded via `@EntityGraph(attributePaths = {...})` or explicit JPQL `JOIN FETCH` queries.

### 5.2 Dynamic Faceted Filtering
Search queries (`/api/v1/properties`) are compiled dynamically via Spring Data JPA `Specification<Property>`:
- Dynamic predicates: `cityId`, `districtId`, `minPrice`, `maxPrice`, `guestCount`, `bedroomCount`, `amenityIds`, `propertyType`.
- Subqueries for availability date filters: ensures properties with collisions are pruned before pagination limits are applied.
- Paginated results (`Page<PropertyResponseDto>`) ensure minimal memory footprint.

---

## 6. Observability, Security & Production Readiness

- **Exception Management**: Handled via `GlobalExceptionHandler` adhering to RFC 7807 (`ProblemDetail`). Standard HTTP status codes (400, 401, 403, 404, 409, 422, 500) with localized, structured JSON bodies.
- **Audit Logging**: Entities inherit from `CreatedAtEntity` or `AuditableEntity` using Spring Data JPA `@EntityListeners(AuditingEntityListener.class)` to maintain immutable creation and modification timestamps.
- **Container Isolation**: Multi-stage Docker container based on Eclipse Temurin JRE 17 headless distroless pattern to minimize attack surfaces.
