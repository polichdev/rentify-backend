# Security Policy

| Parameter | Policy Details |
|---|---|
| **Responsible Disclosure** | Coordinated vulnerability disclosure |
| **Response SLA** | Initial acknowledgment within 24 hours |
| **Resolution Target** | Critical vulnerabilities patched within 7 business days |

---

## 1. Supported Versions

Security updates and patches are actively maintained for the following versions:

| Version | Supported | Notes |
|:---:|:---:|---|
| `1.x.x` | :white_check_mark: | Current production release line |
| `< 1.0` | :x: | Deprecated pre-release versions |

---

## 2. Reporting a Vulnerability

If you discover a security vulnerability in Rentify Backend, please **do NOT report it via public GitHub issues**.

Instead, report it privately through one of the following channels:
1. **GitHub Security Advisory**: Use the [Private Vulnerability Reporting](https://github.com/polichdev/rentify-backend/security/advisories/new) feature on GitHub.
2. **Email**: Send encrypted details to `security@rentify.dev` (or the maintainer's contact email).

### What to Include in Your Report:
- A descriptive summary of the vulnerability (e.g. IDOR, Authentication Bypass, Injection, Concurrency Race Condition).
- Step-by-step reproduction instructions or a minimal Proof of Concept (PoC).
- Potential impact on users, wallets, or confidential data.
- Any suggested fixes or remediations.

---

## 3. Vulnerability Handling Procedure

1. **Acknowledgment**: Within 24 hours of report receipt.
2. **Triage & Reproduction**: Within 48 hours to confirm severity and assess impact using CVSS v3.1.
3. **Patch Development & Testing**: Tested hermetically against Testcontainers and dedicated test suites.
4. **Public Advisory**: Published concurrently with the patch release, giving credit to the reporter (unless anonymity is requested).
