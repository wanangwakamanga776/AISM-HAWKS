# Hawks Hub — Administrator Guide

## 1. Signing in

1. Go to `/login` (footer link: "Administrator sign in"), or `/admin` and follow the prompt.
2. Sign in with your administrator account (email + password, or Google).
3. You land in the **admin workspace**, which is completely separate from the public site.
4. Sign out from the top-right of the admin header.

**Access rules**

- Anyone can read the public site.
- Only accounts with the *admin* role can open `/admin` or change records. This is enforced by the database, not just by hiding buttons — a direct API call is refused too.
- A signed-in non-admin sees "No administrator access" and can still browse the public site.
- Sessions expire per the platform's normal session rules; signing out ends the session immediately.

## 2. The publication workflow

Every match moves through four states, and only the last is public:

| State | Who sees it | Meaning |
|---|---|---|
| Draft | admins | being entered |
| Pending verification | admins | numbers entered, waiting for a second pair of eyes |
| Verified | admins | checked as accurate |
| Published | **everyone** | appears on the site and in all statistics |

You cannot skip a step (draft → published is refused). A match must be **finished with both scores** before it can be published. Unpublishing returns it to *Verified* and removes it from the public site and from every total.

## 3. Players

**Admin → Players**

- **New player**: name, sport, position, shirt number, class year, photo, biography.
- Leave anything you don't actually know blank — blanks display as N/A, never as a guess.
- Editing a name or photo never breaks match history: matches reference the player's id, not their name.
- **Archive** a departing player instead of deleting: they disappear from the roster but stay in past results. There is intentionally no delete button.
- Use the search box and sport filter to find players quickly.

## 4. Opponents and seasons

**Admin → Opponents**: name, short name, sport, city, badge. Opponents used by a recorded match cannot be deleted (the app tells you).

**Admin → Seasons**: one season can be marked *current*; marking a new one replaces the old flag. Seasons with matches attached cannot be deleted, so old records stay intact.

## 5. Recording a match

**Admin → Matches → New match**

1. **Overview** — sport, season, opponent (or create one inline), date, venue (home/away/neutral), competition.
2. **Result** — match status (`scheduled`, `finished`, `postponed`, `cancelled`) and the final score. Leave scores blank until the result is known; blanks are published as "score not recorded", never 0–0.
3. **Lineup** — tap the players who featured. Appearances are counted from this list.
4. **Events (football)** — goal, assist, yellow card, red card, substitution, own goal, penalty missed. Choose the Hawks player and the minute; the second dropdown is for the assisting player or the player coming off.
   - Events **do not** create the score. Enter the score separately and keep the two consistent.
   - An event must be attributed to a Hawks player; if you don't know who scored, don't add the event.
5. **Player statistics (basketball)** — per-player points, rebounds, assists, steals, blocks, turnovers for that game. Blank = not recorded (N/A), not zero.
6. **Quarter scores (basketball)** — add Q1–Q4/OT rows for the period breakdown.
7. **Verification & publishing** — submit for verification, then verify, then publish. The status history (who verified, when published) is shown in the editor.
8. **Notes** — shown on the public match page.

Quick actions in the match list let you **Verify** or **Publish** without reopening the editor. **Archive** hides a record from the site while keeping it in history; **Delete** is permanent and asks for confirmation.

## 6. Gallery

**Admin → Gallery**

1. Choose a file (images or short video), give it a title, category and date, then **Upload**.
2. Uploads are **drafts**. Press **Publish** to show an item on the public gallery; **Unpublish** hides it again.
3. Files are stored publicly (anyone with the link can open them) because the gallery is public. Only upload media you are happy to publish.

## 7. What happens when a save fails

- Nothing is silently discarded. A red banner explains the problem (missing opponent, missing score, illegal status change, permission error, network problem).
- The form keeps your input so you can fix the issue and retry.
- Buttons are disabled while a save is in flight, preventing accidental double submissions.
- There is no offline mode: if the database cannot be reached, you get an error rather than a "saved" message that only lived in the browser.

## 8. Reading N/A versus zero

- `0` means **a recorded zero** (a match played with no goals conceded).
- `N/A` means **never recorded** — the app refuses to pretend it was zero.
- Averages are computed only from rows that actually recorded that value.
- A result appears in records and statistics only when the match is *finished*, has *both scores*, and is *published*.

## 9. Troubleshooting

| Symptom | Cause / fix |
|---|---|
| "No administrator access" | Your account isn't an admin. Ask an existing admin to change your role in the app's user settings. |
| Can't publish | Match status must be `finished` with both scores, and the record must first be verified. |
| A result isn't on the public site | Check its publication state — only *Published* is public. |
| Statistics look low | Only published, finished matches with a recorded score are counted; drafts and blanks are excluded. |
| A stat shows N/A instead of 0 | That stat was never entered for that player/game. Enter it in the match editor. |
| "Opponent is used by recorded matches" | The opponent is referenced by history and cannot be deleted. Rename it instead. |
| Photo/gallery upload rejected | Check file type and size; only images and video are accepted. |

## 10. Deployment

- Build and publish through the Base44 dashboard. There is no separate server or environment file to configure: the database, auth, and storage are part of the app.
- Required configuration: none beyond the platform's own — no API keys are stored in the app.
- After publishing, `/admin` is reachable at your app domain; visitors only ever see published records.
- To give someone admin rights, set their role to *admin* in the app's user management.