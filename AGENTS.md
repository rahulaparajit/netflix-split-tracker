# AGENTS.md

Context file for AI coding agents (Claude Code, Cursor, etc.) working in this repo.

## Project

**plan-ledger** — a single-file SPA for tracking a shared subscription plan (e.g. Netflix) split across multiple members: who owes what, and who's paid. Deployed free on GitHub Pages, with **no backend server** — the GitHub repo itself acts as the database.

## Architecture

- `index.html` — the entire app. Vanilla HTML/CSS/JS, no build step, no framework, no npm dependencies. Fonts loaded from Google Fonts CDN.
- `data.json` — the "database." A single JSON file at repo root holding plan details and the member list.
- Reads: unauthenticated `GET` to the GitHub Contents API (`api.github.com/repos/{owner}/{repo}/contents/{path}`), decoded from base64 client-side.
- Writes: authenticated `PUT` to the same endpoint, using a GitHub personal access token pasted in by the admin and stored in the browser's `localStorage` (key: `planLedger_token`). Never stored in the repo or in code.
- Access control is binary and token-based: anyone can read; only someone with the admin token can write. There are no individual user accounts.

## Data shape (`data.json`)

```json
{
  "plan": {
    "name": "string",
    "totalCost": "number",
    "currency": "string (symbol, e.g. ₹ or $)",
    "billingDate": "YYYY-MM-DD",
    "notes": "string"
  },
  "members": [
    { "id": "string", "name": "string", "share": "number", "paid": "boolean" }
  ],
  "lastUpdated": "ISO 8601 timestamp"
}
```

## Conventions & constraints to respect

- Keep it a **single self-contained HTML file** where practical — this is a deliberate choice (see project preferences below), not an oversight. Don't introduce a build step (webpack/vite/etc.) or split into a JS framework without being asked.
- No backend code. Any new persistence/feature must work within "static HTML + GitHub Contents API" or be flagged as requiring a different hosting approach.
- The `CONFIG` object near the top of the `<script>` block in `index.html` (`owner`, `repo`, `path`, `branch`) is meant to be edited per-deployment. Don't hardcode secrets there — it's committed to a public repo.
- Money amounts and dates render via `JetBrains Mono`; plan title/headings via `Space Grotesk`; body text via `IBM Plex Sans`. Keep this pairing when adding UI.
- Dark ledger/receipt visual style (see `:root` CSS variables in `index.html` for the token palette). Avoid generic light-theme dashboard defaults.
- The segmented "split bar" (proportional bar showing each member's share, solid when paid / hatched when unpaid) is the signature UI element — preserve it when refactoring the member view.
- Base64 encode/decode for `data.json` content must stay UTF-8 safe (see `b64EncodeUnicode` / `b64DecodeUnicode` helpers) — don't replace with plain `btoa`/`atob`.

## Known limitations (by design, not bugs)

- No per-reader identity; "access control" = has-the-token vs. doesn't.
- Saves may appear stale to other viewers for a minute or two due to GitHub API/CDN caching.
- Unauthenticated reads are rate-limited to 60 requests/hour per IP by GitHub.
- Not suitable for high-frequency writes (each save is a git commit).

## Broader user preferences (apply to sibling/future projects too)

- Prefers free static hosting (GitHub Pages) with no dedicated backend.
- One repo per SPA, each living at its own `/<repo-name>/` GitHub Pages subpath.
- Favors reusable, generically-named projects over one-off, tool-specific names.
- Cares about intentional visual design, not just function — avoid templated-looking defaults.
