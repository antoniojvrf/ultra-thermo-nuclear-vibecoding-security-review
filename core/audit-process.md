# Audit Process — 50-Point Security Checklist

## Execution Order

Run checks in this order: **Critical → Standard → Production**. For each check, load the referenced module file only if the codebase uses that technology. Skip irrelevant checks.

---

## Phase A: CRITICAL (Checks 1-15) — Fix Before Deploy

See `checklists/critical-15.md` for detailed instructions.

| # | Check | Reference File |
|---|-------|---------------|
| 1 | Secrets/API keys hardcoded in source | `vulnerabilities/injection.md`, `stack/*.md` |
| 2 | Env vars with public prefix exposing secrets | `stack/nextjs.md`, `stack/react-native.md` |
| 3 | Auth bypass on routes/endpoints | `vulnerabilities/authentication.md` |
| 4 | SQL Injection (missing parameterized queries) | `vulnerabilities/injection.md` |
| 5 | XSS (unsanitized input/output) | `vulnerabilities/injection.md` |
| 6 | IDOR (accessing other users' resources) | `vulnerabilities/access-control.md` |
| 7 | CORS wildcard with credentials | `production/security-headers.md` |
| 8 | Database without RLS/Security Rules | `stack/supabase.md`, `stack/firebase.md` |
| 9 | Client-controlled price in payments | `stack/stripe-payments.md` |
| 10 | JWT alg:none / weak secret | `vulnerabilities/authentication.md` |
| 11 | File upload without validation | `vulnerabilities/file-upload.md` |
| 12 | SSRF in webhooks/URL fetching | `vulnerabilities/csrf-ssrf.md` |
| 13 | Service role key in client bundle | `stack/supabase.md` |
| 14 | Container running as root | `stack/docker-cicd.md` |
| 15 | Exposed .git directory in production | `production/deployment.md` |

---

## Phase B: STANDARD (Checks 16-40) — Fix Within 1 Week

See `checklists/standard-25.md` for detailed instructions.

| # | Check | Reference File |
|---|-------|---------------|
| 16 | CSRF protection on state-changing endpoints | `vulnerabilities/csrf-ssrf.md` |
| 17 | Rate limiting on auth/AI/payment endpoints | `stack/ai-llm.md`, `vulnerabilities/authentication.md` |
| 18 | Mass assignment (field whitelist) | `vulnerabilities/access-control.md` |
| 19 | Open redirect in URL parameters | `vulnerabilities/csrf-ssrf.md` |
| 20 | Path traversal in file handling | `vulnerabilities/file-upload.md` |
| 21 | XXE in XML parsers | `vulnerabilities/injection.md` |
| 22 | Input validation server-side | `core/principles.md` |
| 23 | Password hashing (Argon2id/bcrypt) | `vulnerabilities/authentication.md` |
| 24 | Session management (cookie flags) | `vulnerabilities/authentication.md` |
| 25 | GraphQL introspection in production | `stack/graphql.md` |
| 26 | AI prompt injection vectors | `stack/ai-llm.md` |
| 27 | AI usage caps / cost controls | `stack/ai-llm.md` |
| 28 | Webhook signature verification | `stack/stripe-payments.md` |
| 29 | Mobile secure storage (Keychain/Keystore) | `stack/react-native.md` |
| 30 | Deep link validation | `stack/react-native.md` |
| 31 | OAuth state parameter / PKCE | `vulnerabilities/authentication.md` |
| 32 | WebSocket authentication per-message | `vulnerabilities/authentication.md` |
| 33 | Race conditions in financial ops | `vulnerabilities/race-conditions.md` |
| 34 | Prototype pollution in deep merge | `vulnerabilities/injection.md` |
| 35 | Dependency typosquatting / ghost packages | `production/dependency-audit.md` |
| 36 | Known CVEs in dependencies | `production/dependency-audit.md` |
| 37 | Error messages leaking internals | `core/principles.md` |
| 38 | Content Security Policy configured | `production/security-headers.md` |
| 39 | Server Actions without validation (Next.js) | `stack/nextjs.md` |
| 40 | Cookie security (__Host-/__Secure- prefix) | `vulnerabilities/authentication.md` |

---

## Phase C: PRODUCTION (Checks 41-50) — Before Go-Live

See `checklists/production-10.md` for detailed instructions.

| # | Check | Reference File |
|---|-------|---------------|
| 41 | Security headers complete | `production/security-headers.md` |
| 42 | Source maps disabled in production | `production/deployment.md` |
| 43 | Debug endpoints removed | `production/deployment.md` |
| 44 | GDPR compliance / account deletion | `production/compliance.md` |
| 45 | Backup strategy documented | `production/compliance.md` |
| 46 | Logging without PII / log injection | `production/observability.md` |
| 47 | Environment separation (dev/staging/prod) | `production/deployment.md` |
| 48 | CI/CD pipeline hardened | `stack/docker-cicd.md` |
| 49 | Email security (SPF/DKIM/DMARC) | `production/deployment.md` |
| 50 | Attack surface documented | `production/deployment.md` |

---

## Score Calculation

```
project_score = 100 - sum(severity_of_each_issue)
project_score = max(project_score, 0)  // minimum 0
```

Interpret with the rating table in SKILL.md.
