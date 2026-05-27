# HackClutch — Valorant Esports Dashboard

A clean, fast Valorant esports dashboard built with vanilla HTML, CSS, and JavaScript. No frameworks, no build tools — just the web platform.

## Features

### Pages

| Page | Description |
|------|-------------|
| **News** | Latest esports headlines from VLR.GG |
| **Matches** | Live scores, upcoming matches, and results with series/region filters |
| **Rankings** | Team rankings by region |
| **Stats** | Player statistics with region, timespan, org, and agent filters |
| **Players** | Player directory with region filter |
| **Teams** | Team directory with region filter |
| **Events** | Event browser with status tabs and series/region filters |

### Highlights

- **Hash-based SPA routing** — `#/matches`, `#/team/123`, `#/player/456`
- **Dual API backend** — merges data from two APIs for comprehensive coverage
- **Smart caching** — localStorage with per-key TTL, stale-while-revalidate strategy
- **Dark mode** — automatic via `prefers-color-scheme`, with theme-aware VLR logos
- **Responsive** — works on desktop and mobile
- **Lazy-loading avatars** — `IntersectionObserver` for stats page player images
- **Multi-column sort** — click column headers to stack sort conditions
- **Search** — teams, players, events, and series

## Quick Start

```bash
# Start the local dev server (includes CORS proxy)
python3 server.py

# Open in browser
open http://localhost:8080
```

If port 8080 is in use:

```bash
lsof -ti:8080 | xargs kill -9
```

## Deploy to Vercel

This project deploys to Vercel as a static site with API rewrites — no server needed.

1. Push to GitHub
2. Import the repo on [vercel.com](https://vercel.com)
3. Click **Deploy** — no configuration required

Vercel uses `vercel.json` to proxy API requests:

```
/v2/*     → https://v.kiringo.cn/v2/*     (stats, matches, rankings, events)
/api/v1/* → https://vlr.kiringo.cn/*       (VLR player, team, event data)
```

Every `git push` to `main` triggers an automatic redeploy.

## Architecture

```
├── index.html      # Shell: header, nav, search bar, #app container, modal
├── style.css       # Airbnb-inspired design system with CSS variables
├── app.js          # All application logic: routing, API, rendering, caching
├── server.py       # Local dev server with CORS proxy (not needed for Vercel)
├── vercel.json     # Vercel deployment config (API rewrites)
├── v.json          # OpenAPI spec — main stats/matches API
└── vlr.json        # OpenAPI spec — VLR player/team/event API
```

### Routing

Hash-based SPA. The router in `app.js` reads `location.hash` and dispatches to `render*()` functions that write directly to `#app`.

```
#/                  → News
#/matches           → Matches
#/rankings          → Rankings
#/stats             → Player Stats
#/players           → Players
#/teams             → Teams
#/events            → Events
#/team/:id          → Team Detail
#/player/:id        → Player Detail
#/event/:id         → Event Detail
#/search?q=...      → Search Results
```

### APIs

Two backends, both proxied through the same origin:

| Path | Backend | Data |
|------|---------|------|
| `/v2/*` | `v.kiringo.cn` | Stats, matches, rankings, events, search |
| `/api/v1/*` | `vlr.kiringo.cn` | VLR player profiles, team data, event details |

### Caching

`CacheManager` in `app.js` uses localStorage with per-key TTL:

| Key | TTL | Notes |
|-----|-----|-------|
| `/news` | 5 min | |
| `/match` | 30 sec | Short for live scores |
| `/match/details` | 5 min | |
| `/rankings` | 5 min | |
| `/stats` | 5 min | |
| `/events` | 5 min | |
| `/team` | 5 min | |
| `/player` | 5 min | |
| `/search` | 2 min | |
| VLR API paths | 5 min | Separate cache namespace |

Stale cache is returned immediately while refreshing in the background (stale-while-revalidate).

## Design System

Airbnb-inspired design with warm Rausch accent. See [DESIGN.md](DESIGN.md) for the full token spec.

### Key Tokens

```css
/* Colors */
--primary: #ff385c       /* Rausch pink */
--ink: #222              /* Headlines */
--body: #3f3f3f          /* Body text */
--muted: #6a6a6a         /* Secondary text */
--canvas: #fff           /* Page background */
--surface-soft: #f7f7f7  /* Card/header background */

/* Border Radius */
--r-xs: 4px;  --r-sm: 8px;  --r-md: 14px
--r-lg: 20px; --r-xl: 32px; --r-full: 9999px

/* Spacing */
--s-xxs: 2px; --s-xs: 4px;  --s-sm: 8px
--s-md: 12px; --s-base: 16px; --s-lg: 24px
--s-xl: 32px; --s-xxl: 48px; --s-section: 64px

/* Typography */
--font: 'Inter', -apple-system, system-ui, 'Helvetica Neue', sans-serif
```

### Dark Mode

Automatic via `@media(prefers-color-scheme: dark)`. Overrides all color tokens. VLR API calls append `?theme=light|dark` for theme-specific logos.

## Key Patterns

- **`$()` / `$$()`** — querySelector / querySelectorAll helpers
- **`esc()`** — XSS-safe HTML output
- **`fixImg()`** — protocol-relative URL handler (`//` → `https://`)
- **`flagToEmoji()`** — country code to emoji flag converter
- **`vlrThemePath()`** — appends theme parameter to VLR API paths
- **`IntersectionObserver`** — lazy-loads player avatars on stats page
- **`safeFetch()`** — null-safe fetch wrapper

## Development

```bash
# Start dev server
python3 server.py

# Kill existing server
lsof -ti:8080 | xargs kill -9
```

No build step. Edit `index.html`, `style.css`, or `app.js` directly and refresh.

## Tech Stack

- **HTML5** — semantic markup
- **CSS3** — custom properties, grid, flexbox, `@media(prefers-color-scheme)`
- **Vanilla JavaScript** — ES2020+, async/await, IntersectionObserver
- **Python 3** — local dev server with CORS proxy
- **Vercel** — static hosting with API rewrites

## License

MIT
