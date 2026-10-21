# Hawks Hub — Administrator Guide

Hawks Hub uses Supabase for shared records, staff authentication, media storage, and database-enforced permissions. See [README.md](./README.md) for the initial Supabase and Vercel setup.

## 1. Staff access

1. Open `/login` or `/admin` on the deployed site.
2. Sign in with the administrator email and password created in Supabase Authentication.
3. To grant admin access, set the account's trusted **App Metadata** role to `admin`. Do not use editable User Metadata for authorization.
4. Sign out from the top-right of the admin header.

Only authenticated accounts whose JWT contains `app_metadata.role = "admin"` can write records or upload media. Public sign-up should remain disabled. Supabase row-level security protects writes even if someone bypasses the admin UI.

## 2. Match publication

| State | Who sees it | Meaning |
|---|---|---|
| Draft | admins | being entered |
| Pending verification | admins | waiting for an administrator to review |
| Verified | admins | checked and ready to publish |
| Published | everyone | appears on the public site and in statistics |

The admin UI moves a match through draft → pending → verified → published. A match must be finished with both scores before verification or publication. Unpublishing returns it to Verified and removes it from public pages and statistics.

## 3. Players, opponents, and seasons

- **Players:** Add and edit name, sport, position, shirt number, class year, photo, and biography. Archive former players instead of deleting them so existing match records keep their player references.
- **Opponents:** Add names, badges, sport, and city. Opponents referenced by a match cannot be deleted.
- **Seasons:** Add/edit season dates and sport, mark the current season, and associate matches with it. Seasons referenced by a match cannot be deleted.

## 4. Matches

The match editor supports:

1. Sport, season, opponent, date, venue, and competition.
2. Match status and the recorded Hawks/opponent scores.
3. Player participation, football events, basketball player statistics, and period scores.
4. Review, verify, publish, unpublish, archive, restore, and delete actions.

Football events and basketball stats are stored with the match record and reference stable player IDs. Event details do not generate or replace the separately entered final score.

## 5. Gallery and media

Upload an image or video through **Admin → Gallery**. New items are saved as drafts; publish an item to show it on the public gallery. Files use the public `gallery-media` storage bucket, so uploaded media can be viewed by anyone with its URL. Only upload media approved for public use.

## 6. Shared updates and failures

Records are stored in Supabase and are shared across browsers and visitors. Realtime database changes refresh open pages. If a save fails, the admin UI displays the backend error; it does not report a local-only save as successful. Keep the form open and retry after correcting the issue or restoring connectivity.

## 7. Troubleshooting

| Symptom | Cause / fix |
|---|---|
| Backend configuration message | Set `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` locally and in Vercel, then redeploy. |
| No administrator access | Set trusted Supabase Auth App Metadata `role` to `admin`, then sign out and sign back in to refresh the JWT. |
| Cannot publish a match | Mark it finished and enter both scores; then verify it before publishing. |
| A result is missing from public pages | Confirm its state is Published and it is not archived. |
| An opponent or season cannot be deleted | A match refers to it. Rename it or update the match relationship first. |
| Upload rejected | Check Supabase Storage configuration and upload limits. |
| Realtime updates are not appearing | Confirm the migration ran, the tables are in the `supabase_realtime` publication, and the browser can maintain a WebSocket connection. |
