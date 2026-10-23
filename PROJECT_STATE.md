# ComicsTracker — Project State

> **Next chat name: `ComicsTracker ver 5`**
> Claude: your FIRST reply in a new chat must start with "Rename this chat to: **ComicsTracker ver 5**" (you can't rename chats yourself). In your first commit this session, bump the number here by 1.

**State version:** 5 (moved into the repo 2026-10-22 by the conductor chat "Life Plan Roadmap ver 3")
**Where this lives:** `PROJECT_STATE.md` in the repo. New chats `git pull` and read it; no uploads needed.
Single-file app, live and working. `pulllist-methodology.md` holds the PullList research method (not in this repo yet; attach it when doing PullList work).

## ⚙ Session Rules (every chat that works on this app follows these)

**1. Pick the cheapest model that can do the job.** At the start of every session Claude names the model it recommends and the estimated size (below). If the chat is on a pricier model than needed, Claude says so before starting.

| Model | Use for |
|---|---|
| **Haiku** | State-file edits, text or label changes, small data packs, lookups, "where is X" questions |
| **Sonnet** (default) | Features, bug fixes, refactors, data processing (e.g. novel volumes, PullList), most app work |
| **Opus** | Only for new-app builds, cross-app architecture, or a bug Sonnet has failed on twice. Claude says why before using it |

**2. Every push updates this file in the same commit.** No exceptions, including PR branches and one-line fixes.
- `git pull` before starting, so you build on the latest state.
- Add a dated line to the change log (what changed and why, in plain words) and update any section the change made wrong.
- If a session ends without a push, nothing changes here.

**3. Size the work first, and plan the pause points.**
- Before starting, Claude lists the steps and sizes the job: **S** (under ~20k tokens), **M** (~20–60k), **L** (over ~60k).
- **M and L jobs are split into checkpoints.** Each checkpoint ends with a commit and a state update, so a cut-off session loses nothing.
- **Pause** when usage is around half the window, when the chat has grown long (heavy file reads, ~30+ turns), or before a step that won't finish in what's left. Claude can't see the remaining credit directly, so on L jobs it asks Marvin to check usage at each checkpoint.
- **On pause,** Claude writes a `▶ RESUME HERE` block at the top of this file: what's done, the exact next step, branch and files, and the first command to run. It also gives Marvin a one-line restart prompt ("Continue <App> ver X from RESUME HERE"). The next session starts from that block and deletes it once the step is done.

**4. Chat naming.** Use the **Next chat name** line at the top of this file; tell Marvin in the first reply, then bump it.

## Latest changes (Oct 2026, conductor chat)
- **Sprints 13.3 + 13.2 + `lp-sso` (2026-10-23, conductor chat "Life Plan Roadmap ver 7"):** already on the shared OAuth client, so it now uses and publishes the shared sign-in (`lp-sso` in localStorage: a valid `drive.file` token from Life Plan or another app means no second sign-in; sign-out clears it). Added a manual "Sign-in window not opening? Try another way" link (uses the existing redirect flow) and an iPhone "Share → Add to Home Screen" hint on the sign-in card (iOS can't install web apps from a prompt). 13.2: "← Life Plan" link in the header, `viewport-fit=cover`, safe-area bottom padding, shared theme tokens `--lp-bg/--lp-surface/--lp-text/--lp-muted/--lp-accent`. `sw.js` cache `comicstracker-v3`. Not tested live.
- **Mobile layout (2026-10-23, ver 4 chat):** the header stats + 5 buttons row didn't wrap, so the whole page scrolled sideways on phones. Header now stacks (stats, then wrapping button row); the 6 tabs are a single 6-column row of icon-over-label on phones (Reading Order shortens to "Reading"); on sm+ screens the tab bar is the old scrolling row. `body{overflow-x:clip}` is a safety net. Not yet checked on a real phone by Claude; Marvin to confirm, and send a screenshot of any view that still overflows.
- **Save fix + Save button (2026-10-23, ver 4 chat):** bug: `persist()` only rewrote `comicstracker-archive.json` when the *set* of archived series changed, so renaming/editing an archived series (e.g. the Marvel migration) looked fine but reverted on reload. Now the archive is rewritten whenever its content changed (compared against `lastArchiveJson`; first save after load writes it once). `mutate()` returns the save promise; errors show the reason. New **Save** (floppy) button in the header calls `persist(true)` and toasts "Saved to Drive ✓". Refresh tooltip now warns it discards unsaved changes. Saving is otherwise automatic after every change.
- **iPhone redirect sign-in (2026-10-23, ver 4 chat):** when the app runs as an installed iOS home-screen app, Connect uses a full-page Google OAuth redirect (implicit flow) instead of the GIS popup, and reads the token from the URL hash (state-checked, hash cleared). Desktop and browser tabs are unchanged. **Needs `https://marvyeo.github.io/ComicsTracker/` added under Authorized redirect URIs on the OAuth client in Google Cloud Console, or iPhone sign-in will fail with redirect_uri_mismatch.**
- **`upcoming[]`** on every save: releases in the next 30 days (`type: release`); issues in active events are high priority. Shown on the Life Plan dashboard.
- **PullList is a scheduled task:** "PullList monthly solicitations" runs on the 24th at 8:46am SGT and writes `solicitations_YYYY-MM.json` to the Drive folder "PullList" (push + email summary). Marvin imports it by hand (his choice): Import → Upload PullList .json → review → Import selected.
- **Marvel name migration (done 2026-10-23):** all Marvel `Vol. X` series renamed to `Name (LaunchYear)` in Drive; one-time tool removed from the app (PR #2 code deleted in the ver 4 chat).
- **PR #1 (merged 2026-10-23):** newly solicited series start unticked on PullList import; series already tracked stay ticked.
- **Installable (21 Oct):** `manifest.json`, network-first `sw.js` (cache `comicstracker-v2`), ecosystem icons. iPhone redirect sign-in fallback added 2026-10-23 (see below).
- **Manifest id fixed (22 Oct):** `"id": "/ComicsTracker/"`.

## Stack & hosting
- `index.html` is the app, plus `manifest.json`, `sw.js`, `icons/`. Vanilla JS, Tailwind Play CDN, Lucide icons, no build step.
- Auth: Google Identity Services, `drive.file` scope (single-user).
- Storage: two JSON files in user's own Drive:
  - `comicstracker-data.json` — active series/issues/events/eras/families
  - `comicstracker-archive.json` — completed+read+owned series moved out of main file
- Hosted on GitHub Pages. Setup steps in `README.md`.
- **Client ID (live, do not change):** `669494100250-gvromuq72r7j0djmkd4vdbdq5damncj7.apps.googleusercontent.com`

## Data schema
```js
DATA = {
  series: [{ id, name, publisher, status,   // status: ongoing|limited|completed|upcoming
             eras:[], families:[], archived }],
  issues: [{
    id, seriesId,
    number,          // STRING always (handles "3.1", "0", "27.NOW" etc)
    title,           // "" if no story title — never invented
    releaseDate,     // "YYYY-MM-DD" on-sale date, "" if unknown — never invented
    dateHistory: [{date, note, changedOn}],  // appended on reschedule, never overwritten
    eventIds: [],
    purchased: bool, read: bool,  // read=true forces purchased=true
    readOrder: int|null           // pins position in Reading Order tab, scoped per-event
  }],
  events:   [{ id, name, color }],
  eras:     [{ id, name, color }],
  families: [{ id, name, color }]
}
```
Status derivation: `read` → read · `purchased` → owned · neither → want.

## Full feature list (all live)

### Library
- Grouped by Publisher / Era / Family (select via Group dropdown).
- Publisher/group sections are collapsible (chevron toggle on header).
- Sort within group: A→Z, Year ↑, Year ↓.
- **Status filter row:** All / Read / Owned / Want — hides non-matching series,
  shows only matching issues within each series.
- **Series-type hide toggles:** Undated / Upcoming / Completed / Limited / Archived
  (Archived defaults ON — hides archived series from main view).
- **Week filter:** date picker — pick any day, snaps to that Mon–Sun window and
  flips the container into a flat ungrouped release list (issues sorted by date
  then series name, grouped by publisher header). Respects all active status/type
  filters. Clear button resets. Designed for "what shipped this week that I haven't
  pulled yet" — combine with Status=Want for that exact view.
- Per-series actions (expand accordion): Add issue, Mark all owned, Mark all read,
  Cycle status (upcoming→ongoing→limited→completed), Rename, Archive toggle, Delete.
- Archive checkbox enabled only when status=completed; moves series to archive file.

### Tracker
- Upcoming issues grouped by month; issues without dates grouped separately.
- Reschedule via inline date edit — appends old date to `dateHistory`, shows
  "delayed" badge with original date on hover.

### Reading Order
- Publisher = primary filter (dropdown).
- Event = secondary filter — only shows events with issues under selected publisher,
  sorted by that event's earliest issue date; auto-resets when publisher changes.
- Owned-only toggle (default) or show all.
- `readOrder` integer pins an issue's position; unpinned issues fall back to release date.
- "Next up" banner highlights first unread issue in current filtered list.

### Events
- Publisher filter dropdown (shows only events with issues under that publisher).
- Per-event: Mark all owned / Mark all read / Delete.
- Create new events (auto-assigns color from palette).

### Groups
- Manage Eras and Families (create/rename/delete/recolor).
- Assign series to eras and families from the series accordion in Library.

### Import
- Accepts PullList `.json` file upload or pipe-delimited paste.
- **Merges on `(seriesName, number)`**: unions event tags, fills blank fields only,
  never touches `read`/`purchased`/`dateHistory`.
- Preview with per-row include/exclude, inline editing before commit.
- Fuzzy series name matching on blur (Levenshtein-based) — flags likely matches.
- Auto-appends `(YYYY)` to new series names based on earliest issue date in batch.

### Export / Drive
- Export downloads `comicstracker-records.json` (full DATA) for PullList reference.
- Drive connection dot in header: Connected/Disconnected/Checking, auto-checks 60s.
- Refresh button re-pulls from Drive (for cross-device sync).

## Naming conventions (critical for correct Import merge behaviour)
**Rule (confirmed by Marvin, 2026-10-22): every publisher, Marvel included, uses `Series Name (LaunchYear)`.** LaunchYear = the year that volume's #1 went on sale. The year says when a run is from without ambiguity, which `Vol. X` doesn't. Never use `Vol. X`.
- Examples: `"Avengers (2013)"` (Hickman), `"Avengers (2010)"` (Bendis), `"Thunderbolts (1997)"`, `"Nightwing (2016)"`, `"Mighty Morphin Power Rangers (2016)"`.
- Same-year relaunches (rare): no rule yet; flag to Marvin if one appears.
- The PullList scheduled task prompt was updated to this rule on 2026-10-22.
- **Migration:** done 2026-10-23 (all Marvel series are on `(LaunchYear)` names). Any new `Vol. X` name that appears is a mistake; fix it by hand.
- One-shots/minis get their own `seriesName`; never fold into a parent's numbering.

## PullList (research workflow)
See `pulllist-methodology.md` for full detail. Summary: Claude-side workflow
producing `.json` files in ComicsTracker's Import schema. Publisher-by-publisher,
events-first. Progress log is in that file (currently: Metal, BOOM Power Rangers,
Marvel Eras 1–4 [2004–2015]; next: Secret Wars 2015, then DC from Blackest Night).

## Known trade-offs
- `readOrder` is scoped per-event: an issue in 2 events only saves one position.
- Banner events (Dark Reign, Heroic Age, etc.) are always partial after an events-pass;
  full series-run pulls (step 6 of methodology) backfill them.
- localStorage only caches OAuth token and Drive fileIds; all comic data lives in Drive.

## Open items
- [x] Marvin reported the redirect URIs done on 2026-10-23 (Roadmap ver 7); iPhone sign-in still untested. (Was: add `https://marvyeo.github.io/ComicsTracker/` to Authorized redirect URIs (Google Cloud Console → Credentials → the OAuth client), then test sign-in on the installed iPhone app. Code is done and pushed.)

## Change log
- **ver 2 (2026-10-22):** moved into the repo as `PROJECT_STATE.md`; added Session Rules; recorded `upcoming[]`, the PullList task, PR #1, installability and the manifest id fix.
- **ver 3 (2026-10-23):** merged PR #1 (new series start unticked on PullList import) into main. State file updated here, since the PR itself didn't touch it.
- **ver 3 (2026-10-23, later):** added one-time Marvel `Vol. X` → `(LaunchYear)` migration tool on the Import tab (PR #2, merged). Not yet run on Drive data; once run, the tool can be removed.
- **ver 4 (2026-10-23):** added iPhone redirect sign-in fallback in `index.html` (installed iOS app only). Marvel migration still not run: needs Marvin's signed-in app.
- **ver 4 (2026-10-23, later):** fixed archive-file saves not persisting edits to archived series; added header Save button; migration Apply now confirms the save.
- **ver 4 (2026-10-23, end):** Marvel migration run and verified by Marvin; migration tool removed from `index.html`. Only open item left: the Google Cloud redirect URI for iPhone sign-in.
- **2026-10-23 (conductor, Sprints 13.3 + 13.2):** shared `lp-sso` sign-in, manual redirect link, iPhone Add to Home Screen hint, "← Life Plan" link, safe area, theme tokens; `sw.js` cache `comicstracker-v3`. Not tested live.
- **ver 4 (2026-10-23, mobile):** fixed sideways scrolling on phones (header wrap, compact tab bar, overflow guard).
