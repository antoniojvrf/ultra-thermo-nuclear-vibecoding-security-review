# Standard Checks (16-40) — Fix Within 1 Week

---

## Check 16: CSRF Protection
**Severity: 7/10** | **Ref:** `vulnerabilities/csrf-ssrf.md`

Check: All state-changing endpoints (POST/PUT/DELETE) use CSRF tokens or SameSite=Strict cookies. JSON APIs validate Origin header.

---

## Check 17: Rate Limiting
**Severity: 7/10** | **Ref:** `stack/ai-llm.md`

Check: Auth endpoints (login, register, reset), AI/LLM endpoints, payment endpoints, and expensive operations have rate limits. Rate limit counters use atomic increment (Redis/DB, not in-memory).

---

## Check 18: Mass Assignment
**Severity: 7/10** | **Ref:** `vulnerabilities/access-control.md`

Search for: `Model.update(req.body)`, `Model.create(req.body)`, `Model.create({ ...req.body })` without field whitelisting. Sensitive fields (role, isAdmin, verified, balance) must never come from client.

---

## Check 19: Open Redirect
**Severity: 6/10** | **Ref:** `vulnerabilities/csrf-ssrf.md`

Search for: Redirects using user-controlled URL parameters (`redirect`, `return_to`, `next`, `callback`). Validate against allowlist or restrict to relative paths.

---

## Check 20: Path Traversal
**Severity: 8/10** | **Ref:** `vulnerabilities/file-upload.md`

Search for: User input in file paths without canonicalization. `fs.readFile(userInput)`, `path.join(base, userInput)` without checking result stays within base.

---

## Check 21: XXE in XML Parsers
**Severity: 7/10** | **Ref:** `vulnerabilities/injection.md`

Check: XML parsing has external entities disabled. File uploads accepting DOCX/XLSX (XML-based) process XML safely. Skip if no XML processing.

---

## Check 22: Input Validation Server-Side
**Severity: 6/10** | **Ref:** `core/principles.md`

Check: All API inputs validated with schema (Zod, joi, yup) on server side. Types, lengths, ranges, formats checked. Client-side validation is not the only check.

---

## Check 23: Password Hashing
**Severity: 8/10** | **Ref:** `vulnerabilities/authentication.md`

Check: Passwords hashed with Argon2id, bcrypt, or scrypt — never MD5, SHA1, or plain SHA256. Skip if using third-party auth (Clerk, Auth0, Supabase Auth).

---

## Check 24: Session Management
**Severity: 7/10** | **Ref:** `vulnerabilities/authentication.md`

Check: Cookies have HttpOnly, Secure, SameSite flags. Sessions regenerated on login. Idle/absolute timeouts configured. Sessions revoked on password change.

---

## Check 25: GraphQL Introspection
**Severity: 5/10** | **Ref:** `stack/graphql.md`

Check: Introspection disabled in production. Query depth and complexity limits configured. Skip if no GraphQL.

---

## Check 26: Prompt Injection
**Severity: 6/10** | **Ref:** `stack/ai-llm.md`

Check: User input not directly interpolated into system prompts. System prompts include guardrails against instruction override. Input length limited. Skip if no AI features.

---

## Check 27: AI Usage Caps
**Severity: 6/10** | **Ref:** `stack/ai-llm.md`

Check: Per-user rate limits and daily/monthly token/cost caps on AI endpoints. Usage tracked in database. Skip if no AI features.

---

## Check 28: Webhook Signature Verification
**Severity: 8/10** | **Ref:** `stack/stripe-payments.md`

Check: Incoming webhooks (Stripe, GitHub, etc.) verify signatures before processing. Raw body used for verification (not parsed JSON). Skip if no webhooks.

---

## Check 29: Mobile Secure Storage
**Severity: 8/10** | **Ref:** `stack/react-native.md`

Check: Auth tokens stored in SecureStore/Keychain, not AsyncStorage. No API keys in JavaScript bundle. Skip if no mobile app.

---

## Check 30: Deep Link Validation
**Severity: 6/10** | **Ref:** `stack/react-native.md`

Check: Deep link parameters validated server-side before acting on them. Skip if no mobile app.

---

## Check 31: OAuth State / PKCE
**Severity: 7/10** | **Ref:** `vulnerabilities/authentication.md`

Check: OAuth flows include `state` parameter for CSRF protection and PKCE for code interception protection. Skip if no OAuth.

---

## Check 32: WebSocket Auth Per-Message
**Severity: 6/10** | **Ref:** `vulnerabilities/authentication.md`

Check: WebSocket connections authenticate at handshake AND authorize each message/action. Skip if no WebSocket.

---

## Check 33: Race Conditions
**Severity: 8/10** | **Ref:** `vulnerabilities/race-conditions.md`

Check: Financial operations (balance, credits, inventory) use atomic DB transactions. No check-then-act patterns without atomicity. Idempotency keys for critical operations.

---

## Check 34: Prototype Pollution
**Severity: 6/10** | **Ref:** `vulnerabilities/injection.md`

Check: Custom deep merge functions block `__proto__`, `constructor`, `prototype` keys. Skip if using modern frameworks that handle this.

---

## Check 35: Dependency Typosquatting
**Severity: 5/10** | **Ref:** `production/dependency-audit.md`

Check: All dependencies legitimate (check npm download counts, publish dates, authors). No suspicious package names.

---

## Check 36: Known CVEs in Dependencies
**Severity: 7/10** | **Ref:** `production/dependency-audit.md`

Run: `npm audit` or equivalent. Check for critical/high severity vulnerabilities in dependency tree.

---

## Check 37: Error Messages Leaking Internals
**Severity: 5/10** | **Ref:** `core/principles.md`

Check: Error responses return generic messages. Stack traces, SQL errors, internal paths not exposed to users in production.

---

## Check 38: Content Security Policy
**Severity: 6/10** | **Ref:** `production/security-headers.md`

Check: CSP header configured. At minimum: `default-src 'self'`, `frame-ancestors 'none'`. No `unsafe-eval` for scripts.

---

## Check 39: Server Actions Validation (Next.js)
**Severity: 7/10** | **Ref:** `stack/nextjs.md`

Check: Server Actions validate input with Zod/joi, verify auth from session (not FormData), and whitelist updateable fields. Skip if not Next.js.

---

## Check 40: Cookie Prefixes
**Severity: 4/10** | **Ref:** `vulnerabilities/authentication.md`

Check: Session cookies use `__Host-` or `__Secure-` prefix for additional browser-enforced security.
