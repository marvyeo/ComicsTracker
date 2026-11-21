# ComicsTracker

A single-file comics tracker. Vanilla JS, Tailwind Play CDN, Lucide icons, no build step. Hosted on GitHub Pages; your data lives in your own Google Drive.

Live: https://marvyeo.github.io/ComicsTracker/

## Files
- `index.html` — the whole app
- `manifest.json`, `sw.js`, `icons/` — installable app (network-first service worker)
- `PROJECT_STATE.md` — project memory: features, schema, change log, session rules
- `pulllist-methodology.md` — the PullList research workflow that produces import files
- `CLAUDE.md` — instructions for Claude Code sessions

## Setup (from scratch)
1. **Hosting:** repo Settings → Pages → deploy from `main` / root.
2. **Google Cloud:** create a project, enable the **Google Drive API**, configure the OAuth consent screen, and create an **OAuth client ID (Web application)**.
   - Authorized JavaScript origins: `https://marvyeo.github.io`
   - Authorized redirect URIs: `https://marvyeo.github.io/ComicsTracker/` (needed for the installed iPhone app's redirect sign-in)
3. Put the client ID in `index.html` (current live ID is recorded in `PROJECT_STATE.md`; do not change it casually).
4. Open the site, **Connect** (scope: `drive.file`). The app creates `comicstracker-data.json` and `comicstracker-archive.json` in your Drive.

## Using it
- **Import:** upload a PullList `.json` (or paste pipe-delimited rows), review, untick, import. Merges on `(seriesName, number)`; never touches read/purchased flags.
- **Export:** header download button writes `comicstracker-records.json` for PullList reference.
- **Save:** saving is automatic after every change; the header Save button forces it.
- Mobile: install to the home screen (iPhone: Share → Add to Home Screen).
