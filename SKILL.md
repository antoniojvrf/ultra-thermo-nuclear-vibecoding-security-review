---
name: ultra-thermo-nuclear-vibecoding-security-review
description: >
  Comprehensive security skill for AI-assisted ("vibe-coded") projects. Operates in two modes:
  (1) AUDIT — runs a 50-point security checklist with quantitative scoring against an existing codebase,
  (2) PREVENTIVE — guides secure code generation by consulting vulnerability knowledge bases before writing.
  Covers OWASP Top 10, cloud-native security (Supabase RLS, Firebase Rules), payment security (Stripe),
  mobile security (React Native/Expo), AI/LLM security (prompt injection, usage caps), GraphQL,
  Docker/CI-CD, and production readiness (GDPR, logging, headers, dependency auditing).
  Activate when: writing or reviewing code involving auth, payments, database access, API keys, secrets,
  user data, file uploads, or deployment. Also activate when user says "security audit", "vulnerability check",
  "is this safe?", "check my code", "audit this", "review for vulnerabilities", "can someone hack this?",
  "vibe security", or any mention of security concerns.
license: MIT
metadata:
  version: "1.0"
  based-on: "LadyKerr/Vibe-Security-Skill, BehiSecc/VibeSec-Skill, raroque/vibe-security-skill"
---

# Ultra Thermo Nuclear VibeCoding Security Review — Agent Skill

## Core Principle
**Never trust the client.** Every price, user ID, role, subscription status, feature flag, and rate limit counter must be validated or enforced server-side. If it exists only in the browser, mobile bundle, or request body, an attacker controls it.

Additional principles — see `core/principles.md`:
- Defense in depth: never rely on a single security control
- Fail securely: when something breaks, deny access (fail closed)
- Least privilege: grant minimum permissions necessary
- Input validation: never trust user input — validate everything server-side
- Output encoding: encode data for the context it renders in

---

## Operating Modes

### Mode 1: AUDIT (post-hoc analysis)
Triggered by: "run a security audit", "check this for vulnerabilities", "vibe security audit", "is this safe?", or any explicit request to review existing code.

**Process:**
1. Detect tech stack (see Step 1 below)
2. Run the 50-point checklist — see `core/audit-process.md`
3. Calculate score and compile report — see `core/output-format.md`
4. Load relevant vulnerability references and stack modules as needed

### Mode 2: PREVENTIVE (during code generation)
Triggered by: writing or reviewing code that touches auth, payments, database access, API keys, secrets, user data, or file uploads — even if user does not mention security.

**Process:**
1. Identify which security domain applies to the code being written
2. Consult the relevant reference file(s) BEFORE generating code
3. Apply secure patterns from the start
4. Add inline comments explaining security decisions when non-obvious
5. Flag any security trade-offs to the user

---

## Step 1: Detect Tech Stack

Identify the project's technologies by examining these files:

| Detect          | Look for                                                               |
|:----------------|:-----------------------------------------------------------------------|
| Framework       | `next.config.*`, `nuxt.config.*`, `astro.config.*`, `vite.config.*`, `angular.json`, `svelte.config.*` |
| Package manager | `package.json`, `requirements.txt`, `go.mod`, `Gemfile`, `Cargo.toml`, `pubspec.yaml` |
| Database        | `prisma/`, `drizzle.config.*`, `supabase/`, `firebase.json`, `convex/` |
| Auth            | Imports: `@clerk`, `next-auth`, `@supabase/auth`, `@auth0`, `lucia`, `passport`, `firebase/auth` |
| Storage         | Imports: `@aws-sdk/client-s3`, `@supabase/storage`, `@google-cloud/storage` |
| AI APIs         | Imports: `openai`, `@anthropic-ai/sdk`, `@google/generative-ai`, `langchain`, `@ai-sdk` |
| Payment         | Imports: `stripe`, `@paddle/paddle-node`, `@lemonsqueezy`              |
| Email           | Imports: `resend`, `@sendgrid/mail`, `nodemailer`                      |
| GraphQL         | `schema.graphql`, imports: `@apollo/server`, `graphql-yoga`, `type-graphql` |
| Mobile          | `app.json` (Expo), `react-native.config.js`, `capacitor.config.*`     |
| Deployment      | `vercel.json`, `netlify.toml`, `fly.toml`, `Dockerfile`, `docker-compose.*`, `.github/workflows/` |
| Edge Runtime    | `wrangler.toml`, edge runtime directives in source                     |

**Skip checks that don't apply to the detected stack.**

---

## Step 2: Run Audit

Load the relevant checklist files and execute:

```
Audit Progress:
- [ ] Step 1: Detect tech stack
- [ ] Step 2a: Run critical checks (1-15) — see checklists/critical-15.md
- [ ] Step 2b: Run standard checks (16-40) — see checklists/standard-25.md
- [ ] Step 2c: Run production checks (41-50) — see checklists/production-10.md
- [ ] Step 3: Calculate score and compile report — see core/output-format.md
```

**Run critical checks first** — read `checklists/critical-15.md` and execute all checks before proceeding.

For each check, consult the relevant reference files:
- Vulnerability details → `vulnerabilities/*.md`
- Stack-specific patterns → `stack/*.md`
- Production readiness → `production/*.md`

### Severity Scale

| Score  | Level         | Action Required          |
|:-------|:--------------|:-------------------------|
| 10/10  | Critical      | Fix before deploying     |
| 8-9/10 | High          | Fix within 24 hours      |
| 6-7/10 | Medium        | Fix within 1 week        |
| 4-5/10 | Low           | Fix when convenient      |
| 1-3/10 | Informational | Consider addressing      |

**Project score** = 100 minus sum of severity scores for all issues found (minimum 0).

| Score   | Rating                   |
|:--------|:-------------------------|
| 90-100  | ✅ Excellent              |
| 70-89   | ⚠️ Good — minor issues    |
| 50-69   | 🟡 Fair — needs attention |
| 30-49   | 🟠 Poor — significant risk|
| 0-29    | 🔴 Critical — do not deploy|

---

## Step 3: Output Report

Use the format defined in `core/output-format.md`. Key requirements:
- Reference exact files and line numbers
- Show which pattern matched for each finding
- Provide before/after code diffs for every issue
- Include Quick Wins section (fixable in < 10 minutes)
- End with prioritized summary

---

## Reference Index

### Core
- `core/principles.md` — Security principles and defense-in-depth strategies
- `core/audit-process.md` — 50-point checklist routing and execution order
- `core/output-format.md` — Report template with scoring

### Vulnerability Knowledge Base
- `vulnerabilities/injection.md` — SQL injection, XSS, command injection, LDAP injection
- `vulnerabilities/access-control.md` — IDOR, privilege escalation, mass assignment
- `vulnerabilities/authentication.md` — JWT, OAuth 2.0, PKCE, session management, password security
- `vulnerabilities/csrf-ssrf.md` — CSRF, SSRF, open redirect (with bypass technique tables)
- `vulnerabilities/file-upload.md` — Magic bytes, polyglot files, ZIP slip, SVG XSS
- `vulnerabilities/cryptography.md` — Encryption, hashing, key management, nonce reuse
- `vulnerabilities/race-conditions.md` — TOCTOU, double-spend, mutex patterns

### Stack-Specific Modules
- `stack/supabase.md` — RLS policies, auth, storage, edge functions
- `stack/firebase.md` — Security Rules, Firestore, auth
- `stack/nextjs.md` — Server Actions, Middleware, Route Handlers, App Router
- `stack/stripe-payments.md` — Webhooks, price validation, subscription flows
- `stack/react-native.md` — Secure storage, API proxy, deep links
- `stack/ai-llm.md` — API key protection, prompt injection, usage caps, output sanitization
- `stack/graphql.md` — Introspection, depth limiting, batching, persisted queries
- `stack/docker-cicd.md` — Container security, pipeline hardening

### Production Readiness
- `production/security-headers.md` — CSP, HSTS, X-Frame-Options, Permissions-Policy
- `production/deployment.md` — Source maps, env separation, debug endpoints
- `production/compliance.md` — GDPR, CCPA, account deletion, data retention
- `production/observability.md` — Logging security, PII sanitization, monitoring
- `production/dependency-audit.md` — Ghost packages, typosquatting, supply chain attacks

### Executable Checklists
- `checklists/critical-15.md` — 15 critical checks (fix before deploy)
- `checklists/standard-25.md` — 25 standard checks (fix within 1 week)
- `checklists/production-10.md` — 10 production readiness checks (before go-live)

---

## When Generating Code

These rules apply proactively. Before writing code that touches auth, payments, database access, API keys, or user data, consult the relevant reference file to avoid introducing vulnerabilities in the first place. **Prevention is always better than detection.**

When unsure, choose the more restrictive/secure option and document the security consideration in comments.
