# War Room — News & Markets Design Spec
_Date: 2026-05-04_

---

## Overview

Add two live data sections to the **Home tab** of `warroom.html`, sitting below the existing 6 summary cards:

1. **Markets** — a horizontal strip of 9 market ticker cards showing real-time price change data
2. **News** — a 2-column grid of recent headlines from BBC and The Guardian

No new tabs are added. No localStorage keys are used. Both sections are read-only live data fetched from free public APIs with no API key required.

---

## Placement & Layout

Both sections are appended to `#section-home`, below `#home-grid` (the 6 summary cards).

```
#section-home
  ├── #home-header        (clock, date, greeting — unchanged)
  ├── #home-grid          (6 summary cards — unchanged)
  ├── #home-markets       (NEW — horizontal ticker strip)
  └── #home-news          (NEW — 2-column news grid)
```

### Markets strip

- Section label **"MARKETS"** in green (`#22c55e`), uppercase, Syne font
- Refresh button (↻) and last-updated timestamp (`color: var(--text3)`) aligned right on the same row
- 9 ticker cards in a flex-wrap row, each showing:
  - Ticker name (e.g. "S&P 500")
  - % change — green (`#22c55e`) if positive, red (`#ef4444`) if negative
  - Current price in muted text (`var(--text3)`)
  - Left border coloured to match direction (green/red)
- Skeleton shimmer shown while fetching
- Wraps to 2 rows on mobile (`max-width: 600px`)

**Tickers (in order):** S&P 500 · NASDAQ · FTSE 100 · DAX · BTC · ETH · Gold · Silver · Oil WTI

### News grid

- Section label **"NEWS"** in blue (`#60a5fa`), uppercase, Syne font
- Refresh button (↻) and last-updated timestamp aligned right
- 2-column grid of up to 6 news cards, sorted newest-first
- Each card shows:
  - Headline text
  - Source name + relative time (e.g. "BBC · 14m ago")
  - Blue left border (`#60a5fa`)
  - Entire card is clickable — opens article in new tab
- Collapses to single column on mobile

---

## Data Sources & APIs

All fetches are wrapped in try/catch with a 10-second `AbortController` timeout.

### Markets

| Ticker | Symbol | API |
|--------|--------|-----|
| S&P 500 | `^GSPC` | Yahoo Finance via allorigins proxy |
| NASDAQ | `^IXIC` | Yahoo Finance via allorigins proxy |
| FTSE 100 | `^FTSE` | Yahoo Finance via allorigins proxy |
| DAX | `^GDAXI` | Yahoo Finance via allorigins proxy |
| Gold | `GC=F` | Yahoo Finance via allorigins proxy |
| Silver | `SI=F` | Yahoo Finance via allorigins proxy |
| Oil WTI | `CL=F` | Yahoo Finance via allorigins proxy |
| BTC | `bitcoin` | CoinGecko API (direct, no proxy) |
| ETH | `ethereum` | CoinGecko API (direct, no proxy) |

**Yahoo Finance endpoint (via allorigins):**
```
https://query1.finance.yahoo.com/v8/finance/chart/{symbol}?interval=1d&range=1d
```
Wrapped: `https://api.allorigins.win/get?url=<encoded Yahoo URL>`

**CoinGecko endpoint:**
```
https://api.coingecko.com/api/v3/simple/price?ids=bitcoin,ethereum&vs_currencies=usd&include_24hr_change=true
```

Both market calls fire in parallel (`Promise.allSettled`) on load and on each refresh. If one source fails, its tickers show `—` while the other source's tickers render normally.

### News

**rss2json.com free endpoint:**
```
https://api.rss2json.com/v1/api.json?rss_url=<encoded feed URL>
```

**Feeds:**
- BBC News top stories: `https://feeds.bbci.co.uk/news/rss.xml`
- The Guardian world: `https://www.theguardian.com/world/rss`

Both feeds fetched in parallel. Results merged and sorted by `pubDate` descending. Top 6 shown. Source name extracted from feed `title` field.

---

## Refresh Strategy

| Section | Auto-refresh interval | Manual |
|---------|-----------------------|--------|
| Markets | Every 5 minutes | ↻ button |
| News | Every 15 minutes | ↻ button |

News is capped at 15 minutes to stay within rss2json's free tier limit of 10 requests/hour (2 feeds × 4/hr = 8 req/hr at 15-min interval).

Auto-refresh only fires when `currentTab === 'home'` to avoid background fetches when the user is on another tab.

---

## Error Handling

| Failure | Behaviour |
|---------|-----------|
| Yahoo Finance fetch fails | Affected ticker cards show `—` with muted "unavailable" label |
| CoinGecko fetch fails | BTC/ETH cards show `—` |
| rss2json fetch fails (one feed) | Remaining feed's articles still shown |
| rss2json fetch fails (both feeds) | News section shows `"Couldn't load news"` message |
| Network timeout (10s) | Same as fetch failure — graceful fallback per section |
| Markets outside trading hours | Yahoo Finance returns last known price; timestamp makes staleness clear |

No failure in either section affects the rest of the Home screen or any other tab.

---

## Implementation Notes

- No new localStorage keys
- No new tabs or navigation changes
- All new CSS added inside the existing `<style>` block
- All new JS added inside the existing `<script>` block, after the existing Home section code
- `renderHome()` updated to call `fetchMarkets()` and `fetchNews()` on first render
- Skeleton shimmer uses a CSS animation consistent with the existing app aesthetic
- Colour palette: uses existing CSS variables — `--green`, `--danger`, `--text3`, `--surface`, `--surface2`, `--border`; new `--news-blue: #60a5fa` added

---

## Out of Scope

- Individual stock search or custom ticker configuration
- Historical price charts
- News filtering by category or source
- Push notifications for price alerts
- Saving/bookmarking articles
