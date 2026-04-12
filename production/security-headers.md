# Security Headers

## Complete Header Set

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self' https://api.yourdomain.com; frame-ancestors 'none'; base-uri 'self'; form-action 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
Cache-Control: no-store (for sensitive pages)
X-DNS-Prefetch-Control: off
```

## Header Reference

| Header | Purpose | Value |
|--------|---------|-------|
| `Strict-Transport-Security` | Forces HTTPS | `max-age=31536000; includeSubDomains; preload` |
| `Content-Security-Policy` | Controls allowed resource sources | See CSP section below |
| `X-Content-Type-Options` | Prevents MIME sniffing | `nosniff` |
| `X-Frame-Options` | Prevents clickjacking | `DENY` or `SAMEORIGIN` |
| `Referrer-Policy` | Controls referrer information | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Controls browser features | `camera=(), microphone=(), geolocation=()` |
| `Cache-Control` | Prevents caching sensitive data | `no-store` for auth pages |

## Content Security Policy (CSP) — Deep Dive

### Building a CSP
```
Content-Security-Policy:
  default-src 'self';                              # Fallback for all directives
  script-src 'self' 'nonce-{random}';              # Scripts: self + nonce
  style-src 'self' 'unsafe-inline';                # Styles: self + inline (often needed)
  img-src 'self' data: https://cdn.example.com;    # Images: self + data URIs + CDN
  font-src 'self' https://fonts.gstatic.com;       # Fonts
  connect-src 'self' https://api.example.com;      # XHR/fetch/WebSocket
  frame-ancestors 'none';                          # No framing (replaces X-Frame-Options)
  base-uri 'self';                                 # Restrict <base> tag
  form-action 'self';                              # Restrict form submissions
  upgrade-insecure-requests;                       # Upgrade HTTP→HTTPS
  report-uri /csp-report;                          # Report violations
```

### Critical Rules
- **Never** use `'unsafe-eval'` for scripts (enables `eval()`)
- **Avoid** `'unsafe-inline'` for scripts (use nonces instead)
- Use `nonce-{random}` for inline scripts that are necessary
- Start with `default-src 'none'` and add as needed for strictest policy
- Use `report-uri` to catch violations before enforcing

### CSP with Nonces (Next.js example)
```javascript
// middleware.ts
import { NextResponse } from 'next/server';

export function middleware(request) {
  const nonce = Buffer.from(crypto.randomUUID()).toString('base64');
  const cspHeader = `
    default-src 'self';
    script-src 'self' 'nonce-${nonce}';
    style-src 'self' 'unsafe-inline';
    img-src 'self' data: blob:;
    connect-src 'self';
    frame-ancestors 'none';
  `.replace(/\n/g, '');

  const response = NextResponse.next();
  response.headers.set('Content-Security-Policy', cspHeader);
  response.headers.set('x-nonce', nonce);
  return response;
}
```

## CORS Configuration

```javascript
// ❌ VULNERABLE — allows everything
app.use(cors({ origin: '*', credentials: true })); // wildcard + credentials = CRITICAL

// ❌ VULNERABLE — reflecting origin without validation
app.use(cors({ origin: req.headers.origin, credentials: true }));

// ✅ SECURE — explicit allowlist
const allowedOrigins = [
  'https://yourdomain.com',
  'https://app.yourdomain.com',
];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
```

## Checklist
- [ ] HSTS header set with long max-age + includeSubDomains
- [ ] CSP configured (at minimum: default-src 'self', frame-ancestors 'none')
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options: DENY (or frame-ancestors in CSP)
- [ ] Referrer-Policy set
- [ ] Permissions-Policy restricts unused browser features
- [ ] CORS uses explicit origin allowlist (no wildcard with credentials)
- [ ] Cache-Control: no-store on authenticated/sensitive pages
- [ ] CSP report-uri configured to catch violations
