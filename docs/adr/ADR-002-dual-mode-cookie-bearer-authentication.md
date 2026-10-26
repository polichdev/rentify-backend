# ADR-002: Dual-Mode Authentication (Secure Cookies vs Bearer Tokens)

| Metadata | Details |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-09-18 |
| **Deciders** | Engineering Architecture Team |
| **Tags** | Security, Auth, SPA, Mobile, CSRF |

---

## Context & Problem Statement
Rentify serves two distinct client ecosystems:
1. **Single Page Applications (SPA - React/Vue)**: Storing JWT tokens in browser `localStorage` or `sessionStorage` exposes tokens to Cross-Site Scripting (XSS) exfiltration attacks.
2. **Native Mobile Applications (iOS/Android)**: Mobile clients do not have browser cookie engines or CSRF vulnerabilities, but require standard `Authorization: Bearer <token>` headers.

Supporting only one model compromises either web security (forcing SPAs to store tokens in LocalStorage) or mobile developer experience (forcing mobile clients to manage Cookie Jar sessions).

## Decision Outcome
We implemented a **Configurable Dual-Mode Authentication Strategy** governed by `AUTH_STRATEGY` (`cookie` or `bearer`) and client headers:
- **Cookie Mode (for SPAs)**:
  - Tokens are transmitted via `HttpOnly`, `Secure`, `SameSite=Lax` cookies named `jwt_token`.
  - Mutations (`POST`, `PUT`, `DELETE`, `PATCH`) require a matching `X-CSRF-Token` header derived from a readable CSRF cookie set by the server during login.
- **Bearer Mode (for Mobile & External APIs)**:
  - Tokens are passed directly in HTTP headers (`Authorization: Bearer <token>`).
  - CSRF checks are bypassed because native apps are not vulnerable to browser ambient credentials.

## Consequences
### Positive
- Browser clients are fully immune to JavaScript token exfiltration via XSS.
- Mobile clients enjoy frictionless standard REST semantics.
- Clean architectural encapsulation inside `AuthCookieService` and `JwtAuthenticationFilter`.

### Negative / Trade-offs
- Requires CORS configurations with `allowCredentials = true` in web mode.
- Additional handshake step for frontend web clients to read and send the CSRF token on mutating requests.
