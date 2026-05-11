# Markets Ticker Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add live market prices and daily % change to the War Room — a compact strip on the Home screen and a full Markets tab with cards grouped by category.

**Architecture:** All code lives in `warroom.html` (single-file app). CSS for the home ticker strip already exists at lines ~685-773. JS is added before the `/* INIT */` block. The Markets tab is added as a new section alongside the existing 7 tabs.

**Tech Stack:** Vanilla JS, Yahoo Finance v7 via allorigins.win proxy, CoinGecko free API, existing CSS custom properties.

---

### Task 1: Set up worktree

**Files:**
- Working directory: `C:\Users\samjl\OneDrive\Documents\War Room Planning\`

- [ ] **Step 1: Create the worktree branch**

```bash
cd "C:\Users\samjl\OneDrive\Documents\War Room Planning"
git worktree add ../war-room-markets -b war-room-markets
```

- [ ] **Step 2: Verify worktree exists**

```bash
git worktree list
```

Expected output includes: `../war-room-markets  [war-room-markets]`

- [ ] **Step 3: All subsequent steps work in the worktree folder**

```bash
cd ../war-room-markets
```

---

### Task 2: Add Markets tab CSS for tab layout

The home strip CSS already exists (lines ~685-773). This task adds only the additional CSS needed for the full Markets tab layout.

**Files:**
- Modify: `warroom.html` — inside `<style>` block, just before the closing `</style>` tag (currently line ~774)

- [ ] **Step 1: Add markets tab CSS**

Find the line `  </style>` (just before `</head>`) and insert before it:

```css
    /* ── Markets Tab Layout ── */
    .markets-tab-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 24px;
    }
    .market-group { margin-bottom: 28px; }
    .market-cards {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
      gap: 10px;
      margin-top: 10px;
    }
```

- [ ] **Step 2: Commit**

```bash
git add warroom.html
git commit -m "style: add markets tab layout CSS"
```

---

### Task 3: Add Markets tab HTML and home strip HTML

**Files:**
- Modify: `warroom.html` — tab bar and section HTML

- [ ] **Step 1: Add Markets tab pill to the tab bar**

Find this line (currently ~line 788):
```html
      <div class="tab" data-tab="links"><span class="tab-icon">🔗</span><span class="tab-label"> Links</span></div>
```
Add after it:
```html
      <div class="tab" data-tab="markets"><span class="tab-icon">📈</span><span class="tab-label"> Markets</span></div>
```

- [ ] **Step 2: Add `#home-markets` strip to the Home section**

Find this block (currently ~line 808):
```html
      <div id="home-grid">
        <!-- Cards injected by renderHome() -->
      </div>
```
Add after the closing `</div>`:
```html
      <div id="home-markets">
        <div class="markets-section-header">
          <span class="markets-section-label">Markets</span>
          <div class="markets-section-meta">
            <span class="markets-timestamp" id="markets-timestamp"></span>
            <button class="markets-refresh-btn" onclick="loadMarkets()" title="Refresh">↻</button>
          </div>
        </div>
        <div id="ticker-strip"></div>
      </div>
```

- [ ] **Step 3: Add Markets section shell**

Find the last `<div class="section"` block (the Links section). After its closing `</div>`, add:
```html
  <div class="section" id="section-markets">
    <div class="markets-tab-header">
      <h2 class="section-heading">Markets</h2>
      <div class="markets-section-meta">
        <span class="markets-timestamp" id="markets-tab-timestamp"></span>
        <button class="markets-refresh-btn" onclick="loadMarkets()" title="Refresh">↻</button>
      </div>
    </div>
    <div id="markets-groups"></div>
  </div>
```

- [ ] **Step 4: Commit**

```bash
git add warroom.html
git commit -m "feat: add markets tab pill and HTML shells"
```

---

### Task 4: Add TICKERS config and fetch utilities

**Files:**
- Modify: `warroom.html` — JS section, before the `/* INIT */` block

All JS in this and subsequent tasks goes in the same location: just before the `/* ══ INIT ══ */` comment block.

- [ ] **Step 1: Add TICKERS constant and fetch utilities**

Find the `/* ══════════════════════════════════════\n       INIT` comment block. Insert before it:

```js
    /* ══════════════════════════════════════
       MARKETS
    ══════════════════════════════════════ */
    const TICKERS = [
      { id: 'sp500',  symbol: '^GSPC',    name: 'S&P 500',  group: 'indices',     fmt: 'index' },
      { id: 'nasdaq', symbol: '^IXIC',    name: 'NASDAQ',   group: 'indices',     fmt: 'index' },
      { id: 'ftse',   symbol: '^FTSE',    name: 'FTSE 100', group: 'indices',     fmt: 'index' },
      { id: 'dax',    symbol: '^GDAXI',   name: 'DAX',      group: 'indices',     fmt: 'index' },
      { id: 'btc',    symbol: 'bitcoin',  name: 'BTC',      group: 'crypto',      fmt: 'crypto' },
      { id: 'eth',    symbol: 'ethereum', name: 'ETH',      group: 'crypto',      fmt: 'crypto' },
      { id: 'gold',   symbol: 'GC=F',     name: 'Gold',     group: 'commodities', fmt: 'commodity' },
      { id: 'silver', symbol: 'SI=F',     name: 'Silver',   group: 'commodities', fmt: 'commodity' },
      { id: 'oil',    symbol: 'CL=F',     name: 'Oil WTI',  group: 'commodities', fmt: 'commodity' },
    ];

    let marketsData   = {};
    let marketsLoading = false;

    function fetchWithTimeout(url, ms = 8000) {
      const ctrl = new AbortController();
      const t = setTimeout(() => ctrl.abort(), ms);
      return fetch(url, { signal: ctrl.signal }).finally(() => clearTimeout(t));
    }

    function formatTickerPrice(ticker, val) {
      if (val == null) return '—';
      if (ticker.fmt === 'crypto')     return '$' + Math.round(val).toLocaleString('en-US');
      if (ticker.fmt === 'commodity')  return '$' + val.toFixed(2);
      return val.toLocaleString('en-US', { maximumFractionDigits: 2 });
    }
```

- [ ] **Step 2: Commit**

```bash
git add warroom.html
git commit -m "feat: add TICKERS config, marketsData state, fetchWithTimeout, formatTickerPrice"
```

---

### Task 5: Add Yahoo and CoinGecko fetch functions

**Files:**
- Modify: `warroom.html` — JS section, continuing after Task 4 additions

- [ ] **Step 1: Add fetch functions after the Task 4 block**

```js
    async function fetchYahooMarkets() {
      const nonCrypto = TICKERS.filter(t => t.fmt !== 'crypto');
      const symbols   = nonCrypto.map(t => t.symbol).join(',');
      const yUrl      = `https://query1.finance.yahoo.com/v7/finance/quote?symbols=${encodeURIComponent(symbols)}`;
      const proxy     = `https://api.allorigins.win/get?url=${encodeURIComponent(yUrl)}`;
      const res       = await fetchWithTimeout(proxy);
      const outer     = await res.json();
      const data      = JSON.parse(outer.contents);
      return data.quoteResponse.result.map(q => ({
        id:     TICKERS.find(t => t.symbol === q.symbol)?.id,
        price:  q.regularMarketPrice,
        change: q.regularMarketChange,
        pct:    q.regularMarketChangePercent,
      })).filter(t => t.id);
    }

    async function fetchCryptoMarkets() {
      const url = 'https://api.coingecko.com/api/v3/simple/price?ids=bitcoin,ethereum&vs_currencies=usd&include_24hr_change=true';
      const res = await fetchWithTimeout(url);
      const d   = await res.json();
      return [
        { id: 'btc', price: d.bitcoin.usd,  change: null, pct: d.bitcoin.usd_24h_change },
        { id: 'eth', price: d.ethereum.usd, change: null, pct: d.ethereum.usd_24h_change },
      ];
    }

    async function fetchMarkets() {
      const [yahoo, crypto] = await Promise.allSettled([fetchYahooMarkets(), fetchCryptoMarkets()]);
      if (yahoo.status  === 'fulfilled') yahoo.value.forEach(t  => { marketsData[t.id] = t; });
      if (crypto.status === 'fulfilled') crypto.value.forEach(t => { marketsData[t.id] = t; });
    }
```

- [ ] **Step 2: Commit**

```bash
git add warroom.html
git commit -m "feat: add fetchYahooMarkets, fetchCryptoMarkets, fetchMarkets"
```

---

### Task 6: Add render functions

**Files:**
- Modify: `warroom.html` — JS section, continuing after Task 5 additions

- [ ] **Step 1: Add skeleton and render functions**

```js
    function showMarketsSkeleton() {
      const strip = document.getElementById('ticker-strip');
      if (strip) strip.innerHTML = TICKERS.map(() =>
        `<div class="ticker-card"><div class="skel-line"></div><div class="skel-line short"></div></div>`
      ).join('');
    }

    function updateMarketsTimestamp(stale = false) {
      const time = new Date().toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' });
      const txt  = stale ? '⚠ stale' : `Updated ${time}`;
      ['markets-timestamp', 'markets-tab-timestamp'].forEach(id => {
        const el = document.getElementById(id);
        if (el) el.textContent = txt;
      });
    }

    function tickerCardHTML(t, showChange) {
      const d   = marketsData[t.id];
      if (!d) return `<div class="ticker-card"><div class="ticker-name">${t.name}</div><div class="ticker-change">—</div></div>`;
      const up  = d.pct >= 0;
      const cls = up ? 'up' : 'down';
      const arrow = up ? '▲' : '▼';
      const changeStr = (showChange && d.change != null)
        ? `${d.change >= 0 ? '+' : ''}${d.change.toFixed(2)} · `
        : '';
      return `<div class="ticker-card ${cls}">
        <div class="ticker-name">${t.name}</div>
        <div class="ticker-change">${arrow} ${changeStr}${Math.abs(d.pct).toFixed(2)}%</div>
        <div class="ticker-price">${formatTickerPrice(t, d.price)}</div>
      </div>`;
    }

    function renderMarketsStrip() {
      const strip = document.getElementById('ticker-strip');
      if (strip) strip.innerHTML = TICKERS.map(t => tickerCardHTML(t, false)).join('');
    }

    function renderMarketsTab() {
      const container = document.getElementById('markets-groups');
      if (!container) return;
      const groups = { indices: 'Indices', crypto: 'Crypto', commodities: 'Commodities' };
      container.innerHTML = Object.entries(groups).map(([key, label]) => {
        const cards = TICKERS.filter(t => t.group === key).map(t => tickerCardHTML(t, true)).join('');
        return `<div class="market-group">
          <div class="markets-section-label">${label}</div>
          <div class="market-cards">${cards}</div>
        </div>`;
      }).join('');
    }

    async function loadMarkets() {
      if (Object.keys(marketsData).length > 0) {
        renderMarketsStrip();
        renderMarketsTab();
      } else {
        showMarketsSkeleton();
      }
      if (marketsLoading) return;
      marketsLoading = true;
      let stale = false;
      try {
        await fetchMarkets();
        renderMarketsStrip();
        renderMarketsTab();
      } catch(e) {
        console.warn('loadMarkets failed', e);
        stale = true;
      } finally {
        marketsLoading = false;
        updateMarketsTimestamp(stale);
      }
    }
```

- [ ] **Step 2: Commit**

```bash
git add warroom.html
git commit -m "feat: add renderMarketsStrip, renderMarketsTab, loadMarkets"
```

---

### Task 7: Wire into navigate and INIT

**Files:**
- Modify: `warroom.html` — navigate function and INIT block

- [ ] **Step 1: Update the navigate function**

Find this block (currently ~line 988):
```js
      if (tab === 'home') { renderHome(); startNewsRefresh(); }
      else { stopNewsRefresh(); renderSection(tab); }
```
Replace with:
```js
      if (tab === 'home')    { renderHome(); startNewsRefresh(); renderMarketsStrip(); }
      else if (tab === 'markets') { stopNewsRefresh(); loadMarkets(); }
      else { stopNewsRefresh(); renderSection(tab); }
```

- [ ] **Step 2: Update the INIT block**

Find the INIT block (currently ~line 1751):
```js
    navigate('home');
    setInterval(() => { if (currentTab === 'home') renderHome(); }, 60000);
```
Replace with:
```js
    navigate('home');
    loadMarkets();
    setInterval(() => { if (currentTab === 'home') renderHome(); }, 60000);
    setInterval(() => { if (currentTab === 'home' || currentTab === 'markets') loadMarkets(); }, 5 * 60 * 1000);
```

- [ ] **Step 3: Commit**

```bash
git add warroom.html
git commit -m "feat: wire loadMarkets into navigate and INIT with 5min refresh"
```

---

### Task 8: Manual test and merge

**Files:**
- Working directory: `C:\Users\samjl\OneDrive\Documents\War Room Planning\`

- [ ] **Step 1: Open warroom.html in browser and verify**

Open `../war-room-markets/warroom.html` in a browser. Check:
- Home screen shows the Markets strip below the summary grid with 9 tickers
- Ticker cards show name, % change (green/red), and price
- Markets tab appears in the tab bar and shows grouped cards (Indices, Crypto, Commodities)
- Refresh button triggers a reload
- Timestamp updates after load

- [ ] **Step 2: Push branch to remote**

```bash
cd ../war-room-markets
git push origin war-room-markets
```

- [ ] **Step 3: Merge to master**

```bash
cd "C:\Users\samjl\OneDrive\Documents\War Room Planning"
git merge war-room-markets
git push origin master
```

- [ ] **Step 4: Clean up worktree**

```bash
git worktree remove ../war-room-markets
git branch -d war-room-markets
```
