# AGENTS.md

Context file for AI coding agents working in this repo.

## Project

**Netflix Split Tracker** — a single-file SPA for tracking a shared subscription plan (e.g. Netflix) split across multiple members: who owes what, who's paid, per billing cycle. Deployed free on GitHub Pages, with **no backend server** — the GitHub repo itself acts as the database.

## Architecture

- `index.html` — the entire app. Vanilla HTML/CSS/JS, no build step, no framework, no npm dependencies. Fonts loaded from Google Fonts CDN.
- `data.json` — the "database." A single JSON file at repo root holding plan details, members, and billing-cycle history.
- `sw.js` + `manifest.json` + `icon.svg` — PWA shell. The service worker caches static assets (network-first fallback to cache) but **must keep bypassing `api.github.com`** requests so data reads are never served stale from cache.
- Reads: unauthenticated `GET` to the GitHub Contents API (`api.github.com/repos/{owner}/{repo}/contents/{path}`), cache-busted with `&_=${Date.now()}`, decoded from base64 client-side.
- Writes: authenticated `PUT` to the same endpoint. The token is pasted in by the admin and stored **both** in a 30-day `SameSite=Strict` cookie and `localStorage` (key: `planLedger_token`). Never stored in the repo or in code. PUTs include the last-read `sha` for conflict handling.
- Access control is binary and token-based: anyone can read; only someone with the admin token can write. No individual user accounts.

## Data shape (`data.json`)

```json
{
  "plan": {
    "name": "string",
    "totalCost": "number",
    "currency": "string (symbol, e.g. ₹ or $)",
    "billingDate": "YYYY-MM-DD",
    "notes": "string",
    "status": "string (e.g. \"active\")"
  },
  "registeredMembers": [
    { "id": "string", "name": "string", "active": "boolean", "joinedAt": "ISO 8601" }
  ],
  "members": [
    { "id": "string", "name": "string", "share": "number", "paid": "boolean", "paidAt": "ISO 8601|null" }
  ],
  "history": [
    { "id": "string", "cycleLabel": "string", "billingDate": "YYYY-MM-DD",
      "totalCost": "number", "currency": "string", "archivedAt": "ISO 8601",
      "status": "string (active|future|archived)",
      "members": [ "(same shape as members)" ] }
  ],
  "lastUpdated": "ISO 8601 timestamp"
}
```

- `history[]` holds archived/future billing cycles; each cycle snapshots its own `members` array. The current cycle is `history[?]` with `status:"active"` (see `ensureSingleActiveCycle`).
- `registeredMembers` is the master roster; `members` is the current cycle's share book. The registry panel is a separate admin flow from editing the cycle.
- `loadData` normalizes missing `members`/`history`/`registeredMembers` to `[]` and auto-seeds `registeredMembers` from `members` on first load — don't remove that backfill.

## Conventions & constraints to respect

- Keep it a **single self-contained HTML file** where practical — a deliberate choice. Don't introduce a build step (webpack/vite/etc.) or a JS framework without being asked.
- No backend code. Any new persistence/feature must work within "static HTML + GitHub Contents API" or be flagged as requiring a different hosting approach.
- The `CONFIG` object at line ~798 in `index.html` (`owner`, `repo`, `path`, `branch`) is meant to be edited per-deployment. It's committed with real values to a public repo — don't hardcode secrets there.
- Money amounts and dates render via `JetBrains Mono`; headings via `Space Grotesk`; body text via `IBM Plex Sans`. Keep this pairing when adding UI.
- Styling is a light/dark theme pair driven by `data-theme` on `<html>` (`:root` = dark palette, `[data-theme="light"]` overrides). Persisted via `THEME_KEY` in localStorage, default light. When adding UI, use the `--bg/--surface/--ink/--accent/--gold/--line` CSS variables (see `:root`); avoid hardcoded colors and generic light-theme dashboard defaults.
- The segmented "split bar" (proportional bar showing each member's share, solid when paid / hatched when unpaid) is the signature UI element — preserve it when refactoring the member view.
- Base64 encode/decode for `data.json` must stay UTF-8 safe (see `b64EncodeUnicode` / `b64DecodeUnicode` helpers) — don't replace with plain `btoa`/`atob`.
- Admin UI buttons (registry, cycle manager, etc.) are hidden behind admin login; token validation runs on load via `validateToken`.

## Known limitations (by design, not bugs)

- No per-reader identity; "access control" = has-the-token vs. doesn't.
- Saves may appear stale to other viewers for a minute or two due to GitHub API/CDN caching (and the service worker may hold a stale `data.json` — the API calls are cache-busted for this reason).
- Unauthenticated reads are rate-limited to 60 requests/hour per IP by GitHub.
- Not suitable for high-frequency writes (each save is a git commit).

## Broader user preferences (apply to sibling/future projects too)

- Prefers free static hosting (GitHub Pages) with no dedicated backend.
- One repo per SPA, each living at its own `/<repo-name>/` GitHub Pages subpath.
- Favors reusable, generically-named projects over one-off, tool-specific names.
- Cares about intentional visual design, not just function — avoid templated-looking defaults.
