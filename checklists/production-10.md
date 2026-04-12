# Production Checks (41-50) — Before Go-Live

---

## Check 41: Security Headers Complete
**Severity: 6/10** | **Ref:** `production/security-headers.md`

Verify all headers present:
- [ ] `Strict-Transport-Security` (HSTS)
- [ ] `Content-Security-Policy`
- [ ] `X-Content-Type-Options: nosniff`
- [ ] `X-Frame-Options: DENY`
- [ ] `Referrer-Policy`
- [ ] `Permissions-Policy`

---

## Check 42: Source Maps Disabled
**Severity: 5/10** | **Ref:** `production/deployment.md`

Check: `productionBrowserSourceMaps: false` (Next.js), `build.sourcemap: false` (Vite). No `.map` files served in production.

---

## Check 43: Debug Endpoints Removed
**Severity: 7/10** | **Ref:** `production/deployment.md`

Search for: `/api/debug`, `/api/test`, `/api/internal`, routes that expose `process.env`, database configs, or system info. Must be removed or gated behind auth + admin role in production.

---

## Check 44: GDPR / Account Deletion
**Severity: 5/10** | **Ref:** `production/compliance.md`

Check: Account deletion flow removes/anonymizes all user data, cancels subscriptions, invalidates sessions, and logs the action. Skip if not handling EU user data.

---

## Check 45: Backup Strategy
**Severity: 5/10** | **Ref:** `production/compliance.md`

Check: Automated database backups configured. Backup restore tested. Retention period defined. Backups encrypted and stored in separate region.

---

## Check 46: Logging Without PII
**Severity: 6/10** | **Ref:** `production/observability.md`

Check: Logs don't contain passwords, tokens, full card numbers, SSNs. Request bodies sanitized before logging. Log injection prevented (newlines stripped from user input in log messages).

---

## Check 47: Environment Separation
**Severity: 6/10** | **Ref:** `production/deployment.md`

Check: Separate `.env` files per environment. Production secrets not in repository. `.env` in `.gitignore`. Only `.env.example` committed.

---

## Check 48: CI/CD Pipeline Hardened
**Severity: 6/10** | **Ref:** `stack/docker-cicd.md`

Check:
- [ ] GitHub Actions use minimal `permissions`
- [ ] Actions pinned to commit SHA
- [ ] Secrets not echoed in logs
- [ ] `npm ci` used (not `npm install`)
- [ ] `npm audit` in pipeline
- [ ] No `pull_request_target` with PR code checkout

---

## Check 49: Email Security
**Severity: 4/10** | **Ref:** `production/deployment.md`

Check: SPF, DKIM, and DMARC DNS records configured for email-sending domains. Skip if not sending emails.

---

## Check 50: Attack Surface Documented
**Severity: 3/10** | **Ref:** `production/deployment.md`

Check: All public endpoints, webhooks, and external integrations documented with auth requirements and rate limits. Helps future security reviews.
