# Markets Ticker — Design Spec

**Date:** 2026-05-11
**Status:** Approved

## Overview

Add live market price data to the War Room in two places: a compact strip on the Home screen and a dedicated Markets tab with a cards grid layout.

## Features

### Home Strip

A horizontal bar below the Home summary grid (`#home-grid`) showing all 9 tickers inline. Each ticker displays: short name, current price, and % change (green if positive, red if negative). The strip is always visible when the user lands on Home. CSS for `#home-markets` already exists at line ~685 of `warroom.html` — do not rewrite it, complete it.

### Markets Tab

A new tab added to the tab bar after Links. Full-page layout with cards grouped into three categories:

**Indices (4 cards, 2×2 grid)**
- S&P 500 (`^GSPC`)
- NASDAQ (`^IXIC`)
- FTSE 100 (`^FTSE`)
- DAX (`^GDAXI`)

**Crypto (2 cards, side by side)**
- Bitcoin (`BTC`)
- Ethereum (`ETH`)

**Commodities (3 cards)**
- Gold (`GC=F`)
- Silver (`SI=F`)
- Oil WTI (`CL=F`)

Each card shows: ticker name, current price (formatted with commas and correct currency/unit), absolute daily change, % daily change. Change values are coloured green (positive) or red (negative) with ▲/▼ arrows.

## Data Sources

- **Indices + Commodities:** Yahoo Finance v7 API via `https://api.allorigins.win/get?url=` proxy. Batch request for all non-crypto tickers in one call.
- **Crypto:** CoinGecko free API at `https://api.coingecko.com/api/v3/simple/price` — no API key required. Fetches BTC + ETH with 24h change.
- Both fetched in parallel via `Promise.allSettled` — one source failing does not break the other.
- Timeout: 8 seconds per request. On failure, show last known values with a stale indicator.

## Refresh

- Auto-refreshes every 5 minutes while the Markets tab is active or Home is visible.
- Uses a `currentTab` guard to skip refresh when neither tab is active.
- Shows a skeleton loading state on first load.

## Architecture

- All JS added before the `/* INIT */` comment in `warroom.html`.
- Functions: `TICKERS` const, `fetchWithTimeout`, `fetchYahooMarkets`, `fetchCryptoMarkets`, `fetchMarkets` (Promise.allSettled), `formatTickerPrice`, `showMarketsSkeleton`, `renderMarkets`, `loadMarkets`.
- Home strip HTML: `#home-markets` div inside `#home-main`, after `#home-grid` closing tag.
- Markets tab HTML: `#section-markets` div, with three `.market-group` sections.
- `loadMarkets()` called on init and on `navigate('markets')` and `navigate('home')`.
- `setInterval` every 5 minutes with currentTab guard.
- No data persisted to Supabase — always fetched live.

## Implementation Branch

`war-room-markets` worktree — merged to master when complete.
