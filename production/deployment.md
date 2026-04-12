# Deployment Security

## Source Maps

```javascript
// ❌ VULNERABLE — source maps in production expose original source code
// next.config.js
module.exports = { productionBrowserSourceMaps: true };

// ✅ SECURE — disable source maps in production
module.exports = { productionBrowserSourceMaps: false };

// Vite:
export default defineConfig({
  build: { sourcemap: false }, // or 'hidden' for error tracking only
});
```

## Debug Endpoints

```javascript
// ❌ VULNERABLE — debug routes accessible in production
app.get('/api/debug', (req, res) => res.json(process.env)); // EXPOSES ALL SECRETS
app.get('/api/health', (req, res) => res.json({
  db: dbConfig, // connection strings
  redis: redisConfig,
  uptime: process.uptime(),
  memory: process.memoryUsage(),
}));

// ✅ SECURE — remove or protect debug endpoints
if (process.env.NODE_ENV !== 'production') {
  app.get('/api/debug', authMiddleware, adminOnly, debugHandler);
}

// Health check with minimal info
app.get('/api/health', (req, res) => res.json({ status: 'ok' }));
```

## Environment Separation

```
# ✅ Separate configs per environment
.env.development    # local development
.env.staging        # staging/preview
.env.production     # production (stored in deployment platform, not in repo)

# ✅ .gitignore
.env
.env.*
!.env.example       # commit example template only
```

### .env.example Template
```
# Copy to .env and fill in values
DATABASE_URL=
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
JWT_SECRET=
OPENAI_API_KEY=
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
```

## Exposed Files in Production

### .git Directory
```nginx
# Nginx — block .git access
location ~ /\.git {
  deny all;
  return 404;
}
```

### Sensitive Files to Block
```nginx
# Block access to sensitive files
location ~ /\.(env|git|htaccess|htpasswd) {
  deny all;
  return 404;
}

location ~ /(package\.json|package-lock\.json|yarn\.lock|composer\.json|tsconfig\.json) {
  deny all;
  return 404;
}
```

## Attack Surface Documentation

Document all exposed endpoints, services, and entry points:

```markdown
## Attack Surface Map
| Endpoint | Auth Required | Rate Limited | Public |
|----------|:------------:|:------------:|:------:|
| GET /api/health | ❌ | ✅ | ✅ |
| POST /api/auth/login | ❌ | ✅ (10/min) | ✅ |
| POST /api/auth/register | ❌ | ✅ (5/hour) | ✅ |
| GET /api/users/me | ✅ | ✅ | ❌ |
| POST /api/checkout | ✅ | ✅ (10/min) | ❌ |
| POST /api/webhooks/stripe | ❌* | ❌ | ✅ |
*Verified via webhook signature
```

## Email Security (SPF/DKIM/DMARC)

For transactional emails (password resets, notifications):
```
# DNS Records
# SPF — specify who can send email for your domain
TXT  @  "v=spf1 include:_spf.google.com include:sendgrid.net -all"

# DKIM — cryptographic email signing (configured via email provider)
TXT  selector._domainkey  "v=DKIM1; k=rsa; p=..."

# DMARC — policy for handling failed checks
TXT  _dmarc  "v=DMARC1; p=reject; rua=mailto:dmarc@yourdomain.com"
```

## Checklist
- [ ] Source maps disabled in production builds
- [ ] No debug/test endpoints accessible in production
- [ ] .env files in .gitignore (only .env.example committed)
- [ ] Environment variables separated per environment
- [ ] .git directory not accessible via web
- [ ] Sensitive files blocked from web access
- [ ] Error pages don't leak stack traces or internal details
- [ ] Attack surface documented
- [ ] SPF, DKIM, DMARC configured for email-sending domains
- [ ] HTTPS enforced (no HTTP fallback)
