# Hawks Hub

Hawks Hub is the public athletics site and staff admin workspace for the AISM Hawks football and basketball programs. Supabase provides shared records, staff sign-in, media storage, row-level access controls, and live updates. Vercel hosts the Vite single-page app.

## Run locally

Requirements: Node.js 20.19+.

```bash
npm install
```

Create `.env.local` from `.env.example`, then set the Supabase project URL and public anon/publishable key:

```env
VITE_SUPABASE_URL=https://your-project-ref.supabase.co
VITE_SUPABASE_ANON_KEY=your-public-anon-or-publishable-key
```

These browser-visible values are protected by Supabase Row Level Security. Never put a Supabase `service_role` key in this app, `.env.local`, or Vercel's `VITE_` environment variables.

Run the database migration in the Supabase SQL Editor:

```text
supabase/migrations/20261021000000_hawks_hub.sql
```

Then start the app:

```bash
npm run dev
```

Without a configured Supabase project, public pages show a backend configuration error and staff sign-in/admin edits remain unavailable; the app does not silently substitute browser-local data.

## Configure staff access

1. In Supabase Authentication settings, set the site URL to the production website and allow the local development URL (`http://localhost:5173`) plus the production `/reset-password` redirect URL.
2. Keep public sign-ups disabled. Create or invite each staff account through Supabase Authentication.
3. For each administrator, set the trusted **App Metadata** role to `admin` in the Supabase Auth user record. Do not use editable User Metadata for authorization.
4. Admin policies in the migration allow writes only when the signed-in JWT has `app_metadata.role = "admin"`. Public visitors can read published matches, active players, public seasons/opponents, and published gallery media. Drafts and archived records are private.

Admin changes are written to Supabase, not to the current browser. Realtime database changes refresh open Hawks Hub pages; the public pages read the same shared records.

## Publish with Vercel

1. Push the project to a Git provider and import that repository in Vercel.
2. Use Vercel's default Vite build settings (`npm run build`, output directory `dist`).
3. Add `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` in the Vercel project's Environment Variables for Production and Preview.
4. Deploy, then set the deployed URL as the Supabase Authentication Site URL and add it to the allowed redirect URLs.
5. Open `/login` on the deployed site to verify staff sign-in, then make a draft change and publish a test record before announcing the site.

`vercel.json` routes direct visits and refreshes on client-side paths back to the app.

## Checks

```bash
npm run build
npm run lint
```

## React Bits Pro components

`components.json` is configured for the React Bits Starter and Pro shadcn registries. Add your license key to a local `.env.local` file (never commit it):

```env
REACTBITS_LICENSE_KEY=your-license-key
```

Then install the bending marquee:

```bash
npx shadcn@latest add @reactbits-starter/bending-marquee-tw
```
