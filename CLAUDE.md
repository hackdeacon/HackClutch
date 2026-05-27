# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HackClutch is a Valorant esports dashboard — a vanilla HTML/CSS/JS single-page application with no build tools or frameworks.

## Development Commands

```bash
# Start local dev server (includes CORS proxy)
python3 server.py
# Server runs at http://localhost:8080

# Kill existing server if port in use
lsof -ti:8080 | xargs kill -9
```

## Architecture

**Three-file structure:**
- `index.html` — Shell with header, nav, search bar, `#app` container, modal
- `style.css` — Airbnb-inspired design system using CSS variables (see DESIGN.md)
- `app.js` — All application logic: routing, API calls, rendering, caching

**Routing:** Hash-based SPA (`#/matches`, `#/team/123`). Router in `app.js` dispatches to `render*()` functions that write directly to `#app`.

**APIs (two backends proxied through server.py):**
- `/v2/*` → `https://v.kiringo.cn/v2` (main stats/matches/rankings API)
- `/api/v1/*` → `https://vlr.kiringo.cn` (VLR player/team/event data)

**Caching:** `CacheManager` in `app.js` uses localStorage with per-key TTL (defined in `CacheManager.TTL`). VLR cache is cleared on theme change since VLR API returns theme-specific logos.

**Dark mode:** CSS `@media(prefers-color-scheme:dark)` overrides color variables. VLR API calls append `?theme=light|dark` via `vlrThemePath()`.

**Stats page avatars:** Uses `IntersectionObserver` for lazy-loading — only searches for player data when rows enter viewport. Avatar images preload with `new Image()` before replacing placeholder `<span>` elements.

## Key Patterns

- All DOM manipulation is vanilla JS (`$()` / `$$()` helpers for querySelector)
- API responses use `{status, data}` or `{segments}` structure
- Tables use `.table-wrap` for horizontal scroll
- Sort state stored as arrays (`_statsSort = [{key, dir}]`) for multi-column sort
- `esc()` function for XSS-safe HTML output
- `fixImg()` handles protocol-relative URLs (`//` → `https://`)
- `flagToEmoji()` converts country codes to emoji flags

## Design System

See `DESIGN.md` for full Airbnb-inspired token spec. Key values:
- Primary: `#ff385c` (Rausch pink)
- Border radius: 4/8/14/20/32/9999px scale
- Spacing: 2/4/8/12/16/24/32/48/64px scale
- Font: Inter (fallback for Airbnb Cereal VF)
