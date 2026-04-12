# Supabase Security

## Row-Level Security (RLS) — #1 Critical Issue

### The Problem
AI assistants frequently enable RLS but create permissive policies that offer no real protection.

### DANGEROUS Patterns
```sql
-- ❌ CRITICAL — allows everything (same as no RLS)
CREATE POLICY "allow_all" ON public.profiles
  FOR ALL USING (true);

-- ❌ CRITICAL — using (true) with check (true)
CREATE POLICY "allow_all" ON public.documents
  FOR ALL USING (true) WITH CHECK (true);
```

### SECURE Patterns
```sql
-- ✅ Users can only read their own data
CREATE POLICY "users_read_own" ON public.profiles
  FOR SELECT USING (auth.uid() = user_id);

-- ✅ Users can only insert their own data
CREATE POLICY "users_insert_own" ON public.profiles
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- ✅ Users can only update their own data
CREATE POLICY "users_update_own" ON public.profiles
  FOR UPDATE USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- ✅ Users can only delete their own data
CREATE POLICY "users_delete_own" ON public.profiles
  FOR DELETE USING (auth.uid() = user_id);

-- ✅ Organization-scoped access
CREATE POLICY "org_members_read" ON public.projects
  FOR SELECT USING (
    org_id IN (
      SELECT org_id FROM public.org_members
      WHERE user_id = auth.uid()
    )
  );
```

### Service Role Key Exposure
```typescript
// ❌ CRITICAL — service_role key in client bundle
const supabase = createClient(url, process.env.NEXT_PUBLIC_SUPABASE_SERVICE_KEY!)

// ✅ CORRECT — anon key for client, service_role ONLY server-side
// Client:
const supabase = createClient(url, process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!)

// Server only (API route, Server Action):
import { createClient } from '@supabase/supabase-js'
const supabaseAdmin = createClient(url, process.env.SUPABASE_SERVICE_ROLE_KEY!) // no NEXT_PUBLIC_
```

### Auth Validation
```typescript
// ✅ Always verify auth server-side
import { createServerClient } from '@supabase/ssr'

export async function getAuthUser(request: Request) {
  const supabase = createServerClient(url, anonKey, {
    cookies: { /* cookie handling */ }
  });
  const { data: { user }, error } = await supabase.auth.getUser(); // NOT getSession()
  // getSession() reads from JWT without server validation
  // getUser() makes a server call to verify the token
  if (error || !user) throw new Error('Unauthorized');
  return user;
}
```

### Storage Security
```sql
-- ✅ Storage bucket policies
CREATE POLICY "users_own_files" ON storage.objects
  FOR ALL USING (
    bucket_id = 'user-uploads' AND
    (storage.foldername(name))[1] = auth.uid()::text
  );
```

### Checklist
- [ ] RLS enabled on ALL tables with user data
- [ ] No `USING (true)` or `WITH CHECK (true)` policies
- [ ] Policies use `auth.uid()` to scope data
- [ ] Service role key is NOT prefixed with `NEXT_PUBLIC_` or `VITE_`
- [ ] Service role key used only in server-side code
- [ ] `getUser()` used instead of `getSession()` for auth verification
- [ ] Storage policies restrict file access by user
- [ ] Edge functions validate auth tokens
