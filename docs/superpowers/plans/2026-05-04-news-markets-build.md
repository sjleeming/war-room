# News & Markets Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a live Markets ticker strip and a News headline grid to the War Room Home tab, fetching real data from free public APIs with no key required.

**Architecture:** All changes are confined to `warroom.html`. CSS goes inside the existing `<style>` block. HTML is appended inside `#section-home`. JS is appended inside the existing `<script>` block before the `/* INIT */` comment. No new files, no new localStorage keys, no new tabs.

**Tech Stack:** Vanilla JS (ES6+), Yahoo Finance v7 quote API via allorigins.win CORS proxy, CoinGecko public API, rss2json.com free RSS-to-JSON endpoint.

**Spec:** `docs/superpowers/specs/2026-05-04-news-markets-design.md`

---

### Task 1: CSS — Markets strip + News grid styles

**Files:**
- Modify: `warroom.html` — add CSS inside `<style>` block before `</style>` (line 590)

- [ ] **Step 1: Add the following CSS block inside `<style>`, immediately before `</style>`**

```css
    /* ── Live Section Shared ── */
    .live-section {
      margin-top: 28px;
    }
    .live-section-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 12px;
    }
    .live-section-label {
      font-family: 'Syne', sans-serif;
      font-size: 11px;
      font-weight: 700;
      letter-spacing: 1.5px;
      text-transform: uppercase;
    }
    .markets-label { color: var(--green); }
    .news-label    { color: #60a5fa; }
    .live-section-meta {
      display: flex;
      align-items: center;
      gap: 8px;
    }
    .live-timestamp {
      font-size: 11px;
      color: var(--text3);
    }
    .refresh-btn {
      background: none;
      border: none;
      color: var(--text3);
      cursor: pointer;
      font-size: 15px;
      padding: 2px 5px;
      border-radius: 4px;
      transition: color 0.15s;
      line-height: 1;
    }
    .refresh-btn:hover { color: var(--text2); }

    /* ── Ticker strip ── */
    #ticker-strip {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
    }
    .ticker-card {
      background: var(--surface);
      border-radius: 8px;
      padding: 10px 12px;
      min-width: 88px;
      flex: 1;
      border-left: 3px solid var(--border2);
    }
    .ticker-card.up   { border-left-color: var(--green); }
    .ticker-card.down { border-left-color: var(--danger); }
    .ticker-name {
      font-size: 11px;
      color: var(--text3);
      margin-bottom: 4px;
      white-space: nowrap;
    }
    .ticker-change {
      font-size: 14px;
      font-weight: 600;
      color: var(--text2);
    }
    .ticker-card.up   .ticker-change { color: var(--green); }
    .ticker-card.down .ticker-change { color: var(--danger); }
    .ticker-price {
      font-size: 11px;
      color: var(--text3);
      margin-top: 2px;
    }

    /* ── News grid ── */
    #news-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 8px;
    }
    a.news-card {
      background: var(--surface);
      border-radius: 8px;
      padding: 12px 14px;
      border-left: 3px solid #60a5fa;
      text-decoration: none;
      display: block;
      transition: background 0.15s;
    }
    a.news-card:hover { background: var(--surface2); }
    .news-headline {
      font-size: 13px;
      color: var(--text);
      margin-bottom: 5px;
      line-height: 1.4;
      display: -webkit-box;
      -webkit-line-clamp: 2;
      -webkit-box-orient: vertical;
      overflow: hidden;
    }
    .news-meta {
      font-size: 11px;
      color: var(--text3);
    }
    .news-error {
      color: var(--text3);
      font-size: 13px;
      grid-column: 1 / -1;
      text-align: center;
      padding: 20px 0;
    }

    /* ── Skeleton shimmer ── */
    .skeleton { pointer-events: none; }
    .skel-line {
      background: linear-gradient(90deg, var(--surface2) 25%, var(--border2) 50%, var(--surface2) 75%);
      background-size: 200% 100%;
      animation: shimmer 1.4s infinite;
      border-radius: 4px;
      height: 11px;
      margin-bottom: 6px;
    }
    .skel-line.short { width: 55%; }
    @keyframes shimmer {
      0%   { background-position: 200% 0; }
      100% { background-position: -200% 0; }
    }

    /* ── Mobile extras for live sections ── */
    @media (max-width: 600px) {
      #news-grid    { grid-template-columns: 1fr; }
      #ticker-strip { gap: 5px; }
      .ticker-card  { min-width: 70px; padding: 8px 9px; }
    }
```

- [ ] **Step 2: Open `warroom.html` in a browser and verify**

Reload. The page should look identical to before — no new visible elements yet (the HTML shells don't exist yet), no console errors.

- [ ] **Step 3: Commit**

```bash
cd "C:\Users\samjl\OneDrive\Documents\War Room Planning"
git add warroom.html
git commit -m "feat: add CSS for markets ticker strip and news grid"
```

---

### Task 2: HTML — Add Markets and News shells to Home section

**Files:**
- Modify: `warroom.html` — update `#section-home` HTML (around line 612)

- [ ] **Step 1: Find this block in `#section-home` (lines 612–614)**

```html
    <div id="home-grid">
      <!-- Cards injected by renderHome() -->
    </div>
  </div>
```

Replace it with:

```html
    <div id="home-grid">
      <!-- Cards injected by renderHome() -->
    </div>

    <!-- Markets -->
    <div class="live-section" id="home-markets">
      <div class="live-section-header">
        <span class="live-section-label markets-label">Markets</span>
        <div class="live-section-meta">
          <span class="live-timestamp" id="markets-timestamp"></span>
          <button class="refresh-btn" onclick="loadMarkets()" title="Refresh markets">↻</button>
        </div>
      </div>
      <div id="ticker-strip"></div>
    </div>

    <!-- News -->
    <div class="live-section" id="home-news">
      <div class="live-section-header">
        <span class="live-section-label news-label">News</span>
        <div class="live-section-meta">
          <span class="live-timestamp" id="news-timestamp"></span>
          <button class="refresh-btn" onclick="loadNews()" title="Refresh news">↻</button>
        </div>
      </div>
      <div id="news-grid"></div>
    </div>
  </div>
```

- [ ] **Step 2: Verify in browser**

Reload. On the Home tab, below the 6 summary cards you should now see:
- A "Markets" label in green with a ↻ button
- A "News" label in blue with a ↻ button
- Both sections are empty (no data yet — JS not wired)
- No console errors

- [ ] **Step 3: Commit**

```bash
git add warroom.html
git commit -m "feat: add markets and news HTML shells to Home section"
```

---

### Task 3: JS — Utilities, constants, and market data functions

**Files:**
- Modify: `warroom.html` — add JS inside `<script>` block, immediately before the `/* INIT */` comment (line 1407)

- [ ] **Step 1: Add the following JS block inside `<script>`, immediately before the `/* INIT */` comment**

```js
    /* ══════════════════════════════════════
       LIVE DATA — constants
    ══════════════════════════════════════ */

    const TICKERS = [
      { key: '^GSPC',    name: 'S&P 500',  source: 'yahoo' },
      { key: '^IXIC',    name: 'NASDAQ',   source: 'yahoo' },
      { key: '^FTSE',    name: 'FTSE 100', source: 'yahoo' },
      { key: '^GDAXI',   name: 'DAX',      source: 'yahoo' },
      { key: 'bitcoin',  name: 'BTC',      source: 'coingecko' },
      { key: 'ethereum', name: 'ETH',      source: 'coingecko' },
      { key: 'GC=F',     name: 'Gold',     source: 'yahoo' },
      { key: 'SI=F',     name: 'Silver',   source: 'yahoo' },
      { key: 'CL=F',     name: 'Oil WTI',  source: 'yahoo' },
    ];

    const NEWS_FEEDS = [
      { url: 'https://feeds.bbci.co.uk/news/rss.xml', name: 'BBC' },
      { url: 'https://www.theguardian.com/world/rss',  name: 'Guardian' },
    ];

    /* ══════════════════════════════════════
       LIVE DATA — fetch utility
    ══════════════════════════════════════ */

    async function fetchWithTimeout(url, ms = 10000) {
      const controller = new AbortController();
      const timer = setTimeout(() => controller.abort(), ms);
      try {
        const res = await fetch(url, { signal: controller.signal });
        clearTimeout(timer);
        if (!res.ok) throw new Error('HTTP ' + res.status);
        return res;
      } catch (e) {
        clearTimeout(timer);
        throw e;
      }
    }

    /* ══════════════════════════════════════
       MARKETS — fetch
    ══════════════════════════════════════ */

    async function fetchYahooMarkets() {
      const symbols = TICKERS
        .filter(t => t.source === 'yahoo')
        .map(t => t.key)
        .join(',');
      const yahooUrl = 'https://query1.finance.yahoo.com/v7/finance/quote?symbols=' +
        encodeURIComponent(symbols);
      const proxyUrl = 'https://api.allorigins.win/get?url=' + encodeURIComponent(yahooUrl);
      const res  = await fetchWithTimeout(proxyUrl);
      const data = await res.json();
      const parsed = JSON.parse(data.contents);
      const out = {};
      (parsed.quoteResponse.result || []).forEach(q => {
        out[q.symbol] = {
          price:  q.regularMarketPrice,
          change: q.regularMarketChangePercent,
        };
      });
      return out;
    }

    async function fetchCryptoMarkets() {
      const ids = TICKERS
        .filter(t => t.source === 'coingecko')
        .map(t => t.key)
        .join(',');
      const url = 'https://api.coingecko.com/api/v3/simple/price?ids=' + ids +
        '&vs_currencies=usd&include_24hr_change=true';
      const res  = await fetchWithTimeout(url);
      const data = await res.json();
      const out  = {};
      TICKERS.filter(t => t.source === 'coingecko').forEach(t => {
        if (data[t.key]) {
          out[t.key] = {
            price:  data[t.key].usd,
            change: data[t.key].usd_24h_change,
          };
        }
      });
      return out;
    }

    async function fetchMarkets() {
      const [yahooResult, cryptoResult] = await Promise.allSettled([
        fetchYahooMarkets(),
        fetchCryptoMarkets(),
      ]);
      return {
        ...(yahooResult.status  === 'fulfilled' ? yahooResult.value  : {}),
        ...(cryptoResult.status === 'fulfilled' ? cryptoResult.value : {}),
      };
    }

    /* ══════════════════════════════════════
       MARKETS — render
    ══════════════════════════════════════ */

    function formatTickerPrice(ticker, price) {
      if (price == null) return '';
      if (ticker.source === 'coingecko') {
        return '$' + (price > 1000
          ? price.toLocaleString('en-US', { maximumFractionDigits: 0 })
          : price.toFixed(2));
      }
      if (['GC=F', 'SI=F', 'CL=F'].includes(ticker.key)) return '$' + price.toFixed(2);
      return price.toLocaleString('en-US', { maximumFractionDigits: 2 });
    }

    function showMarketsSkeleton() {
      document.getElementById('ticker-strip').innerHTML =
        Array(9).fill(
          '<div class="ticker-card skeleton">' +
          '<div class="skel-line"></div>' +
          '<div class="skel-line short"></div>' +
          '</div>'
        ).join('');
    }

    function renderMarkets(data) {
      const strip = document.getElementById('ticker-strip');
      strip.innerHTML = TICKERS.map(t => {
        const d = data[t.key];
        if (!d) {
          return '<div class="ticker-card">' +
            '<div class="ticker-name">' + t.name + '</div>' +
            '<div class="ticker-change">—</div>' +
            '</div>';
        }
        const up = d.change >= 0;
        return '<div class="ticker-card ' + (up ? 'up' : 'down') + '">' +
          '<div class="ticker-name">' + t.name + '</div>' +
          '<div class="ticker-change">' + (up ? '+' : '') + d.change.toFixed(2) + '%</div>' +
          '<div class="ticker-price">' + formatTickerPrice(t, d.price) + '</div>' +
          '</div>';
      }).join('');
      const ts = document.getElementById('markets-timestamp');
      if (ts) ts.textContent = 'Updated ' +
        new Date().toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' });
    }

    let marketsLoading = false;

    async function loadMarkets() {
      if (marketsLoading) return;
      marketsLoading = true;
      showMarketsSkeleton();
      try {
        const data = await fetchMarkets();
        renderMarkets(data);
      } catch (e) {
        renderMarkets({});
      }
      marketsLoading = false;
    }
```

- [ ] **Step 2: Verify in browser console**

Reload. Open DevTools console and run:
```js
loadMarkets()
```
Expected: ticker strip shows skeleton shimmer briefly, then populates with 9 ticker cards. Green/red colouring reflects real market moves. Timestamp updates to current time. No uncaught errors.

If CoinGecko or Yahoo returns an error you'll see `—` on those cards — that is correct graceful-fallback behaviour.

- [ ] **Step 3: Commit**

```bash
git add warroom.html
git commit -m "feat: markets data fetch and render — Yahoo Finance + CoinGecko, skeleton loading state"
```

---

### Task 4: JS — News fetch and render functions

**Files:**
- Modify: `warroom.html` — add JS inside `<script>` block, immediately before the `/* INIT */` comment

- [ ] **Step 1: Add the following JS block immediately before the `/* INIT */` comment (after the markets block from Task 3)**

```js
    /* ══════════════════════════════════════
       NEWS — fetch
    ══════════════════════════════════════ */

    async function fetchNewsFeed(feed) {
      const url = 'https://api.rss2json.com/v1/api.json?rss_url=' +
        encodeURIComponent(feed.url);
      const res  = await fetchWithTimeout(url);
      const data = await res.json();
      if (data.status !== 'ok') throw new Error('Feed error: ' + feed.name);
      return data.items.map(item => ({
        title:   item.title,
        link:    item.link,
        pubDate: new Date(item.pubDate),
        source:  feed.name,
      }));
    }

    async function fetchNews() {
      const results  = await Promise.allSettled(NEWS_FEEDS.map(f => fetchNewsFeed(f)));
      const articles = [];
      results.forEach(r => { if (r.status === 'fulfilled') articles.push(...r.value); });
      return articles
        .sort((a, b) => b.pubDate - a.pubDate)
        .slice(0, 6);
    }

    /* ══════════════════════════════════════
       NEWS — render
    ══════════════════════════════════════ */

    function relativeTime(date) {
      const diff = Math.floor((Date.now() - date.getTime()) / 1000);
      if (diff < 60)    return 'just now';
      if (diff < 3600)  return Math.floor(diff / 60)   + 'm ago';
      if (diff < 86400) return Math.floor(diff / 3600)  + 'h ago';
      return Math.floor(diff / 86400) + 'd ago';
    }

    function showNewsSkeleton() {
      document.getElementById('news-grid').innerHTML =
        Array(6).fill(
          '<div class="news-card skeleton">' +
          '<div class="skel-line"></div>' +
          '<div class="skel-line short"></div>' +
          '</div>'
        ).join('');
    }

    function renderNews(articles) {
      const grid = document.getElementById('news-grid');
      const ts   = document.getElementById('news-timestamp');
      if (!articles.length) {
        grid.innerHTML = '<p class="news-error">Couldn\'t load news — check connection</p>';
        if (ts) ts.textContent = '';
        return;
      }
      grid.innerHTML = articles.map(a =>
        '<a class="news-card" href="' + escHtml(a.link) + '" target="_blank" rel="noopener">' +
        '<div class="news-headline">' + escHtml(a.title) + '</div>' +
        '<div class="news-meta">' + escHtml(a.source) + ' · ' + relativeTime(a.pubDate) + '</div>' +
        '</a>'
      ).join('');
      if (ts) ts.textContent = 'Updated ' +
        new Date().toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' });
    }

    let newsLoading = false;

    async function loadNews() {
      if (newsLoading) return;
      newsLoading = true;
      showNewsSkeleton();
      try {
        const articles = await fetchNews();
        renderNews(articles);
      } catch (e) {
        renderNews([]);
      }
      newsLoading = false;
    }
```

- [ ] **Step 2: Verify in browser console**

Reload. Open DevTools console and run:
```js
loadNews()
```
Expected: news grid shows skeleton shimmer briefly, then populates with up to 6 news cards in 2 columns. Each card shows a headline, source name, and relative timestamp. Cards are clickable links. No uncaught errors.

If rss2json rate-limits you (unlikely in testing), you'll see the "Couldn't load news" message — correct fallback.

- [ ] **Step 3: Commit**

```bash
git add warroom.html
git commit -m "feat: news fetch and render — BBC + Guardian via rss2json, skeleton loading state"
```

---

### Task 5: Wire up INIT — auto-load and refresh intervals

**Files:**
- Modify: `warroom.html` — update the `/* INIT */` block (lines 1407–1414)

- [ ] **Step 1: Find the INIT block**

```js
    runMorningCheckin();
    navigate('home');
    renderHome();
    setInterval(() => { if (currentTab === 'home') updateClock(); }, 1000);
    setInterval(() => { if (currentTab === 'home') renderHome(); }, 60000);
```

Replace it with:

```js
    runMorningCheckin();
    navigate('home');
    renderHome();
    loadMarkets();
    loadNews();
    setInterval(() => { if (currentTab === 'home') updateClock(); }, 1000);
    setInterval(() => { if (currentTab === 'home') renderHome(); }, 60000);
    setInterval(() => { if (currentTab === 'home') loadMarkets(); }, 5 * 60 * 1000);
    setInterval(() => { if (currentTab === 'home') loadNews(); }, 15 * 60 * 1000);
```

- [ ] **Step 2: Verify full end-to-end in browser**

Reload `warroom.html`. On the Home tab verify:
- Skeleton shimmers appear immediately in both Markets and News sections
- Within ~5 seconds, ticker cards populate with real % changes and prices
- Within ~5 seconds, 6 news headlines appear in 2 columns
- "Updated HH:MM" timestamps appear next to both section labels
- Clicking a news card opens the article in a new tab
- Clicking ↻ next to Markets triggers a fresh fetch (skeletons flash, then new data)
- Clicking ↻ next to News triggers a fresh fetch
- All other tabs (Tasks, Watchlist, etc.) still work correctly
- No console errors

- [ ] **Step 3: Test graceful fallback**

In DevTools Network tab, set throttling to "Offline". Click ↻ on Markets and ↻ on News.
Expected: skeleton briefly appears, then Markets shows all `—` cards, News shows "Couldn't load news" message. Re-enable network, click ↻ again — data comes back.

- [ ] **Step 4: Commit**

```bash
git add warroom.html
git commit -m "feat: wire markets and news auto-load and 5min/15min refresh intervals"
```

---

## Self-Review

**Spec coverage check:**

| Spec requirement | Task |
|---|---|
| Both sections on Home tab, below summary cards | Task 2 |
| Horizontal ticker strip layout | Tasks 1, 3 |
| 9 tickers: S&P, NASDAQ, FTSE, DAX, BTC, ETH, Gold, Silver, Oil | Task 3 (TICKERS constant) |
| Green/red colouring per direction | Task 1 (CSS), Task 3 (renderMarkets) |
| Price shown in muted text | Task 1 (CSS `.ticker-price`), Task 3 (formatTickerPrice) |
| Section label + ↻ button + timestamp | Tasks 1, 2 |
| Yahoo Finance via allorigins proxy | Task 3 (fetchYahooMarkets) |
| CoinGecko for BTC/ETH | Task 3 (fetchCryptoMarkets) |
| Both market sources fire in parallel | Task 3 (Promise.allSettled in fetchMarkets) |
| 2-column news grid | Task 1 (CSS `#news-grid`) |
| Up to 6 articles, sorted newest-first | Task 4 (fetchNews) |
| BBC + Guardian feeds via rss2json | Task 4 (NEWS_FEEDS constant) |
| Source name + relative time on each card | Task 4 (renderNews, relativeTime) |
| Cards clickable — open in new tab | Task 4 (renderNews — `<a target="_blank">`) |
| Markets refresh every 5 min | Task 5 (setInterval 5 * 60 * 1000) |
| News refresh every 15 min | Task 5 (setInterval 15 * 60 * 1000) |
| Only refresh when Home tab is active | Task 5 (currentTab === 'home' guard) |
| Manual ↻ buttons | Task 2 (HTML onclick), Task 3/4 (loadMarkets/loadNews) |
| Skeleton shimmer on load | Tasks 1 (CSS), 3 (showMarketsSkeleton), 4 (showNewsSkeleton) |
| 10s fetch timeout | Task 3 (fetchWithTimeout) |
| Per-section graceful fallback on failure | Tasks 3, 4 (try/catch → renderMarkets({})/renderNews([])) |
| No new localStorage keys | ✓ (no Store calls in any task) |
| No new tabs | ✓ (no changes to tab bar or navigate function) |
| Mobile responsive | Task 1 (media query for news-grid and ticker-card) |

**All spec requirements covered. No placeholders. No TBDs.**
