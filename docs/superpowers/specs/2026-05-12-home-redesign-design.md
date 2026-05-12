# Home Screen Redesign — Design Spec

**Date:** 2026-05-12
**Status:** Approved

## Overview

Restructure the War Room home screen: remove the Habits feature entirely, replace the flat 6-card summary grid with a two-column layout, and add a mini calendar panel with month navigation and event dot indicators.

## What's Changing

| Area | Change |
|---|---|
| Habits tab | Removed from tab bar and entirely from the codebase |
| Habits summary card | Removed from the home grid |
| Calendar summary card | Removed — replaced by the mini calendar panel |
| Home layout | Restructured to a two-column main area |
| Summary grid | 2×2 grid: Tasks, Watchlist, Notes, Links |
| Mini calendar | New panel below the summary grid (left column) |
| News feed | Moves to right column, gains full height |
| Markets strip | Unchanged — stays full-width at the bottom |
| Hero section | Unchanged — clock, date, greeting, theme swatches |

## Layout

```
┌─────────────────────────────────────────────────────┐
│  Clock / Date / Greeting           Theme Swatches    │  ← hero (unchanged)
├──────────────────────────┬──────────────────────────┤
│  SUMMARY                 │  NEWS                    │
│  ┌──────────┬──────────┐ │  article headline        │
│  │ Tasks    │ Watchlist│ │  article sub              │
│  ├──────────┼──────────┤ │  article headline        │
│  │ Notes    │ Links    │ │  article sub              │
│  └──────────┴──────────┘ │  article headline        │
│                          │  article sub              │
│  CALENDAR                │  article headline        │
│  ◀  May 2026  ▶          │  article sub              │
│  M  T  W  T  F  S  S    │  article headline        │
│  …  …  … 12  …  …  …    │  ...                     │
│  …  • …  …  …  •  …    │                          │
│  …  …  …  …  …  …  …    │                          │
├──────────────────────────┴──────────────────────────┤
│  S&P ▲  NASDAQ ▲  FTSE ▼  BTC ▲  ETH ▲  Gold ▲ … │  ← markets strip
└─────────────────────────────────────────────────────┘
```

Left column is ~60% width, right column ~40%.

## Mini Calendar

### Display
- Full month grid (7 columns Mon–Sun, variable rows)
- Day-of-week header row (M T W T F S S)
- Today's date is highlighted (accent background)
- Days with events show a small purple dot below the number

### Navigation
- ◀ ▶ buttons change the displayed month
- Month/year label updates accordingly (e.g. "May 2026")
- Navigation state is local JS variable — not synced to Supabase
- On home render, calendar resets to current month

### Event Dots
- Reads from `Store.get('wr_calevents', [])` — same data source as the Calendar tab
- Each event has a `date` field (ISO string `YYYY-MM-DD`)
- A dot is shown on any day that has one or more events in the current displayed month

### Interaction
- Clicking any day cell calls `navigate('calendar')` to open the full Calendar tab
- No tooltip or inline event display — click takes you straight to the Calendar tab

## Habits Removal

- Remove `<div class="tab" data-tab="habits">` from the tab bar
- Remove `<div class="section" id="section-habits">` and all its contents
- Remove the Habits card from `renderHome()` summary grid
- Remove `renderHabits()`, `addHabit()`, `toggleHabit()`, `deleteHabit()` functions and any habits-specific helpers
- Remove `wr_habits` and `wr_completions` Store references
- The `wr_habits` and `wr_completions` keys remain in Supabase/localStorage harmlessly (no cleanup needed)

## JS Architecture

### New functions
- `renderMiniCalendar()` — builds and injects the mini calendar HTML into `#home-mini-cal`
- `miniCalPrev()` — decrements displayed month, calls `renderMiniCalendar()`
- `miniCalNext()` — increments displayed month, calls `renderMiniCalendar()`

### State
```js
let miniCalYear  = new Date().getFullYear();
let miniCalMonth = new Date().getMonth(); // 0-indexed
```

### Updated functions
- `renderHome()` — updated to render new two-column layout and call `renderMiniCalendar()`
- `navigate(tab)` — no change needed; Habits section is removed entirely so no guard required

### Key function (DO NOT REDEFINE)
`renderMiniCalendar`, `miniCalPrev`, `miniCalNext` — added to the project_warroom.md key functions list after implementation.

## CSS

### New selectors needed
- `.home-main` — two-column flex/grid layout replacing the current single-column layout
- `.home-left-col` — left column, ~60% width
- `.home-right-col` — right column, ~40% width, contains news
- `#home-mini-cal` — mini calendar container
- `.mini-cal-header` — nav row with ◀ month/year ▶
- `.mini-cal-nav-btn` — prev/next arrow buttons
- `.mini-cal-grid` — 7-column CSS grid for day cells
- `.mini-cal-day` — individual day cell
- `.mini-cal-day.today` — today highlight
- `.mini-cal-day.has-event::after` — dot indicator (pseudo-element)
- `.mini-cal-dow` — day-of-week header cells

### Existing selectors to update
- `#home-grid` — changes from 3×2 to 2×2 (`grid-template-columns: 1fr 1fr`)
- `#home-news` / `.news-section` — moves into right column; remove any fixed width

## Implementation Branch

`war-room-home-redesign` worktree — merged to master when complete.
