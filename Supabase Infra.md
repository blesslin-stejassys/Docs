# Supabase with Next.js Integration Guide

## 1. Introduction

This guide explains how to **set up Supabase with a Next.js application**, including creating a project, adding sample data, connecting your app, and querying your database.  
Supabase serves as the backend service — providing authentication, database, and storage features for your app.

---

## 2. Create a Supabase Project

### Option 1: Dashboard Setup
Go to [https://database.new](https://database.new) and create a new Supabase project.

### Option 2: API Setup via Management API
```bash
# First, get your access token from https://supabase.com/dashboard/account/tokens
export SUPABASE_ACCESS_TOKEN="your-access-token"

# List your organizations to get the organization ID
curl -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN"   https://api.supabase.com/v1/organizations

# Create a new project (replace <org-id> with your organization ID)
curl -X POST https://api.supabase.com/v1/projects   -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN"   -H "Content-Type: application/json"   -d '{
    "organization_id": "<org-id>",
    "name": "My Project",
    "region": "us-east-1",
    "db_pass": "<your-secure-password>"
  }'
```

Once your project is running, go to the **Table Editor**, create a new table, and insert some data.

### SQL Example
Run the following snippet in your **SQL Editor** to create an `instruments` table with sample data:
```sql
-- Create the table
create table instruments (
  id bigint primary key generated always as identity,
  name text not null
);

-- Insert sample data
insert into instruments (name)
values
  ('violin'),
  ('viola'),
  ('cello');

-- Enable Row Level Security (RLS)
alter table instruments enable row level security;

-- Add policy for public read access
create policy "public can read instruments"
on public.instruments
for select to anon
using (true);
```

---

## 3. Create a Next.js App

Use the **with-supabase template** to create a Next.js app pre-configured with Supabase Auth, TypeScript, and Tailwind CSS:

```bash
npx create-next-app -e with-supabase
```

This sets up a ready-to-use project with Supabase client-side configuration included.

---

## 4. Declare Supabase Environment Variables

Rename `.env.example` to `.env.local` and populate it with your Supabase credentials.

Example:
```bash
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=your-anon-key
```

---

## 5. Create Supabase Client

Create a new file: `utils/supabase/server.ts`

```ts
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch {
            // The `setAll` method was called from a Server Component.
            // This can be ignored if you have middleware refreshing
            // user sessions.
          }
        },
      },
    }
  )
}
```

This client uses the credentials from `.env.local` and manages cookies for authentication automatically.

---

## 6. Query Supabase Data from Next.js

Create a new page file: `app/instruments/page.tsx`

```tsx
import { createClient } from '@/utils/supabase/server';

export default async function Instruments() {
  const supabase = await createClient();
  const { data: instruments } = await supabase.from("instruments").select();
  return <pre>{JSON.stringify(instruments, null, 2)}</pre>
}
```

This page fetches all records from the `instruments` table and displays them as formatted JSON.

---

## 7. Run the App

Start the development server:

```bash
npm run dev
```

Visit [http://localhost:3000/instruments](http://localhost:3000/instruments) in your browser — you should see your sample instruments listed.

---

## 8. Next Steps

- ✅ **Set up Auth** for user sign-in and session management.  
- ✅ **Insert more data** into your Supabase database for testing.  
- ✅ **Upload and serve static files** using Supabase **Storage**.  

For more advanced features, check out [Supabase Documentation](https://supabase.com/docs).

---

## 9. Conclusion

You’ve successfully connected **Supabase** with **Next.js** and performed your first database query.  
From here, you can expand your app with authentication, real-time updates, and file storage integrations to create a complete full-stack experience.
