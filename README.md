![PREFLIGHT Score](https://img.shields.io/badge/PREFLIGHT-100%2F100-3ED88B)
# gitspark — find your first commit

A single-page tool that helps beginner developers discover real, open, beginner-friendly
GitHub issues to contribute to — with an optional AI explainer that breaks each issue down
in plain English.

No signup. No login. No backend. No database. It's one HTML file that talks directly to the
GitHub API from your browser.

![hero](producthunt-gallery/01_hero.png)

---

## What it does

1. **Searches GitHub live.** Every issue you see is fetched in real time from GitHub's public
   Search API — nothing is pre-scraped or cached on a server, because there is no server.
2. **Filters the noise.** Beginners drown in thousands of repos before finding one issue worth
   trying. gitspark filters by:
   - **Label** — `good first issue` / `help wanted`
   - **Language** — preset chips (JavaScript, Python, Go, Rust...) or type any language
   - **Status** — "unassigned only", so you don't waste time on an issue someone already claimed
   - **Sort** — newest, recently updated, or most discussed
3. **Explains issues in plain English.** Click "explain this issue" on any card:
   - **Without an API key** — you get a free, instant, rule-based "quick take": what kind of
     change it is (bug fix, docs, UI, etc.), how big it looks, and a first step. No network
     call, no cost, works for everyone immediately.
   - **With your own Anthropic API key** — the same button calls Claude directly from your
     browser and returns a real AI-written explanation of what the issue is asking for and how
     to start. The key is BYOK (bring your own key): it's stored only in your browser's
     `localStorage` and is never sent anywhere except directly to `api.anthropic.com`.
4. **Lets you save issues for later.** Star an issue and it's saved to `localStorage` — no
   account needed, but it also means saved issues don't sync across devices or browsers.
5. **Paginates results.** 10 issues per page, using GitHub's own pagination — so a single
   search never has to load hundreds of issues at once.
6. **Light & dark mode.** Toggle in the top-right; your choice is remembered.

---

## How it actually works (architecture)

```
index.html  (this is the entire app)
  ├── Tailwind CSS (loaded from CDN)
  ├── Vanilla JavaScript (no framework, no build step)
  ├── GitHub Search Issues API   → https://api.github.com/search/issues
  │     called directly from the browser (GitHub's API supports CORS)
  ├── Anthropic Messages API     → https://api.anthropic.com/v1/messages
  │     called directly from the browser using the
  │     `anthropic-dangerous-direct-browser-access: true` header (opt-in BYOK support)
  │     — only if the user has entered their own API key
  └── window.localStorage
        ├── saved issues (bookmarks)
        ├── theme preference (light/dark)
        └── the user's own Anthropic API key (if provided)
```

There is no build step, no `npm install`, no server-side code, and no database. Opening
`index.html` in any modern browser is the entire deployment.

### Why no shared AI key for everyone?

Because this is a single static file with no backend, there's nowhere safe to hide a shared
API key — anyone could open dev tools and read it, then run up the bill. The two honest
options for a project like this are: (a) every user brings their own key, or (b) stand up a
real backend to hold a shared key. This project intentionally stays backend-free, so it does
(a), and falls back to a free non-AI explanation for everyone who doesn't want to set up a key.

### Why doesn't the GitHub rate limit run out immediately?

Because every visitor's browser calls GitHub directly with their own IP — usage is never
pooled through a shared server. Unauthenticated GitHub Search API requests are limited to
~10 requests/minute per IP; if that becomes a real problem for you personally, GitHub raises
it significantly for authenticated requests.

---

## Running it locally

No installation needed:

```bash
# just open it
open index.html          # macOS
start index.html         # Windows
xdg-open index.html      # Linux
```

Or serve it with any static server if you prefer (optional, just for a nicer local URL):

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

> **Note:** `localStorage` (saved issues, theme, API key) only works when the file is opened
> as a real page in your own browser — not inside an embedded preview/sandbox.

## Deploying it

Since it's static files, any static host works — for example **GitHub Pages** (push this
folder to a repo, enable Pages in Settings) or **Netlify/Vercel** (drag-and-drop deploy).
Keep `index.html`, `favicon.png`, and `og-image.png` in the same folder so the icon and the
social preview image resolve correctly once it's live.

## Files in this project

| File | Purpose |
|---|---|
| `index.html` | The entire application |
| `favicon.png` | Browser tab icon |
| `og-image.png` | Social preview image (Twitter/Slack/Discord/Product Hunt link unfurls) |
| `producthunt-gallery/` | 4 gallery images sized for a Product Hunt launch post |

## Tech

- HTML5 + [Tailwind CSS](https://tailwindcss.com/) (CDN, no build step)
- Vanilla JavaScript — no framework
- [GitHub Search Issues API](https://docs.github.com/en/rest/search#search-issues-and-pull-requests)
- [Anthropic Messages API](https://docs.claude.com/en/api/messages) (optional, user-provided key)
- `localStorage` for all persistence

## Limitations

- GitHub's Search API only exposes the first **1,000 results** for any query, regardless of
  how large the total count shown is — this is a GitHub limitation, not a bug here.
- Unauthenticated requests are rate-limited by GitHub (~10/min per IP).
- Saved issues and settings live in one browser only; there's no account or sync.
- The AI explanation is only as good as the issue's own description — vague issues get vague
  explanations.

## License

Use it, fork it, ship it — no license restrictions implied by this README; add a `LICENSE`
file (MIT is a reasonable default for a project like this) before publishing if you want one.
