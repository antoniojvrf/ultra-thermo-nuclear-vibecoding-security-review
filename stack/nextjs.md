# Next.js Security

## Server Actions (Next.js 14+)

### The Problem
Server Actions are server-side functions callable from the client. AI assistants often create them without input validation or auth checks.

```typescript
// ❌ VULNERABLE — no auth, no validation
'use server'
export async function updateProfile(data: FormData) {
  await db.users.update({
    where: { id: data.get('userId') as string }, // attacker controls userId
    data: { name: data.get('name'), role: data.get('role') } // mass assignment
  });
}

// ✅ SECURE — auth + validation + field whitelist
'use server'
import { z } from 'zod';
import { auth } from '@/lib/auth';

const updateProfileSchema = z.object({
  name: z.string().min(1).max(100),
  bio: z.string().max(500).optional(),
});

export async function updateProfile(formData: FormData) {
  const session = await auth();
  if (!session?.user?.id) throw new Error('Unauthorized');

  const parsed = updateProfileSchema.safeParse({
    name: formData.get('name'),
    bio: formData.get('bio'),
  });
  if (!parsed.success) throw new Error('Invalid input');

  await db.users.update({
    where: { id: session.user.id }, // from session, not from client
    data: parsed.data, // validated and whitelisted fields only
  });
}
```

## Route Handlers (App Router)

```typescript
// ❌ VULNERABLE — no auth check
export async function GET(req: Request) {
  const users = await db.users.findMany();
  return Response.json(users);
}

// ✅ SECURE
export async function GET(req: Request) {
  const session = await auth();
  if (!session?.user) return new Response('Unauthorized', { status: 401 });

  const users = await db.users.findMany({
    where: { orgId: session.user.orgId },
    select: { id: true, name: true, email: true }, // don't return sensitive fields
  });
  return Response.json(users);
}
```

## Middleware Authentication

```typescript
// middleware.ts — protect routes globally
import { auth } from '@/lib/auth';
import { NextResponse } from 'next/server';

export async function middleware(request: Request) {
  const session = await auth();

  // Protect all /dashboard and /api routes (except /api/auth and /api/public)
  const isProtected = request.nextUrl.pathname.startsWith('/dashboard') ||
    (request.nextUrl.pathname.startsWith('/api') &&
     !request.nextUrl.pathname.startsWith('/api/auth') &&
     !request.nextUrl.pathname.startsWith('/api/public'));

  if (isProtected && !session) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

## Environment Variables

```
# ✅ Server-only (not exposed to browser)
DATABASE_URL=postgresql://...
STRIPE_SECRET_KEY=sk_live_...
JWT_SECRET=...

# ⚠️ Client-exposed (ONLY for public, non-secret values)
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...  # anon key is designed to be public
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_...

# ❌ NEVER prefix secrets with NEXT_PUBLIC_
# NEXT_PUBLIC_SUPABASE_SERVICE_KEY=...  ← CRITICAL VULNERABILITY
# NEXT_PUBLIC_STRIPE_SECRET_KEY=...     ← CRITICAL VULNERABILITY
```

## Security Headers (next.config.js)
```javascript
const securityHeaders = [
  { key: 'X-Frame-Options', value: 'DENY' },
  { key: 'X-Content-Type-Options', value: 'nosniff' },
  { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
  { key: 'Strict-Transport-Security', value: 'max-age=31536000; includeSubDomains' },
  { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
];

module.exports = {
  async headers() {
    return [{ source: '/(.*)', headers: securityHeaders }];
  },
  // Disable source maps in production
  productionBrowserSourceMaps: false,
};
```

## Checklist
- [ ] Server Actions validate input with Zod/joi
- [ ] Server Actions verify auth (not trusting client FormData for user identity)
- [ ] Server Actions whitelist updatable fields (no mass assignment)
- [ ] Route Handlers check authentication
- [ ] Middleware protects routes globally
- [ ] No secrets prefixed with `NEXT_PUBLIC_`
- [ ] Security headers configured
- [ ] Source maps disabled in production
- [ ] API routes return minimal data (select specific fields)
