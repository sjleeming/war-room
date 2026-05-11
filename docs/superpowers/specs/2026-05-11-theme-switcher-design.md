# Theme Switcher — Design Spec

**Date:** 2026-05-11
**Status:** Approved

## Overview

Replace the existing binary light/dark toggle with two named themes — Midnight and Arctic — selectable via colour swatches on the Home screen. Choice persists to Supabase.

## Themes

### Midnight (dark)
The current dark theme, renamed and locked in. Deep black/zinc background, purple accent.
- `--bg: #18181b`
- `--surface: #27272a`
- `--surface2: #3f3f46`
- `--border: #3f3f46`
- `--border2: #52525b`
- `--text: #fafafa`
- `--text2: #a1a1aa`
- `--text3: #71717a`
- `--accent: #a855f7`
- `--accent-bg: #2e1065`

### Arctic (light)
New clean light theme. White/slate background, blue accent.
- `--bg: #f8fafc`
- `--surface: #f1f5f9`
- `--surface2: #e2e8f0`
- `--border: #e2e8f0`
- `--border2: #cbd5e1`
- `--text: #0f172a`
- `--text2: #475569`
- `--text3: #94a3b8`
- `--accent: #3b82f6`
- `--accent-bg: #eff6ff`

## UI — Theme Swatches on Home

Below the clock/greeting on the Home screen, a small row of two circular swatches is shown:

```
THEME  ⬤ ⬤
       ↑   ↑
   Midnight Arctic
```

- Each swatch is a circle filled with that theme's `--bg` colour and a border in its `--accent` colour.
- The active theme's swatch has a visible ring/highlight (e.g. 2px offset outline in the accent colour).
- Clicking a swatch switches the theme instantly.
- The swatch row sits below the greeting line and above the summary grid.

## How It Works Technically

- The site already uses `html[data-theme]` to drive CSS custom property switching. Midnight maps to `data-theme="dark"`, Arctic maps to `data-theme="light"`.
- The existing theme toggle button (🌙/☀️) is removed and replaced by the Home screen swatches.
- `Store.set('wr_theme', themeName)` persists the choice — this syncs to Supabase automatically via the existing Store implementation.
- On load, `Store.get('wr_theme')` is read and applied before first render to avoid a flash.
- The existing `html[data-theme="light"]` and `html[data-theme="dark"]` CSS blocks are updated to match the Midnight/Arctic colour values above.

## Architecture

- CSS: update existing `html[data-theme="light"]` and `html[data-theme="dark"]` blocks with new values.
- Remove: existing theme toggle button HTML and its JS handler.
- Add: `.theme-swatches` HTML block inside `#home-main`, below the greeting/clock.
- Add: `setTheme(name)` JS function that sets `data-theme`, updates swatch active state, and calls `Store.set`.
- Add: CSS for `.theme-swatches`, `.theme-swatch`, `.theme-swatch.active`.
- On init: read saved theme from `Store.get('wr_theme', 'dark')` and call `setTheme`.

## Implementation Branch

`war-room-themes` worktree — merged to master when complete.
