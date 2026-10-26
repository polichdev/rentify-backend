# Changelog

All notable changes to **Rentify Backend** will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Added
- Comprehensive Architecture & System Design documentation (`docs/ARCHITECTURE.md`).
- Entity-Relationship & Indexing specifications (`docs/DATABASE_SCHEMA.md`).
- Concurrency & Double-Booking prevention strategy (`docs/CONCURRENCY_AND_LOCKING.md`).
- STRIDE Threat Model & Security Architecture (`docs/THREAT_MODEL.md`).
- Failures, Circuit Breaking & Recovery documentation (`docs/FAILURES_AND_RECOVERY.md`).
- Testing Strategy and Testcontainers integration guidelines (`docs/TESTING_STRATEGY.md`).
- DevOps & Production Deployment matrix (`docs/DEPLOYMENT_AND_DEVOPS.md`).
- Architecture Decision Records ADR-001 through ADR-006 (`docs/adr/`).
- Standardized open-source governance (`CONTRIBUTING.md`, `SECURITY.md`, GitHub issue/PR templates).

---

## [1.0.0] - 2026-09-16

### Added
- **Core Platform REST API**: Initial production release.
- **Dual-Mode Authentication**: Bearer tokens and HTTP-only Secure Cookies with CSRF token handshake.
- **Google OAuth 2.0 Integration**: Single Sign-On and user profile provisioning.
- **Faceted Property Search**: Filtering by location, amenities, dates, guest counts, and `/map-pins` geo-clustering.
- **Smart Availability Engine**: Collision detection merging bookings and host manual calendar blocks.
- **End-to-End Booking Workflow**: Multi-stage state machine (`PENDING`, `CONFIRMED`, `REJECTED`, `CANCELLED`, `COMPLETED`).
- **Internal Wallet & Ledger**: Predefined top-up amounts, balance validation, and immutable transaction audit history.
- **Direct Messaging**: Contextual host-guest threads tied to property and booking records.
- **Verified Review Engine**: Multi-criteria rating calculation with dynamic score aggregation.
- **Media CDN Management**: Cloudinary SDK integration for multi-image uploads and transformations.
- **Test Suite**: Over 200 unit and integration tests powered by JUnit 5, Mockito, and Testcontainers.
