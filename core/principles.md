# Security Principles

## The 7 Pillars

### 1. Never Trust the Client
Every value from the browser, mobile app, or API request body can be manipulated. Prices, user IDs, roles, subscription status, feature flags, rate limit counters — if it's not validated server-side, an attacker controls it.

### 2. Defense in Depth
Never rely on a single security control. Layer defenses so that a failure in one does not compromise the system:
- Input validation + parameterized queries + least-privilege DB user
- CSRF tokens + SameSite cookies + Origin header validation
- Auth middleware + per-resource ownership check + audit logging

### 3. Fail Securely (Fail Closed)
When something breaks, deny access by default:
```javascript
// WRONG — fails open
let isAuthorized = false;
try { isAuthorized = await checkAuth(req); } catch (e) { /* swallow */ }
if (isAuthorized) { /* proceed */ } else { /* still proceeds on error */ }

// CORRECT — fails closed
try {
  const user = await requireAuth(req); // throws on failure
  return handleRequest(user);
} catch (e) {
  return new Response("Unauthorized", { status: 401 });
}
```

### 4. Least Privilege
Grant the minimum permissions necessary:
- Database users should only SELECT/INSERT/UPDATE what they need, never superuser
- API keys should be scoped to specific operations
- Cloud IAM roles should use fine-grained policies, not `*` wildcards
- File system permissions should be restrictive (no 777)

### 5. Input Validation (Server-Side)
Never trust user input. Client-side validation is a UX convenience, not a security control:
- Validate data types, lengths, ranges, and formats server-side
- Use allowlists over denylists when possible
- Reject unexpected input rather than trying to sanitize it
- Validate at the boundary (API layer) before business logic

### 6. Output Encoding
Encode data appropriately for the context it's rendered in:
- HTML context → HTML entity encoding (`<` → `&lt;`)
- JavaScript context → JavaScript escaping
- URL context → URL encoding (`%20`, etc.)
- CSS context → CSS escaping
- SQL context → parameterized queries (not encoding)
- Use your framework's built-in escaping (React JSX, Vue `{{ }}`, Angular templates)

### 7. Secure Defaults
Security should be the default state, requiring explicit opt-out rather than opt-in:
- New routes should require authentication by default
- Database tables should deny access by default (then add RLS policies)
- CORS should be restrictive by default
- Cookies should be `Secure; HttpOnly; SameSite=Strict` by default
- Content-Security-Policy should be restrictive by default

---

## Anti-Patterns Common in Vibe-Coded Apps

These are the patterns AI assistants consistently get wrong:

| Anti-Pattern | Why It Happens | Correct Pattern |
|-------------|---------------|-----------------|
| Hardcoded API keys in source | AI copies from examples/docs | Use environment variables, never commit `.env` |
| `NEXT_PUBLIC_` for secret keys | AI prefixes everything for convenience | Secret keys must stay server-side only |
| `RLS USING (true)` in Supabase | AI enables RLS but with no real policy | Write actual policies checking `auth.uid()` |
| Client-side price in checkout | AI passes `req.body.price` to Stripe | Look up prices server-side from database |
| `jwt.decode()` without verify | AI uses decode for "reading" the token | Always use `jwt.verify()` with algorithm pinning |
| Storing tokens in localStorage | AI uses localStorage for convenience | Use `httpOnly` cookies with `Secure` and `SameSite` |
| Missing rate limiting | AI doesn't add it unless asked | Always rate-limit auth, AI, payment, and expensive endpoints |
| Admin routes without auth check | AI creates routes and forgets middleware | Apply auth middleware globally, whitelist public routes |
| `SELECT *` with user-controlled ID | AI doesn't verify ownership | Always add `WHERE user_id = $currentUser` |
| Error messages leaking details | AI returns raw error objects | Return generic messages, log details server-side |
