# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Tech Stack

- **Next.js 14.2.23** with App Router
- **TypeScript** with path alias `@/*` for `src/*`
- **Supabase** for auth, database, and edge functions
- **Stripe** for payments (primary payment processor)
- **Tailwind CSS** with shadcn/ui components
- **React Hook Form** for form handling

## Development Commands

```bash
npm run dev    # Start development server (http://localhost:3000)
npm run build  # Build for production
npm run start  # Start production server

./deploy-webhooks.sh  # Deploy Supabase edge functions
```

## Architecture Overview

### Authentication Flow
- Supabase Auth handles email/password authentication
- Server-side auth: `createClient` from `@/utils/supabase/server`
- Client-side auth: `createClient` from `@/utils/supabase/client`
- Auth callback route: `/auth/callback`
- Protected routes use middleware checks

### Payment Integration
- Stripe checkout sessions created via `/supabase/functions/create-checkout`
- Webhook events processed at `/supabase/functions/payments-webhook`
- Subscription data stored in `subscriptions` table
- Webhook signature verification is enforced

### Database Schema
Key tables with RLS (Row Level Security) enabled:
- `users` - User profiles, linked to auth.users
- `subscriptions` - Stripe subscription records
- `webhook_events` - Payment event logs

### Project Structure
```
src/app/(auth)/       # Auth pages (sign-in, sign-up, etc.)
src/app/dashboard/    # Protected dashboard area
src/components/ui/    # shadcn/ui components
supabase/functions/   # Edge functions (Deno runtime)
```

## Key Patterns

1. **Server Components by Default** - Use `"use client"` only when needed
2. **Form Handling** - Use server actions in `app/actions.ts`
3. **Error Messages** - Display via `<FormMessage />` component
4. **Protected Routes** - Middleware checks auth state
5. **Environment Variables** - Required: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`

## Supabase Edge Functions

Edge functions run in Deno runtime and require:
- Import maps are not supported - use full URLs or npm: specifiers
- JWT verification disabled for webhook endpoints (see `config.toml`)
- Deploy with `./deploy-webhooks.sh` script