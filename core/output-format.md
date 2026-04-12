# Output Format — Security Audit Report

Use this exact structure when producing audit reports.

---

```markdown
# 🛡️ VibeSec Security Report

## Tech Stack Detected
- **Framework**: [detected or N/A]
- **Database**: [detected or N/A]
- **Auth**: [detected or N/A]
- **Payments**: [detected or N/A]
- **AI/LLM**: [detected or N/A]
- **Mobile**: [detected or N/A]
- **GraphQL**: [detected or N/A]
- **Deployment**: [detected or N/A]

## Summary
[1-2 sentence executive summary of overall security posture]

**Score**: [X/100] — [Rating Emoji] [Rating Label]
**Checks Run**: [X/50] ([Y] not applicable to this stack)
**Issues Found**: [total] ([a] Critical, [b] High, [c] Medium, [d] Low)

---

## ⚡ Quick Wins (fixable in < 10 minutes)
| # | Issue | Severity | Fix Time | One-liner |
|---|-------|----------|----------|-----------|
| 1 | [issue description] | [X/10] | [est. time] | [one-line fix or command] |

---

## 🚨 Critical Issues (Severity 10/10)

### C[n]. `[file:line]` — [Vulnerability Name]
**Vulnerability**: [Category]
**Impact**: [Concrete description of what an attacker could do]
**Exploitability**: [Trivial / Moderate / Complex] ([estimated time])

```diff
- [vulnerable code]
+ [fixed code]
```

> ⚠️ **Immediate action**: [specific step to take RIGHT NOW, e.g., rotate keys]

---

## 🔴 High Issues (Severity 8-9/10)

### H[n]. `[file:line]` — [Vulnerability Name]
**Vulnerability**: [Category]
**Impact**: [What an attacker could do]

```diff
- [before]
+ [after]
```

---

## 🟠 Medium Issues (Severity 6-7/10)

### M[n]. `[file:line]` — [Vulnerability Name]
**Vulnerability**: [Category]
**Impact**: [What an attacker could do]

```diff
- [before]
+ [after]
```

---

## 🟡 Low Issues (Severity 4-5/10)

### L[n]. `[file:line]` — [Issue]
[Brief description and fix]

---

## ℹ️ Informational (Severity 1-3/10)

### I[n]. [Issue]
[Brief description and recommendation]

---

## ✅ Passed Checks
- [x] #[n] [Check Name] — [brief reason it passed]

## ⏭️ Not Applicable
- #[n] [Check Name] — [reason skipped, e.g., "No mobile app detected"]

---

## 📋 Prioritized Summary
1. **[Issue name] (Critical):** [one-line summary]. [Immediate action].
2. **[Issue name] (High):** [one-line summary]. [Fix within 24h].
3. **[Issue name] (Medium):** [one-line summary]. [Fix within 1 week].
```

---

## Rules for Report Generation

1. **Reference exact files and line numbers** — never use vague references like "in your auth code"
2. **Show which pattern matched** — explain what triggered the finding
3. **Provide before/after diffs** for EVERY issue, not just critical ones
4. **Concrete impact descriptions** — "an attacker could read all user data" not "this could be a security risk"
5. **Quick Wins first** — items fixable in under 10 minutes go at the top
6. **Skip areas with no issues** — don't pad the report with passing checks inline
7. **Passed checks go at the bottom** — show what was verified as secure
8. **Prioritized summary at the end** — numbered list ordered by severity for action planning
