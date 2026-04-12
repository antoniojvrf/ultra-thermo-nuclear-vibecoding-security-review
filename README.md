# 🛡️ VibeCoding Security Skill

> A comprehensive security skill for AI coding assistants — combining preventive guidance and post-hoc auditing for "vibe-coded" projects.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0-blue)](SKILL.md)
[![Checks](https://img.shields.io/badge/security%20checks-50-green)](#checklist-coverage)

## What Is This?

AI coding assistants (Copilot, Cursor, Claude, Codex) are incredibly productive — but they consistently introduce the same security vulnerabilities. This skill teaches your AI agent to **think like a security engineer**, both while writing code and when reviewing it.

This skill was synthesized from the best ideas across three public vibe-security skills ([LadyKerr](https://github.com/LadyKerr/Vibe-Security-Skill), [BehiSecc](https://github.com/BehiSecc/VibeSec-Skill), [raroque](https://github.com/raroque/vibe-security-skill)) and extended with new coverage areas.

## Two Operating Modes

| Mode | When | What It Does |
|------|------|-------------|
| 🛡️ **PREVENTIVE** | While writing code | Consults vulnerability knowledge base before generating code that touches auth, payments, DB, secrets |
| 🔍 **AUDIT** | On existing code | Runs a 50-point checklist, scores the project 0-100, produces a structured report with before/after diffs |

## Checklist Coverage

**50 security checks** across three severity tiers:

### 🚨 Critical (15) — Fix Before Deploy
Exposed secrets, auth bypass, SQL injection, XSS, IDOR, CORS wildcard, missing RLS, client-side prices, JWT alg:none, file upload bypass, SSRF, service_role key exposure, root containers, exposed .git

### 🔍 Standard (25) — Fix Within 1 Week  
CSRF, rate limiting, mass assignment, open redirect, path traversal, XXE, input validation, password hashing, session management, GraphQL introspection, prompt injection, AI usage caps, webhook verification, secure storage, deep links, OAuth/PKCE, WebSocket auth, race conditions, prototype pollution, dependency CVEs, error leaking, CSP, Server Actions validation, cookie prefixes

### 🚀 Production (10) — Before Go-Live
Security headers, source maps, debug endpoints, GDPR deletion, backup strategy, logging PII, env separation, CI/CD hardening, email security (SPF/DKIM/DMARC), attack surface docs

## Coverage Map

| Domain | Coverage |
|--------|---------|
| 🔐 Authentication (JWT, OAuth, PKCE, Sessions) | ✅ Deep |
| 💉 Injection (SQL, XSS, Command, XXE, Prototype Pollution) | ✅ Deep |
| 🔑 Secrets Management | ✅ Deep |
| 🗄️ Supabase RLS | ✅ Deep |
| 🔥 Firebase Security Rules | ✅ Deep |
| ▲ Next.js (Server Actions, Middleware, App Router) | ✅ Deep |
| 💳 Stripe Payments | ✅ Deep |
| 📱 React Native / Expo | ✅ Deep |
| 🤖 AI / LLM Integration | ✅ Deep |
| 🌐 GraphQL | ✅ Deep |
| 🐳 Docker / CI-CD | ✅ Deep |
| 🔒 Cryptography | ✅ Deep |
| ⚡ Race Conditions | ✅ Deep |
| 🏗️ CORS, CSP, Security Headers | ✅ Deep |
| 📋 GDPR Compliance | ✅ Deep |
| 📊 Logging / Observability | ✅ Deep |
| 📦 Supply Chain / Dependencies | ✅ Deep |
| 🔗 CSRF, SSRF, Open Redirect | ✅ Deep (with bypass tables) |
| 📂 File Upload Security | ✅ Deep (magic bytes table) |
| 🌍 Access Control (IDOR, Privilege Escalation) | ✅ Deep |

## File Structure

```
vibecoding-security/
├── SKILL.md                         # Main manifest — stack detection + routing
├── core/
│   ├── principles.md                # 7 security pillars + vibe-coding anti-patterns
│   ├── audit-process.md             # 50-point checklist routing table
│   └── output-format.md             # Report template with scoring
├── vulnerabilities/
│   ├── injection.md                 # SQLi, XSS, Command Injection, XXE, Prototype Pollution
│   ├── access-control.md            # IDOR, Privilege Escalation, Mass Assignment
│   ├── authentication.md            # JWT, OAuth/PKCE, Passwords, Sessions, WebSocket
│   ├── csrf-ssrf.md                 # CSRF, SSRF (12 bypasses), Open Redirect (11 bypasses)
│   ├── file-upload.md               # Magic bytes, Polyglot, ZIP Slip, Path Traversal
│   ├── cryptography.md              # AES-GCM, Hashing, Key Management, Nonce reuse
│   └── race-conditions.md           # TOCTOU, Double-spend, Idempotency, Distributed Locks
├── stack/
│   ├── supabase.md                  # RLS policies, service_role key, getUser vs getSession
│   ├── firebase.md                  # Firestore rules, Storage rules, ID token verification
│   ├── nextjs.md                    # Server Actions, Route Handlers, Middleware, env vars
│   ├── stripe-payments.md           # Price manipulation, Webhook sig verification
│   ├── react-native.md              # SecureStore, API proxy, Deep links, Expo env vars
│   ├── ai-llm.md                    # API key protection, Prompt injection, Usage caps
│   ├── graphql.md                   # Introspection, Depth limiting, Complexity, Batching
│   └── docker-cicd.md               # Non-root containers, Pipeline permissions, Action pinning
├── production/
│   ├── security-headers.md          # CSP with nonces, CORS, HSTS, complete header set
│   ├── deployment.md                # Source maps, Debug endpoints, .git, SPF/DKIM/DMARC
│   ├── compliance.md                # GDPR account deletion, Data export, Backup strategy
│   ├── observability.md             # PII sanitization, Log injection, Structured logging
│   └── dependency-audit.md          # Typosquatting, Supply chain, Lockfile, License
└── checklists/
    ├── critical-15.md               # 15 critical checks with regex search patterns
    ├── standard-25.md               # 25 standard checks with cross-references
    └── production-10.md             # 10 pre-deploy checks
```

## Installation

### Antigravity (Gemini AI)
The skill is read automatically from `~/.gemini/antigravity/skills/vibecoding-security/`.

```bash
# Clone into your Antigravity skills directory
git clone https://github.com/antoniojvrf/vibecoding-security \
  ~/.gemini/antigravity/skills/vibecoding-security
```

### Other Agents (Claude, Cursor, Copilot)
Copy the contents of `SKILL.md` into your agent's system prompt or rules file (`.cursorrules`, `CLAUDE.md`, `.github/copilot-instructions.md`). For the full modular experience, include the referenced files in your agent's context.

## Usage

The skill activates automatically when context matches security-related code. You can also trigger it explicitly:

```
"Run a vibe security audit on this project"
"Is this code safe to deploy?"
"Check my Supabase setup for vulnerabilities"
"Review this payment flow for security issues"
"Can someone hack this?"
```

## Example Output

```markdown
# 🛡️ VibeSec Security Report

**Score**: 64/100 — 🟡 Fair — needs attention
**Issues Found**: 5 (2 Critical, 2 High, 1 Medium)

## ⚡ Quick Wins (fixable in < 10 minutes)
| # | Issue             | Severity | Fix Time | One-liner                          |
|---|-------------------|----------|----------|------------------------------------|
| 1 | .env not ignored  | 10/10    | 1 min    | echo ".env*" >> .gitignore         |

## 🚨 Critical Issues

### C1. `lib/supabase.ts:3` — Service role key in client bundle
**Impact**: Anyone can read, modify, or delete every row in your database.

\`\`\`diff
- const supabase = createClient(url, process.env.NEXT_PUBLIC_SUPABASE_SERVICE_KEY!)
+ const supabase = createClient(url, process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!)
\`\`\`
> ⚠️ Rotate the key immediately in Supabase Dashboard → Settings → API.
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on adding new vulnerability patterns, stack modules, or improving existing checks.

## Credits

Built on the shoulders of:
- [LadyKerr/Vibe-Security-Skill](https://github.com/LadyKerr/Vibe-Security-Skill) — structured audit format and scoring
- [BehiSecc/VibeSec-Skill](https://github.com/BehiSecc/VibeSec-Skill) — offensive depth and bypass technique tables
- [raroque/vibe-security-skill](https://github.com/raroque/vibe-security-skill) — modular architecture and stack-specific modules

## License

[MIT](LICENSE) — Use freely, contribute back.
