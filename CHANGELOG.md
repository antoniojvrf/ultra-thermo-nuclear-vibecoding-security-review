# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [1.0.0] - 2026-04-12

### Added
- **SKILL.md** — Main skill manifest with dual-mode operation (PREVENTIVE + AUDIT), automatic tech stack detection, and reference index
- **core/principles.md** — 7 security pillars and catalog of AI-introduced anti-patterns
- **core/audit-process.md** — 50-point checklist routing table across three severity tiers
- **core/output-format.md** — Standardized report template with score 0-100, before/after diffs, Quick Wins section
- **vulnerabilities/injection.md** — SQL injection, XSS (with indirect sources), Command injection, XXE (multi-language), Prototype pollution
- **vulnerabilities/access-control.md** — IDOR, Privilege escalation, Mass assignment
- **vulnerabilities/authentication.md** — JWT (alg:none, algorithm confusion), OAuth 2.0 / PKCE, Password hashing, Session management, Cookie prefixes, WebSocket authentication
- **vulnerabilities/csrf-ssrf.md** — CSRF, SSRF with 12 IP bypass techniques and DNS rebinding, Open redirect with 11 bypass techniques
- **vulnerabilities/file-upload.md** — Magic bytes reference table, polyglot files, ZIP slip, path traversal, secure upload handler
- **vulnerabilities/cryptography.md** — AES-GCM encryption, hashing algorithms, key management, nonce reuse, timing-safe comparison
- **vulnerabilities/race-conditions.md** — TOCTOU, double-spend, idempotency keys, distributed locks (Redis)
- **stack/supabase.md** — RLS policy patterns (dangerous vs secure), service_role key exposure, getUser vs getSession
- **stack/firebase.md** — Firestore security rules, Storage rules, ID token server-side verification
- **stack/nextjs.md** — Server Actions (Zod validation + auth), Route Handlers, Middleware, env var scoping, security headers config
- **stack/stripe-payments.md** — Client-side price manipulation, webhook signature verification, subscription status validation
- **stack/react-native.md** — SecureStore vs AsyncStorage, API key proxying, Expo env vars, deep link validation
- **stack/ai-llm.md** — API key server-side protection, rate limiting + usage caps, prompt injection prevention, output sanitization
- **stack/graphql.md** — Introspection, depth limiting, complexity analysis, batching prevention, resolver authorization
- **stack/docker-cicd.md** — Non-root containers, secrets in Dockerfile, image pinning, GitHub Actions permissions, action SHA pinning
- **production/security-headers.md** — Complete header set, CSP with nonces (Next.js example), CORS allowlist configuration
- **production/deployment.md** — Source maps, debug endpoints, env separation, .git exposure, email security (SPF/DKIM/DMARC)
- **production/compliance.md** — GDPR complete account deletion flow, data export, consent management, backup strategy
- **production/observability.md** — PII sanitization in logs, log injection prevention, structured logging with redaction, security event monitoring
- **production/dependency-audit.md** — Typosquatting detection, CVE scanning, supply chain attacks, lockfile integrity, license compliance
- **checklists/critical-15.md** — 15 critical checks with regex search patterns and pass criteria
- **checklists/standard-25.md** — 25 standard checks with cross-references to module files
- **checklists/production-10.md** — 10 production readiness checks

### Sources
- Synthesized from [LadyKerr/Vibe-Security-Skill](https://github.com/LadyKerr/Vibe-Security-Skill), [BehiSecc/VibeSec-Skill](https://github.com/BehiSecc/VibeSec-Skill), and [raroque/vibe-security-skill](https://github.com/raroque/vibe-security-skill)
- Extended with new coverage: Cryptography, Race conditions, Docker/CI-CD, OAuth/PKCE, WebSocket auth, Logging security, Cookie prefixes, Prototype pollution
