# Authentication & Session Security

## JWT Security

### Common Vulnerabilities

| Vulnerability | Prevention |
|---------------|-----------|
| `alg: none` attack | Always verify algorithm server-side, reject `none` |
| Algorithm confusion (RS256→HS256) | Explicitly specify expected algorithm, never derive from token |
| Weak HMAC secrets | Use 256+ bit cryptographically random secrets |
| Missing expiration | Always set `exp` claim |
| Token in localStorage | Store in httpOnly, Secure, SameSite=Strict cookies |
| Missing `jti` claim | Add unique ID for token revocation |
| Refresh token reuse | Implement rotation — invalidate old refresh token on use |

### Secure Implementation
```javascript
// 1. SIGNING
const secret = process.env.JWT_SECRET; // 256+ bits, cryptographically random

const token = jwt.sign({
  sub: userId,
  iat: Math.floor(Date.now() / 1000),
  exp: Math.floor(Date.now() / 1000) + (15 * 60), // 15 mins max
  jti: crypto.randomUUID()
}, secret, { algorithm: 'HS256' });

// 2. SENDING — httpOnly cookie
res.cookie('token', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 15 * 60 * 1000,
  path: '/',
});

// 3. VERIFYING — pin algorithm
jwt.verify(token, secret, { algorithms: ['HS256'] }, (err, decoded) => {
  if (err) return res.status(401).json({ error: 'Invalid token' });
  req.user = decoded;
});
```

### JWT Checklist
- [ ] Algorithm explicitly specified on verification (never trust token header)
- [ ] `alg: none` rejected
- [ ] Secret is 256+ bits of random data (not a password or phrase)
- [ ] `exp` claim always set and validated
- [ ] Tokens stored in httpOnly cookies (not localStorage/sessionStorage)
- [ ] Refresh token rotation implemented

---

## OAuth 2.0 / PKCE

### Common Mistakes
1. **Missing `state` parameter** — enables CSRF on OAuth flow
2. **Missing PKCE** — authorization code can be intercepted (especially mobile)
3. **Accepting tokens from URL fragment** — exposed in browser history and referrer
4. **Not validating redirect_uri** — open redirect via OAuth

### Secure Implementation
```javascript
// 1. Generate state and PKCE challenge
const state = crypto.randomUUID();
const codeVerifier = crypto.randomBytes(32).toString('base64url');
const codeChallenge = crypto.createHash('sha256')
  .update(codeVerifier).digest('base64url');

// Store in session
session.oauthState = state;
session.codeVerifier = codeVerifier;

// 2. Redirect to provider with state + PKCE
const authUrl = `https://provider.com/authorize?` +
  `client_id=${CLIENT_ID}&` +
  `redirect_uri=${encodeURIComponent(REDIRECT_URI)}&` +
  `response_type=code&` +
  `state=${state}&` +
  `code_challenge=${codeChallenge}&` +
  `code_challenge_method=S256`;

// 3. On callback — validate state
if (req.query.state !== session.oauthState) {
  throw new Error('Invalid state — possible CSRF');
}

// 4. Exchange code with PKCE verifier
const tokenResponse = await fetch('https://provider.com/token', {
  method: 'POST',
  body: new URLSearchParams({
    grant_type: 'authorization_code',
    code: req.query.code,
    redirect_uri: REDIRECT_URI,
    client_id: CLIENT_ID,
    code_verifier: session.codeVerifier,
  }),
});
```

---

## Password Security

### Requirements
- Minimum 8 characters (12+ recommended)
- No maximum length limit (or very high, e.g., 128 chars)
- Allow all characters including special chars and spaces
- Don't require specific character types (complexity requirements reduce entropy)

### Storage
- **Use**: Argon2id, bcrypt, or scrypt
- **Never**: MD5, SHA1, plain SHA256, or any unsalted hash

```javascript
import { hash, verify } from '@node-rs/argon2'; // or bcrypt

// Hashing
const hashedPassword = await hash(password, {
  memoryCost: 65536,  // 64 MB
  timeCost: 3,
  parallelism: 4,
});

// Verification
const isValid = await verify(hashedPassword, password);
```

---

## Session Management

### Cookie Flags
```
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
```

| Flag | Purpose |
|------|---------|
| `HttpOnly` | Prevents JavaScript access (mitigates XSS token theft) |
| `Secure` | Only sent over HTTPS |
| `SameSite=Strict` | Not sent on cross-site requests (strongest CSRF protection) |
| `SameSite=Lax` | Sent on top-level navigations only (good balance) |
| `Path=/` | Scope cookie to entire site |
| `Max-Age` | Set expiration (prefer over `Expires`) |

### Cookie Prefixes
```
// __Host- prefix: most restrictive
Set-Cookie: __Host-session=abc; Secure; HttpOnly; SameSite=Strict; Path=/

// __Secure- prefix: requires Secure flag
Set-Cookie: __Secure-session=abc; Secure; HttpOnly; SameSite=Strict; Path=/
```

`__Host-` prefix guarantees: Secure flag set, sent from secure origin, Path=/, no Domain attribute (prevents subdomain attacks).

### Session Lifecycle
1. **Regenerate session ID** after login (prevent session fixation)
2. **Invalidate sessions** on password change
3. **Implement idle timeout** (e.g., 30 minutes of inactivity)
4. **Implement absolute timeout** (e.g., 24 hours max)
5. **Revoke all sessions** on account deletion or deactivation

---

## WebSocket Authentication

### The Problem
WebSocket connections authenticate once at handshake but messages aren't individually authenticated.

### Secure Pattern
```javascript
// Server-side
wss.on('connection', (ws, req) => {
  // 1. Authenticate at connection
  const token = parseCookies(req.headers.cookie)?.session;
  const user = await verifyToken(token);
  if (!user) { ws.close(1008, 'Unauthorized'); return; }

  ws.userId = user.id;

  ws.on('message', async (data) => {
    const msg = JSON.parse(data);

    // 2. Authorize EACH message/action
    if (msg.action === 'readDocument') {
      const hasAccess = await checkAccess(ws.userId, msg.documentId);
      if (!hasAccess) { ws.send(JSON.stringify({ error: 'Forbidden' })); return; }
    }

    // 3. Validate and sanitize message data
    const validated = validateInput(msg);
    // ... process
  });
});
```

### Checklist
- [ ] Authentication at WebSocket handshake (cookie or token)
- [ ] Authorization check on each message/action
- [ ] Input validation on all message data
- [ ] Rate limiting on WebSocket messages
- [ ] Connection timeout for idle connections
- [ ] Message size limits
