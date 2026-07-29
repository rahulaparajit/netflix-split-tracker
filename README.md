# Plan Ledger

A single-file SPA to track a shared Netflix (or any) subscription plan, its members, and who's paid — hosted free on GitHub Pages, with data stored as `data.json` in the same repo and edited via the GitHub API.

## How it works
- **Readers**: open the page, it fetches `data.json` straight from the GitHub API. Read-only, no login needed.
- **Admin (you)**: click "Admin login," paste a GitHub personal access token (stored only in your browser's `localStorage`). This unlocks an "Edit" button to change plan details, add/remove members, edit amounts, and toggle paid/unpaid. "Save changes" commits the update straight to `data.json` via the GitHub API.

## Setup steps

1. **Create a new GitHub repo** (public), e.g. `plan-ledger`.
2. **Upload these three files** to the repo root:
   - `index.html`
   - `data.json`
   - this `README.md` (optional)
3. **Edit `index.html`** — find the `CONFIG` block near the top of the `<script>` section:
   ```js
   const CONFIG = {
     owner: "YOUR_GITHUB_USERNAME",
     repo: "YOUR_REPO_NAME",
     path: "data.json",
     branch: "main"
   };
   ```
   Fill in your actual GitHub username and repo name. Commit the change.
4. **Enable GitHub Pages**: repo Settings → Pages → Source: "Deploy from a branch" → Branch: `main` / root. Save. Your site will be live at `https://YOUR_GITHUB_USERNAME.github.io/YOUR_REPO_NAME/` within a minute or two.
5. **Edit `data.json`** with your actual plan details (or just leave the starter template and edit it from the page once your token is set up).

## Creating your GitHub token (for admin access only)

1. GitHub → Settings → Developer settings → **Personal access tokens** → **Fine-grained tokens** → Generate new token.
2. Give it a name (e.g. "plan-ledger-admin"), set an expiration (e.g. 90 days — you'll just generate a new one when it expires).
3. Under **Repository access**, choose "Only select repositories" and pick your `plan-ledger` repo.
4. Under **Permissions** → **Repository permissions** → set **Contents** to **Read and write**.
5. Generate, and copy the token somewhere safe — GitHub only shows it once.
6. On your live site, click "Admin login" and paste it in.

**Keep this token private.** Anyone with it can edit your `data.json`. Don't commit it into the repo or share it outside of pasting it into your own browser's admin login prompt.

## Notes & limitations
- This is access control by "who has the token," not individual user accounts — there's no way to identify or restrict specific readers.
- After saving, other viewers may see slightly stale data for a minute or two due to GitHub's API caching.
- Unauthenticated GitHub API requests are rate-limited to 60/hour per IP; this only affects readers refreshing very frequently, not normal use.
- If you'd ever want named logins (e.g., each member sees only their own balance), that would need a proper auth provider (Firebase/Supabase Auth) instead of this token-based approach.
