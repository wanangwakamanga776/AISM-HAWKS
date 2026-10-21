# Hawks Hub — Architecture and Launch Status

Hawks Hub is a React/Vite single-page app backed by Supabase and configured for Vercel hosting. No Base44 account or browser-local persistence is used.

## Architecture

```text
Public visitors / staff
         ↓
React pages and admin workspace
         ↓
src/services/dataService.js
         ↓
Supabase Auth · Postgres with RLS · Storage · Realtime
```

| Table | Purpose |
|---|---|
| `players` | Stable identities and public roster profiles |
| `opponents` | Opponent names, badges, and locations |
| `seasons` | Sport and season dates/current status |
| `matches` | Results, publication state, participants, events, and game stats |
| `gallery` | Media metadata and publication state |
| `audit_logs` | Administrator match actions |

Events and player stat lines live on each match as JSONB so RLS protects all related details with the match's publication state.

## Security and persistence

- Supabase Auth signs staff in; public sign-up is not exposed by the application.
- A trusted `app_metadata.role = "admin"` claim is required by Postgres RLS for all writes and media uploads.
- Public reads are limited by RLS to published matches and gallery items, active non-archived players, and public seasons/opponents.
- No service-role secret is used in the browser. The Supabase anon/publishable key is safe to expose only with these RLS policies in place.
- Admin edits persist in Postgres and are visible to other clients. Realtime subscriptions refresh open pages.
- Uploads use the `gallery-media` public bucket; publish-state visibility applies to gallery records, while any uploaded media URL itself is public.

## Deployment and remaining setup

Implemented in this workspace:

- Supabase client, auth state, admin sign-in, password reset, and sign-out.
- Shared CRUD for matches, players, opponents, seasons, and gallery records.
- Admin-only RLS policies, Storage bucket policies, audit-log table, and Realtime publication setup in `supabase/migrations/20261021000000_hawks_hub.sql`.
- Vercel SPA route rewrites and environment-variable sample.
- Editable season management.
- Official hawk favicon.

Still required before this is a public live website:

1. Create or select a Supabase project and run the migration.
2. Set local `.env.local` values and configure the first administrator's trusted App Metadata.
3. Push the workspace to a Git provider and import it into Vercel.
4. Add the public Supabase URL/key as Vercel environment variables and deploy.
5. Verify sign-in, a persisted admin edit, public visibility, and Realtime behavior against the configured project.

This environment has not been given Supabase project credentials or a Vercel deployment target, so no remote database migration or production deploy has been performed. The application build and lint checks should be run after setup; successful local compilation does not substitute for a live RLS/auth integration check.
