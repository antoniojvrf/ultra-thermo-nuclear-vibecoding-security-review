# CSRF, SSRF & Open Redirect

## Cross-Site Request Forgery (CSRF)

### Endpoints Requiring Protection
- All POST, PUT, PATCH, DELETE requests
- Login, signup, password reset/change endpoints
- Payment/transaction endpoints
- OAuth callback endpoints
- Any GET that changes state (refactor to POST)

### Protection Mechanisms

**1. CSRF Tokens**
- Generate cryptographically random tokens tied to user session
- Validate on every state-changing request
- Regenerate after login

**2. SameSite Cookies**
```
Set-Cookie: session=abc123; SameSite=Strict; Secure; HttpOnly
```

**3. Double Submit Cookie Pattern**
- Send CSRF token in both cookie and request header
- Server validates they match

### Edge Cases
- Token validation must NOT depend on whether token is present — always require it
- JSON APIs are NOT immune — validate Origin/Referer headers AND use tokens
- CORS misconfiguration can bypass SameSite cookies
- Don't include CSRF tokens in URLs (leaked via Referer header)

### Verification Checklist
- [ ] Token is cryptographically random
- [ ] Token is tied to user session
- [ ] Token validated server-side on all state-changing requests
- [ ] Missing token = rejected request
- [ ] Token regenerated on auth state change
- [ ] SameSite cookie attribute set
- [ ] Secure and HttpOnly flags on session cookies

---

## Server-Side Request Forgery (SSRF)

### Vulnerable Features
- Webhooks (user provides callback URL)
- URL previews / link unfurling
- PDF generators from URLs
- Image/file fetching from URLs
- Import-from-URL features
- RSS/feed readers
- API integrations with user-provided endpoints
- HTML-to-PDF/image converters

### IP and DNS Bypass Techniques to Block

| Technique | Example | Description |
|-----------|---------|-------------|
| Decimal IP | `http://2130706433` | 127.0.0.1 as decimal |
| Octal IP | `http://0177.0.0.1` | Octal representation |
| Hex IP | `http://0x7f.0x0.0x0.0x1` | Hexadecimal |
| IPv6 localhost | `http://[::1]` | IPv6 loopback |
| IPv6 mapped IPv4 | `http://[::ffff:127.0.0.1]` | IPv4-mapped IPv6 |
| Short IPv6 | `http://[::]` | All zeros |
| DNS rebinding | Attacker DNS returns internal IP | First→external, second→internal |
| CNAME to internal | Attacker domain CNAMEs to internal | DNS points to internal hostname |
| Redirect chains | External URL redirects to internal | Follow redirects carefully |
| Shortened IP | `http://127.1` | Shortened notation |

### Cloud Metadata Protection
Block these endpoints:
- AWS: `169.254.169.254`, `fd00:ec2::254`
- GCP: `metadata.google.internal`, `169.254.169.254`
- Azure: `169.254.169.254`

### DNS Rebinding Prevention
1. Resolve DNS before making request
2. Validate resolved IP is not internal/private
3. Pin the resolved IP for the request (don't re-resolve)

### SSRF Checklist
- [ ] Validate URL scheme is HTTP/HTTPS only
- [ ] Resolve DNS and validate IP is not private/internal
- [ ] Block cloud metadata IPs explicitly
- [ ] Limit or disable redirect following
- [ ] If following redirects, validate each hop
- [ ] Set timeout on requests
- [ ] Limit response size
- [ ] Use network isolation where possible

---

## Open Redirect

### Protection Strategies
1. **Allowlist validation** — only allow pre-approved domains
2. **Relative URLs only** — accept paths, not full URLs
3. **Indirect references** — `?redirect=dashboard` → server lookup

### Bypass Techniques to Block

| Technique | Example | Why It Works |
|-----------|---------|-------------|
| @ symbol | `https://legit.com@evil.com` | Browser navigates to evil.com |
| Subdomain abuse | `https://legit.com.evil.com` | evil.com owns the subdomain |
| Protocol tricks | `javascript:alert(1)` | XSS via redirect |
| Double URL encoding | `%252f%252fevil.com` | Decodes to `//evil.com` |
| Backslash | `https://legit.com\@evil.com` | Parsers normalize `\` to `/` |
| Null byte | `https://legit.com%00.evil.com` | Parsers truncate at null |
| Unicode normalization | `https://legіt.com` (Cyrillic і) | IDN homograph attack |
| Data URLs | `data:text/html,<script>...` | Direct payload execution |
| Protocol-relative | `//evil.com` | Uses current page's protocol |
| Fragment abuse | `https://legit.com#@evil.com` | Parsed differently |
| Tab/newline | `https://legit.com%09.evil.com` | Whitespace confusion |

### Secure Validation
```javascript
function isValidRedirect(url) {
  // Only allow relative paths
  if (!url.startsWith('/') || url.startsWith('//')) return false;
  
  // Block protocol handlers
  if (url.includes(':')) return false;
  
  // Or: allowlist approach
  const allowed = ['yourdomain.com', 'app.yourdomain.com'];
  try {
    const parsed = new URL(url, 'https://yourdomain.com');
    return allowed.includes(parsed.hostname);
  } catch {
    return false;
  }
}
```
