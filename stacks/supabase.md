# VibeShield Security Rules: Supabase Stack

**Project:** VibeShield
**Description:** Supabase-specific security rules for AI-generated code validation
**Version:** 0.1.0
**Supplement to:** vibeshield-rules.md

## Overview

This document provides Supabase-specific security rules that complement the core VibeShield rules. Supabase is extremely prevalent in AI-generated applications, and AI tools consistently make critical security errors with Row Level Security, authentication, and key management.

---

## Row Level Security (RLS)

### RLS Enablement

Always enable Row Level Security on every table. [V-01, V-04]

Never rely on application-level access control alone. RLS is your database-level authorization boundary. [V-01]

NEVER: Create tables without RLS enabled

```sql
CREATE TABLE user_profiles (
  id UUID PRIMARY KEY,
  user_id UUID,
  email TEXT,
  data JSONB
);
-- Missing: ALTER TABLE user_profiles ENABLE ROW LEVEL SECURITY;
```

ALWAYS: Enable RLS on all tables

```sql
CREATE TABLE user_profiles (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id),
  email TEXT,
  data JSONB
);

ALTER TABLE user_profiles ENABLE ROW LEVEL SECURITY;
```

### RLS Policy Definition

Always write explicit SELECT, INSERT, UPDATE, and DELETE policies for each table. [V-01]

Never use `USING (true)` as a real policy. It defeats the purpose of RLS. [V-01]

Never leave tables with RLS enabled but zero policies. This blocks all access, including legitimate queries. [V-01]

NEVER: Placeholder or permissive policies

```sql
CREATE POLICY "allow_all" ON user_profiles
  FOR ALL USING (true);  -- Bypasses all security

CREATE POLICY "temporary" ON user_profiles
  FOR SELECT USING (true);  -- TODO: fix later (never gets fixed)
```

ALWAYS: Explicit, restrictive policies

```sql
-- Users can only read their own profile
CREATE POLICY "users_select_own" ON user_profiles
  FOR SELECT USING (auth.uid() = user_id);

-- Users can only insert their own profile
CREATE POLICY "users_insert_own" ON user_profiles
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- Users can only update their own profile
CREATE POLICY "users_update_own" ON user_profiles
  FOR UPDATE USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- Users can only delete their own profile
CREATE POLICY "users_delete_own" ON user_profiles
  FOR DELETE USING (auth.uid() = user_id);
```

### Policy Validation

Always test RLS policies as different users and as anonymous users. [V-01]

Never assume policies work without testing. AI-generated policies frequently have logic errors. [V-01]

Always verify that `WITH CHECK` is used for INSERT and UPDATE policies to prevent privilege escalation. [V-01]

---

## Key Management

### Service Role vs Anon Key

Never expose the `service_role` key to the client. It bypasses all RLS policies. [V-02, V-09]

Always use the `anon` key in frontend code. It respects RLS and is designed to be public. [V-02]

Never commit either key to version control. Use environment variables. [V-09]

NEVER: Service role key in client code

```typescript
// frontend/lib/supabase.ts
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  'https://xxx.supabase.co',
  'eyJhbGc...service_role_key...'  // BYPASSES ALL RLS!
)
```

ALWAYS: Anon key in client, service role only server-side

```typescript
// frontend/lib/supabase.ts
const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!  // Public key, RLS enforced
)

// backend/lib/supabase.ts (server only)
const supabaseAdmin = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!  // Server only, never exposed
)
```

### Environment Variables

Always prefix client-side environment variables with `NEXT_PUBLIC_`, `VITE_`, `REACT_APP_`, or framework equivalent. [V-02]

Never use service role keys in client-side environment variables, even with prefixes. [V-02, V-09]

Always validate that service role keys are only loaded in server-side code paths. [V-02]

---

## Authentication

### Auth Implementation

Always use Supabase Auth, not custom authentication, when on Supabase. [V-04, V-05]

Never store JWT secrets in client code. Supabase manages this server-side. [V-02]

Never implement custom password hashing when Supabase Auth is available. [V-05]

### Token Validation

Always use `supabase.auth.getUser()` server-side to verify tokens. Never use only `getSession()`. [V-04]

Never trust client-side session state alone. Always validate on the server. [V-04]

Always verify the JWT signature server-side for protected API routes. [V-04]

NEVER: Client-side session check only

```typescript
// API route handler
export async function POST(request: Request) {
  const { data: { session } } = await supabase.auth.getSession()
  // getSession() only checks local storage, doesn't validate with server

  if (session) {
    // INSECURE: session could be forged or expired
    return await processRequest()
  }
}
```

ALWAYS: Server-side user verification

```typescript
// API route handler
export async function POST(request: Request) {
  const { data: { user }, error } = await supabase.auth.getUser()
  // getUser() validates JWT with Supabase servers

  if (error || !user) {
    return new Response('Unauthorized', { status: 401 })
  }

  return await processRequest(user)
}
```

### Auth Configuration

Always configure email verification in production. [V-04]

Always set secure password requirements (minimum length, complexity) in Supabase dashboard. [V-05]

Always enable email rate limiting to prevent abuse. [V-08]

Never disable CAPTCHA on signup in production without alternative bot protection. [V-08]

---

## Storage Security

### Bucket Policies

Always set storage bucket policies. Buckets are NOT automatically private. [V-01]

Never assume default buckets are secure. They are accessible with the service role key. [V-01]

Always create explicit SELECT, INSERT, UPDATE, DELETE policies for storage objects matching RLS patterns. [V-01]

NEVER: Bucket without policies

```sql
-- Bucket created via dashboard with no policies
-- Anyone with service role key has full access
```

ALWAYS: Explicit bucket policies

```sql
-- Users can only read their own uploaded files
CREATE POLICY "users_select_own_files" ON storage.objects
  FOR SELECT USING (
    bucket_id = 'avatars' AND
    auth.uid()::text = (storage.foldername(name))[1]
  );

-- Users can only upload to their own folder
CREATE POLICY "users_insert_own_files" ON storage.objects
  FOR INSERT WITH CHECK (
    bucket_id = 'avatars' AND
    auth.uid()::text = (storage.foldername(name))[1]
  );
```

### File Upload Validation

Always validate file types and sizes in storage policies or edge functions. [V-16]

Never allow unrestricted file uploads to public buckets. [V-16]

Always sanitize filenames before storage. Reject path traversal characters. [V-16]

---

## Edge Functions

### Secret Management

Never hardcode secrets in edge functions. Use Supabase secrets management. [V-02]

Always use `supabase secrets set SECRET_NAME=value` for sensitive values. [V-02]

Always access secrets via `Deno.env.get('SECRET_NAME')` in edge functions. [V-02]

NEVER: Hardcoded secrets

```typescript
// supabase/functions/payment/index.ts
const stripe = new Stripe('sk_live_hardcoded_key_12345')  // EXPOSED IN REPO
```

ALWAYS: Environment-based secrets

```bash
# Set secret via CLI
supabase secrets set STRIPE_SECRET_KEY=sk_live_xxx
```

```typescript
// supabase/functions/payment/index.ts
const stripe = new Stripe(Deno.env.get('STRIPE_SECRET_KEY')!)
```

### Input Validation

Always validate input in edge functions the same as any server endpoint. [V-06]

Never trust request bodies without validation. [V-06]

Always use schema validation libraries (Zod, Yup) for edge function inputs. [V-06]

Always set CORS headers explicitly in edge functions. [V-10]

NEVER: Unvalidated edge function input

```typescript
// supabase/functions/process-order/index.ts
Deno.serve(async (req) => {
  const { orderId, amount } = await req.json()
  // No validation - amount could be negative, orderId could be malicious
  return await processPayment(orderId, amount)
})
```

ALWAYS: Validated edge function input

```typescript
import { z } from 'zod'

const OrderSchema = z.object({
  orderId: z.string().uuid(),
  amount: z.number().positive().max(1000000),
})

Deno.serve(async (req) => {
  const body = await req.json()
  const validatedData = OrderSchema.parse(body)  // Throws if invalid

  return await processPayment(validatedData.orderId, validatedData.amount)
})
```

---

## Database Functions

### Security Definer

Never mark database functions as `SECURITY DEFINER` unless absolutely necessary. [V-01]

Always use `SECURITY INVOKER` (default) when the function should run with caller's permissions. [V-01]

Always set explicit `search_path` when using `SECURITY DEFINER` to prevent search path attacks. [V-17]

NEVER: Unsafe SECURITY DEFINER

```sql
CREATE FUNCTION delete_any_user(target_id UUID)
RETURNS VOID
SECURITY DEFINER  -- Runs as function owner (often superuser)
AS $$
  DELETE FROM users WHERE id = target_id;
$$ LANGUAGE sql;
-- Any user can call this to delete any other user!
```

ALWAYS: Safe function design

```sql
CREATE FUNCTION delete_own_account()
RETURNS VOID
SECURITY INVOKER  -- Runs as caller, respects RLS
AS $$
  DELETE FROM users WHERE id = auth.uid();
$$ LANGUAGE sql;

-- If SECURITY DEFINER is required:
CREATE FUNCTION admin_function()
RETURNS VOID
SECURITY DEFINER
SET search_path = public, pg_temp  -- Prevent search path attacks
AS $$
  -- Validate caller has admin role
  IF NOT EXISTS (
    SELECT 1 FROM user_roles
    WHERE user_id = auth.uid() AND role = 'admin'
  ) THEN
    RAISE EXCEPTION 'Unauthorized';
  END IF;

  -- Perform privileged operation
$$ LANGUAGE plpgsql;
```

### Function Exposure

Never expose `SECURITY DEFINER` functions via PostgREST without input validation. [V-01, V-06]

Always validate all function parameters for type, range, and business logic constraints. [V-06, V-07]

Always use RLS policies on tables accessed by functions, even for `SECURITY DEFINER`. [V-01]

---

## Real-time Subscriptions

### Channel Authorization

Always implement authorization checks for real-time subscriptions. [V-01]

Never assume real-time channels are automatically secured by RLS. Verify independently. [V-01]

Always use RLS policies on tables used in real-time subscriptions. [V-01]

### Presence and Broadcast

Always validate user identity in presence and broadcast channels. [V-04]

Never trust client-supplied user metadata in real-time channels. [V-04]

Always rate limit real-time events to prevent abuse. [V-08]

---

## Migration and Schema Management

### Migration Security

Always enable RLS in the same migration that creates a table. [V-01]

Never deploy table migrations without corresponding RLS policies. [V-01]

Always review AI-generated migrations for missing security controls. [V-01]

### Schema Changes

Always test RLS policies after schema changes. Policy logic may break with column additions/removals. [V-01]

Never assume existing policies work correctly after foreign key or column type changes. [V-01]

---

## Implementation Notes

### Priority Rules

Supabase-specific high-priority rules:
1. **Enable RLS on every table** (most critical AI failure)
2. **Never expose service_role key to client** (most dangerous AI failure)
3. **Use getUser() not getSession() server-side** (frequent auth bypass)
4. **Set storage bucket policies** (frequently forgotten)
5. **Validate edge function inputs** (frequently overlooked)

### Common AI Failures

AI tools frequently:
- Create tables without RLS enabled
- Use service_role key in frontend code
- Write `USING (true)` placeholder policies
- Forget storage bucket policies
- Use only `getSession()` for server-side auth checks
- Hardcode secrets in edge functions
- Create `SECURITY DEFINER` functions without access control

### Integration with Core Rules

These Supabase rules extend core VibeShield rules:
- V-01 (IDOR): RLS is the primary defense
- V-02 (Hardcoded Secrets): Service role key management
- V-04 (Client-Side Auth): getUser() vs getSession()
- V-06 (Injection): Edge function input validation
- V-09 (Committed Secrets): .env for both keys

---

**End of VibeShield Supabase Security Rules v0.1.0**
