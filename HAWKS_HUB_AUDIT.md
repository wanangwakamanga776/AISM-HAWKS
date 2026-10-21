# Hawks Hub — Architecture Audit

## 1. Platform reality (read this first)

The brief asks for React + Vite + Tailwind **+ Supabase**. This app runs on **Base44**, whose backend is platform-owned: database (entities), authentication, and row-level security are provided by Base44, not Supabase. Supabase credentials, SQL migrations, and `supabase-js` cannot be used here, and a Supabase project would require its own hosting.

The **architecture** requested is preserved in full — one domain model, one repository layer, one statistics engine, admin → repository → database → public pages, publication gating, and database-enforced authorization. Only the database vendor differs (Base44 entities instead of Postgres tables), which is a platform constraint rather than a design choice.

There is **no LocalStorage fallback**. Deliberate decision: a silent local copy would let a failed save look successful. Every write either reaches the shared database or throws an error the UI shows.

## 2. Actual data flow

```
Administrator (Base44 account, role = admin)
   ↓  admin UI (matches / players / opponents / seasons / gallery)
src/services/dataService.js        ← the only module that touches the database
   ↓  validation + publication-state transitions + audit entry
Base44 database (entities: Match, Player, Opponent, Season, GalleryItem, AuditLog)
   ↓  reads (public pages ask for published records only; RLS refuses drafts)
Home · Football · Basketball · Gallery · Statistics · Player profiles · Match pages
   ↓
src/utils/statisticsEngine.js      ← every figure on the site is derived here
```

## 3. Domain model

| Entity | Purpose | Key fields |
|---|---|---|
| `Player` | stable identity across matches/seasons | name, sport, position, shirt_number, photo_url, bio, class_year, is_active, archived |
| `Opponent` | opponent identity + badge | name, short_name, sport, badge_url, city |
| `Season` | groups matches and statistics | name, sport, start/end date, is_current |
| `Match` | single source of truth for a game | sport, opponent_id, season_id, competition, match_date, venue, status, publication_status, hawks_score, opponent_score, period_scores, participants, events, player_stats, notes, verified_by/on, published_on, archived |
| `GalleryItem` | real media | title, media_url, media_type, sport, captured_on, is_published |
| `AuditLog` | who changed what | action, entity_type, entity_id, summary, actor, status_from/to |

**Design decisions**

- **Events and stat lines live on the match record** (arrays of objects), referencing players by stable id. This makes publication gating automatic: a draft match's events cannot leak, because the whole record is invisible until published. It also avoids the duplicate-counting risk of a parallel events table.
- **Match status vs publication status are separate**: `scheduled / finished / postponed / cancelled` vs `draft → pending → verified → published`. A finished match is not published automatically.
- **Player deletion is never offered** — only inactive + archived, so historical events never dangle.

## 4. Confirmed working (verified in the running app)

- Public routes: Home, Football, Basketball, Gallery, Statistics, Match archive, Player profile, Match detail, plus Login/Register/Forgot/Reset.
- Admin route `/admin`, gated by authentication **and** the admin role.
- End-to-end test recorded a match through the admin UI: opponents/seasons/lineup/event → submit for verification → verify → publish. The record persisted with `publication_status = published`, score `3–1`, one goal event at 12', two participants, `verified_by` set.
- The published result appeared on Football (record 1-1-0-0, 3 points, W badge), on the match page (event timeline), and in Statistics (leaderboards + full squad).
- **N/A vs zero**: in the same table, the player with a recorded goal shows `1`; the player with no recorded event shows `N/A` — not `0`.
- Player creation through the admin UI persisted correctly and the success banner appeared.
- Deleting the test records removed the published result and statistics, restoring the empty states.
- Validation surfaced in-place ("Choose or create an opponent.", "Choose the Hawks player for this event.") and blocked the save.
- No console errors on any route; no horizontal overflow at mobile width.

## 5. Security

- Authentication: Base44 auth (email/password and Google), session handled by the platform.
- Authorization: enforced **in the database** by row-level security on every entity, not just in the UI.
  - `Match`: read = published **or** admin; create/update/delete = admin only.
  - `GalleryItem`: read = published **or** admin; writes admin only.
  - `Player`, `Opponent`, `Season`: public read (roster data is public), writes admin only.
  - `AuditLog`: admin only, all operations.
- Public visitors therefore cannot read drafts, and cannot write anything even with a direct API call.
- No secrets, passwords, or keys exist in the frontend bundle; the app uses the platform SDK with the signed-in user's token.

## 6. Known gaps / not implemented (honest list)

1. **Roles**: only the built-in `admin` / `user` roles. The suggested extra "Data Entry" (edit but cannot publish) role is **not** implemented — it would need a separate User field plus a server-side check, and is deliberately not faked.
2. **Audit log viewer**: entries are written for create/update/publish/archive/delete, but there is no admin screen to browse them yet.
3. **Real-time**: no websocket subscriptions. Public pages read fresh data on every load/navigation, so a published result appears immediately on refresh — reliable, but not live-push.
4. **Gallery uploads** and player photos are stored in public storage (permanent public URLs), since they are meant for the public gallery.
5. **Build command / type-check**: this environment has no shell, so `npm run build` cannot be executed here. Correctness was verified by loading every route in a live browser instead (no build or runtime errors reported).
6. **Non-admin write refusal** was verified by rule inspection and configuration, not by driving a second account — no second identity is available in this environment.
7. **Supabase-specific items** (SQL migrations, RLS policies in Postgres, `VITE_SUPABASE_*` env vars) do not apply and are not provided.

## 7. Prioritised repair plan (ongoing)

1. Add the audit log viewer (data already collected).
2. Add a "Data Entry" role with publish separation, once a role model beyond Base44's admin/user is available.
3. Season-scoped squad participation records (a player appearing in a season without a match).
4. Bulk media upload and image cropping for the gallery.