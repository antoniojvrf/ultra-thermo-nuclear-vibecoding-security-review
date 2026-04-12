# Critical Checks (1-15) — Fix Before Deploy

Run these checks first. Each critical issue warrants severity 8-10/10.

---

## Check 1: Secrets/API Keys in Source Code
**Severity: 10/10**

**Search for:**
```
# Regex patterns to scan
/(['"])(sk_live_|sk_test_|pk_live_|rk_live_|whsec_)[a-zA-Z0-9]+\1/
/(['"])(AKIA[A-Z0-9]{16})\1/                    # AWS Access Key
/(['"])(ghp_|gho_|ghu_|ghs_|ghr_)[a-zA-Z0-9]+\1/ # GitHub tokens
/(api[_-]?key|apikey|secret[_-]?key|password|token)\s*[:=]\s*['"][^'"]{8,}['"]/i
/-----BEGIN (RSA |EC )?PRIVATE KEY-----/
```

**Check files:** all source files, `.env` files committed to git, config files, README

**Pass criteria:** No secrets found in source code; `.env` in `.gitignore`

---

## Check 2: Public Env Var Prefix Exposing Secrets
**Severity: 10/10**

**Search for:**
```
NEXT_PUBLIC_.*SECRET
NEXT_PUBLIC_.*SERVICE_ROLE
NEXT_PUBLIC_.*PRIVATE
VITE_.*SECRET
VITE_.*SERVICE_ROLE
EXPO_PUBLIC_.*SECRET
REACT_APP_.*SECRET
```

**Check files:** `.env*`, `next.config.*`, `vite.config.*`, `app.json`

**Pass criteria:** No secret values prefixed with public env var names

---

## Check 3: Auth Bypass on Routes/Endpoints
**Severity: 10/10**

**Check:** Look for API routes and server endpoints that handle sensitive data without authentication middleware.

**Patterns:**
- Route handlers missing `auth()`, `getSession()`, `getUser()`, or auth middleware
- `app.get('/api/users', ...)` without auth check
- Server Actions without `await auth()`
- Admin endpoints accessible without role verification

**Pass criteria:** All data-access endpoints verify authentication

---

## Check 4: SQL Injection
**Severity: 10/10**

**Search for:**
```javascript
// String concatenation in queries
`SELECT * FROM ${table} WHERE id = ${id}`
`SELECT * FROM users WHERE name = '${name}'`
db.query(`... ${userInput} ...`)
$queryRawUnsafe(...)
sequelize.query(`... ${userInput} ...`)
connection.query(`... ${req.body...} ...`)
```

**Pass criteria:** All queries use parameterized statements or ORM with auto-parameterization

---

## Check 5: XSS (Cross-Site Scripting)
**Severity: 9/10**

**Search for:**
```javascript
dangerouslySetInnerHTML={{ __html: userContent }}
v-html="userContent"
innerHTML = userInput
document.write(userInput)
$(element).html(userInput)
```

**Check:** User input rendered without encoding in HTML, JS, URL, or CSS contexts

**Pass criteria:** All user-controlled output is encoded or sanitized (DOMPurify for HTML)

---

## Check 6: IDOR (Insecure Direct Object Reference)
**Severity: 9/10**

**Search for:** Data access using IDs from request without ownership verification:
```javascript
db.findUnique({ where: { id: req.params.id } })    // no userId check
db.findFirst({ where: { id: req.body.id } })        // no ownership
db.delete({ where: { id: req.query.id } })           // no ownership
```

**Pass criteria:** All data access includes `userId` / ownership check in the query

---

## Check 7: CORS Wildcard with Credentials
**Severity: 9/10**

**Search for:**
```javascript
cors({ origin: '*', credentials: true })
Access-Control-Allow-Origin: *  // with credentials
cors({ origin: true })          // reflects any origin
```

**Pass criteria:** CORS uses explicit origin allowlist; no wildcard with credentials

---

## Check 8: Database Without RLS/Security Rules
**Severity: 10/10**

**Check for Supabase:**
- Tables without RLS enabled
- RLS policies using `USING (true)` or `WITH CHECK (true)`
- Service role key used client-side

**Check for Firebase:**
- Firestore rules with `allow read, write: if true`
- Wildcard `{document=**}` with permissive rules
- No `request.auth.uid` checks

**Pass criteria:** All tables with user data have proper RLS/rules scoped by user

---

## Check 9: Client-Controlled Price in Payments
**Severity: 10/10**

**Search for:**
```javascript
price: req.body.price
unit_amount: req.body.amount
amount: formData.get('price')
```

**Pass criteria:** All prices/amounts looked up server-side from database or Stripe Price IDs

---

## Check 10: JWT alg:none / Weak Secret
**Severity: 10/10**

**Search for:**
```javascript
jwt.decode(token)               // decode without verify!
jwt.verify(token, secret)       // no algorithm pinning
jwt.sign(payload, 'secret123')  // weak secret
jwt.sign(payload, '')           // empty secret
```

**Pass criteria:** `jwt.verify()` with explicit `{ algorithms: ['HS256'] }` and 256+ bit random secret

---

## Check 11: File Upload Without Validation
**Severity: 8/10**

**Check:** File upload endpoints that accept files without:
- Extension allowlist
- Magic bytes verification
- File size limits
- Filename sanitization

**Pass criteria:** Uploads validate extension + magic bytes + size; filenames replaced with UUIDs

---

## Check 12: SSRF in URL Fetching
**Severity: 9/10**

**Search for:** Server-side code that fetches user-provided URLs:
```javascript
fetch(req.body.url)
axios.get(userProvidedUrl)
got(webhookUrl)
```

**Pass criteria:** URL fetching validates scheme (http/https only), resolves DNS to check for internal IPs, blocks cloud metadata endpoints

---

## Check 13: Service Role Key in Client Bundle
**Severity: 10/10**

**Search for:**
```
NEXT_PUBLIC_SUPABASE_SERVICE
process.env.NEXT_PUBLIC_.*SERVICE
createClient(url, serviceRoleKey) // in client-side code
```

**Pass criteria:** Service role keys exist only in server-side code (no public prefix)

---

## Check 14: Container Running as Root
**Severity: 7/10**

**Check:** Dockerfile without `USER` directive (defaults to root)

**Pass criteria:** Dockerfile includes non-root `USER` directive after setup

---

## Check 15: Exposed .git Directory
**Severity: 8/10**

**Check:** Web server configuration allows access to `/.git/`

**Pass criteria:** `.git` directory blocked by server config or not deployed
