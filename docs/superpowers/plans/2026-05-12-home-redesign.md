# Home Screen Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remove the Habits feature, restructure the home screen into a two-column layout (left: 2×2 summary cards + mini calendar; right: news), and add a navigable mini calendar panel with event dot indicators.

**Architecture:** All code lives in `warroom.html` (single-file app). Changes are grouped into: CSS removal, CSS additions, HTML restructure, JS removal, JS additions, and `renderHome()` update. Each task commits independently.

**Tech Stack:** Vanilla JS, CSS custom properties, existing `Store` helper and `wr_calevents` data.

---

### Task 1: Set up worktree

**Files:**
- Working directory: `C:\Users\samjl\OneDrive\Documents\War Room Planning\`

- [ ] **Step 1: Create the worktree branch**

```bash
cd "C:\Users\samjl\OneDrive\Documents\War Room Planning"
git worktree add ../war-room-home-redesign -b war-room-home-redesign
```

- [ ] **Step 2: Verify worktree exists**

```bash
git worktree list
```

Expected output includes a line ending with `[war-room-home-redesign]`.

- [ ] **Step 3: All subsequent steps work in the worktree folder**

```bash
cd ../war-room-home-redesign
```

---

### Task 2: Remove Habits CSS and HTML

**Files:**
- Modify: `warroom.html` — CSS block lines ~599–625, tab pill line ~834, section HTML lines ~948–957

- [ ] **Step 1: Remove the Habits CSS block**

Find and delete this entire block (lines ~599–625):
```css
    /* ── Habits ── */
    #habits-add-row { display: flex; gap: 8px; margin-bottom: 16px; }
    #habits-add-row .input { flex: 1; }
    #habits-summary { font-size: 13px; color: var(--text3); margin-bottom: 14px; }
    .habit-row {
      display: flex; align-items: center; gap: 12px;
      padding: 12px 14px; background: var(--surface);
      border-radius: 8px; margin-bottom: 6px;
    }
    .habit-check {
      width: 20px; height: 20px; border: 2px solid var(--border2);
      border-radius: 6px; cursor: pointer; flex-shrink: 0;
      display: flex; align-items: center; justify-content: center;
      transition: background 0.15s, border-color 0.15s;
    }
    .habit-check.done { background: var(--green); border-color: var(--green); }
    .habit-check.done::after { content: '✓'; color: #fff; font-size: 12px; }
    .habit-name { flex: 1; font-size: 14px; }
    .habit-dots { display: flex; gap: 4px; }
    .habit-dot {
      width: 8px; height: 8px; border-radius: 50%;
      background: var(--green);
    }
    .habit-dot.empty { background: none; border: 1.5px solid var(--border2); }
    .habit-del { background: none; border: none; color: var(--text3); cursor: pointer; font-size: 16px; }
    .habit-del:hover { color: var(--danger); }
```

- [ ] **Step 2: Remove the Habits tab pill**

Find and delete this line (~line 834):
```html
      <div class="tab" data-tab="habits"><span class="tab-icon">⬜</span><span class="tab-label"> Habits</span></div>
```

- [ ] **Step 3: Remove the Habits section HTML**

Find and delete this entire block (~lines 948–957):
```html
  <div class="section" id="section-habits">
    <h2 class="section-heading">Habits</h2>
    <div id="habits-add-row">
      <input class="input" id="habit-input" placeholder="New habit name…" />
      <button class="btn btn-primary" onclick="addHabit()">Add</button>
    </div>
    <div id="habits-summary"></div>
    <div id="habits-list"></div>
    <p id="habits-empty" style="color:var(--text3);display:none">No habits added yet.</p>
  </div>
```

- [ ] **Step 4: Commit**

```bash
git add warroom.html
git commit -m "feat: remove Habits tab, section HTML, and CSS"
```

---

### Task 3: Remove Habits JS

**Files:**
- Modify: `warroom.html` — JS section, the `/* HABITS */` block and `renderSection` map

- [ ] **Step 1: Remove the entire Habits JS block**

Find and delete this entire block (lines ~1681–1761):
```js
    /* ══════════════════════════════════════
       HABITS
    ══════════════════════════════════════ */

    function last7Days() {
      return Array.from({ length: 7 }, (_, i) => {
        const d = new Date();
        d.setDate(d.getDate() - (6 - i));
        return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`;
      });
    }

    document.addEventListener('DOMContentLoaded', () => {
      const hi = document.getElementById('habit-input');
      if (hi) hi.addEventListener('keydown', e => { if (e.key === 'Enter') addHabit(); });
    });

    function addHabit() {
      const input = document.getElementById('habit-input');
      const name  = input.value.trim();
      if (!name) return;
      const habits = Store.get('wr_habits', []);
      habits.push({ id: uid(), name });
      Store.set('wr_habits', habits);
      input.value = '';
      renderHabits();
    }

    function toggleHabit(id) {
      const today       = todayStr();
      const completions = Store.get('wr_completions', {});
      if (!completions[today]) completions[today] = [];
      const idx = completions[today].indexOf(id);
      if (idx === -1) completions[today].push(id);
      else completions[today].splice(idx, 1);
      Store.set('wr_completions', completions);
      renderHabits();
    }

    function deleteHabit(id) {
      Store.set('wr_habits', Store.get('wr_habits', []).filter(h => h.id !== id));
      renderHabits();
    }

    function renderHabits() {
      const habits      = Store.get('wr_habits', []);
      const completions = Store.get('wr_completions', {});
      const today       = todayStr();
      const days        = last7Days();
      const todayDone   = (completions[today] || []);
      const list  = document.getElementById('habits-list');
      const empty = document.getElementById('habits-empty');
      const sumEl = document.getElementById('habits-summary');
      sumEl.textContent = habits.length
        ? `${todayDone.length} of ${habits.length} done today`
        : '';
      if (!habits.length) { list.innerHTML = ''; empty.style.display = ''; return; }
      empty.style.display = 'none';
      list.innerHTML = habits.map(h => {
        const isDone = todayDone.includes(h.id);
        const dots   = days.map(day => {
          const done = (completions[day] || []).includes(h.id);
          return `<div class="habit-dot${done ? '' : ' empty'}" title="${day}"></div>`;
        }).join('');
        return `
          <div class="habit-row">
            <div class="habit-check${isDone ? ' done' : ''}" onclick="toggleHabit('${h.id}')"></div>
            <span class="habit-name">${escHtml(h.name)}</span>
            <div class="habit-dots">${dots}</div>
            <button class="habit-del" onclick="deleteHabit('${h.id}')">×</button>
          </div>`;
      }).join('');
    }

    function getHabitsSummary() {
      const habits      = Store.get('wr_habits', []);
      const completions = Store.get('wr_completions', {});
      const today       = todayStr();
      const done        = (completions[today] || []).length;
      if (!habits.length) return [{ text: 'No habits set', urgent: false }];
      return [{ text: `${done} of ${habits.length} done today`, urgent: false }];
    }
```

- [ ] **Step 2: Remove `habits: renderHabits` from renderSection**

Find:
```js
    function renderSection(tab) {
      const fns = { tasks: renderTasks, watchlist: renderWatchlist,
                    calendar: renderCalendar, notes: renderNotes,
                    habits: renderHabits, links: renderLinks };
      if (fns[tab]) fns[tab]();
    }
```

Replace with:
```js
    function renderSection(tab) {
      const fns = { tasks: renderTasks, watchlist: renderWatchlist,
                    calendar: renderCalendar, notes: renderNotes,
                    links: renderLinks };
      if (fns[tab]) fns[tab]();
    }
```

- [ ] **Step 3: Commit**

```bash
git add warroom.html
git commit -m "feat: remove Habits JS functions and renderSection entry"
```

---

### Task 4: Add two-column layout CSS and mini calendar CSS

**Files:**
- Modify: `warroom.html` — CSS section

- [ ] **Step 1: Update `#home-grid` from auto-fill to fixed 2-column**

Find:
```css
    #home-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
      gap: 14px;
      margin-bottom: 24px;
    }
```

Replace with:
```css
    #home-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 14px;
    }
```

- [ ] **Step 2: Add two-column layout and mini calendar CSS**

Find the closing `  </style>` tag and insert before it:

```css
    /* ── Home Two-Column Layout ── */
    #home-body {
      display: flex;
      gap: 20px;
      margin-bottom: 20px;
    }
    #home-left-col {
      flex: 3;
      display: flex;
      flex-direction: column;
      gap: 16px;
      min-width: 0;
    }
    #home-right-col {
      flex: 2;
      min-width: 0;
    }
    @media (max-width: 900px) {
      #home-body { flex-direction: column; }
      #home-right-col { flex: none; }
    }

    /* ── Mini Calendar (Home) ── */
    #home-mini-cal {
      background: var(--surface);
      border-radius: 12px;
      padding: 16px;
    }
    .mini-cal-section-label {
      font-family: 'Syne', sans-serif;
      font-size: 11px;
      font-weight: 700;
      letter-spacing: 1.5px;
      text-transform: uppercase;
      color: var(--teal);
      margin-bottom: 10px;
    }
    .mini-cal-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 10px;
    }
    .mini-cal-month-label {
      font-size: 13px;
      font-weight: 600;
      color: var(--text);
    }
    .mini-cal-nav-btn {
      background: none;
      border: none;
      color: var(--text2);
      font-size: 18px;
      cursor: pointer;
      padding: 0 6px;
      line-height: 1;
    }
    .mini-cal-nav-btn:hover { color: var(--accent); }
    .mini-cal-grid {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 3px;
    }
    .mini-cal-dow {
      text-align: center;
      font-size: 10px;
      font-weight: 700;
      color: var(--text3);
      padding: 3px 0;
    }
    .mini-cal-day {
      text-align: center;
      font-size: 12px;
      color: var(--text2);
      padding: 5px 0;
      border-radius: 6px;
      cursor: pointer;
      position: relative;
      line-height: 1.2;
    }
    .mini-cal-day:hover { background: var(--surface2); }
    .mini-cal-day.today {
      background: var(--accent-bg);
      color: var(--accent);
      font-weight: 700;
    }
    .mini-cal-day.other-month {
      color: var(--text3);
      cursor: default;
    }
    .mini-cal-day.other-month:hover { background: none; }
    .mini-cal-day.has-event::after {
      content: '';
      display: block;
      width: 4px;
      height: 4px;
      border-radius: 50%;
      background: var(--accent);
      margin: 2px auto 0;
    }
```

- [ ] **Step 3: Commit**

```bash
git add warroom.html
git commit -m "style: add two-column home layout CSS and mini calendar CSS"
```

---

### Task 5: Restructure home section HTML

**Files:**
- Modify: `warroom.html` — `#section-home` HTML (~lines 841–878)

- [ ] **Step 1: Replace the home section inner HTML**

Find this entire block:
```html
  <div class="section active" id="section-home">
    <div id="home-main">
      <div id="home-hero">
        <div id="home-hero-left">
          <div id="home-clock">00:00:00</div>
          <div id="home-date"></div>
          <div id="theme-swatches">
            <span class="theme-swatch-label">THEME</span>
            <button class="theme-swatch" data-theme="dark"  onclick="setTheme('dark')"  title="Midnight"></button>
            <button class="theme-swatch" data-theme="light" onclick="setTheme('light')" title="Arctic"></button>
          </div>
        </div>
        <div id="home-hero-centre">
          <div id="home-greeting"></div>
        </div>
        <div id="home-hero-right">
          <div id="home-focus-strip"></div>
        </div>
      </div>
      <div id="home-grid">
        <!-- Cards injected by renderHome() -->
      </div>
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
      <div id="news-sidebar" style="margin-top:24px">
        <div id="news-sidebar-label">Latest News</div>
        <div id="news-list"><p style="color:var(--text3);font-size:12px">Loading news…</p></div>
      </div>
    </div>
  </div>
```

Replace with:
```html
  <div class="section active" id="section-home">
    <div id="home-main">
      <div id="home-hero">
        <div id="home-hero-left">
          <div id="home-clock">00:00:00</div>
          <div id="home-date"></div>
          <div id="theme-swatches">
            <span class="theme-swatch-label">THEME</span>
            <button class="theme-swatch" data-theme="dark"  onclick="setTheme('dark')"  title="Midnight"></button>
            <button class="theme-swatch" data-theme="light" onclick="setTheme('light')" title="Arctic"></button>
          </div>
        </div>
        <div id="home-hero-centre">
          <div id="home-greeting"></div>
        </div>
        <div id="home-hero-right">
          <div id="home-focus-strip"></div>
        </div>
      </div>
      <div id="home-body">
        <div id="home-left-col">
          <div id="home-grid">
            <!-- Cards injected by renderHome() -->
          </div>
          <div id="home-mini-cal">
            <!-- Mini calendar injected by renderMiniCalendar() -->
          </div>
        </div>
        <div id="home-right-col">
          <div id="news-sidebar">
            <div id="news-sidebar-label">Latest News</div>
            <div id="news-list"><p style="color:var(--text3);font-size:12px">Loading news…</p></div>
          </div>
        </div>
      </div>
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
    </div>
  </div>
```

- [ ] **Step 2: Commit**

```bash
git add warroom.html
git commit -m "feat: restructure home HTML into two-column layout"
```

---

### Task 6: Add mini calendar JS

**Files:**
- Modify: `warroom.html` — JS section, just before the `/* ══ INIT ══ */` comment block

- [ ] **Step 1: Insert mini calendar state and functions**

Find the `/* ══════════════════════════════════════\n       INIT` comment block. Insert before it:

```js
    /* ══════════════════════════════════════
       MINI CALENDAR (HOME)
    ══════════════════════════════════════ */
    let miniCalYear  = new Date().getFullYear();
    let miniCalMonth = new Date().getMonth();

    function miniCalPrev() {
      miniCalMonth--;
      if (miniCalMonth < 0) { miniCalMonth = 11; miniCalYear--; }
      renderMiniCalendar();
    }

    function miniCalNext() {
      miniCalMonth++;
      if (miniCalMonth > 11) { miniCalMonth = 0; miniCalYear++; }
      renderMiniCalendar();
    }

    function renderMiniCalendar() {
      const container = document.getElementById('home-mini-cal');
      if (!container) return;

      const events      = Store.get('wr_calevents', {});
      const today       = todayStr();
      const label       = new Date(miniCalYear, miniCalMonth, 1)
        .toLocaleDateString('en-GB', { month: 'long', year: 'numeric' });
      const firstDay    = new Date(miniCalYear, miniCalMonth, 1).getDay();
      const offset      = (firstDay + 6) % 7;
      const daysInMonth = new Date(miniCalYear, miniCalMonth + 1, 0).getDate();
      const prevDays    = new Date(miniCalYear, miniCalMonth, 0).getDate();

      const DAYS = ['M','T','W','T','F','S','S'];
      let daysHtml = DAYS.map(d => `<div class="mini-cal-dow">${d}</div>`).join('');

      for (let i = offset - 1; i >= 0; i--) {
        daysHtml += `<div class="mini-cal-day other-month">${prevDays - i}</div>`;
      }
      for (let d = 1; d <= daysInMonth; d++) {
        const key       = `${miniCalYear}-${String(miniCalMonth+1).padStart(2,'0')}-${String(d).padStart(2,'0')}`;
        const isToday   = key === today;
        const hasEvents = events[key] && events[key].length > 0;
        daysHtml += `<div class="mini-cal-day${isToday ? ' today' : ''}${hasEvents ? ' has-event' : ''}" onclick="navigate('calendar')">${d}</div>`;
      }
      const total     = offset + daysInMonth;
      const remaining = total % 7 === 0 ? 0 : 7 - (total % 7);
      for (let d = 1; d <= remaining; d++) {
        daysHtml += `<div class="mini-cal-day other-month">${d}</div>`;
      }

      container.innerHTML = `
        <div class="mini-cal-section-label">Calendar</div>
        <div class="mini-cal-header">
          <button class="mini-cal-nav-btn" onclick="event.stopPropagation();miniCalPrev()">‹</button>
          <span class="mini-cal-month-label">${label}</span>
          <button class="mini-cal-nav-btn" onclick="event.stopPropagation();miniCalNext()">›</button>
        </div>
        <div class="mini-cal-grid">${daysHtml}</div>
      `;
    }
```

- [ ] **Step 2: Commit**

```bash
git add warroom.html
git commit -m "feat: add renderMiniCalendar, miniCalPrev, miniCalNext"
```

---

### Task 7: Update renderHome()

**Files:**
- Modify: `warroom.html` — `renderHome` function (~lines 1557–1604)

- [ ] **Step 1: Replace the entire renderHome function**

Find:
```js
    function renderHome() {
      updateClock();

      // Daily focus strip
      const focusStrip = document.getElementById('home-focus-strip');
      if (focusStrip) {
        const activeTasks  = Store.get('wr_todos', []).filter(t => !t.done).length;
        const habits       = Store.get('wr_habits', []);
        const completions  = Store.get('wr_completions', {});
        const habitsDone   = (completions[todayStr()] || []).length;
        const todayEvents  = (Store.get('wr_calevents', {})[todayStr()] || []).length;
        const pills = [];
        if (activeTasks)   pills.push(`${activeTasks} task${activeTasks !== 1 ? 's' : ''}`);
        if (habits.length) pills.push(`${habitsDone}/${habits.length} habits`);
        if (todayEvents)   pills.push(`${todayEvents} event${todayEvents !== 1 ? 's' : ''} today`);
        focusStrip.innerHTML = pills.length
          ? pills.map(p => `<span class="focus-pill">${p}</span>`).join('')
          : '<span class="focus-pill">All clear ✓</span>';
      }

      const grid = document.getElementById('home-grid');
      const cards = [
        { tab: 'tasks',     label: 'Tasks',     color: 'var(--blue)',   items: getTaskSummary()     },
        { tab: 'watchlist', label: 'Watchlist',  color: 'var(--purple)', items: getWatchSummary()    },
        { tab: 'calendar',  label: 'Calendar',   color: 'var(--teal)',   items: getCalSummary()      },
        { tab: 'notes',     label: 'Notes',      color: 'var(--amber)',  items: getNotesSummary()    },
        { tab: 'habits',    label: 'Habits',     color: 'var(--green)',  items: getHabitsSummary()   },
        { tab: 'links',     label: 'Links',      color: 'var(--coral)',  items: getLinksSummary()    },
      ];
      const CARD_ICONS = {
        tasks: '✓', watchlist: '👁', calendar: '📅',
        notes: '📝', habits: '⬜', links: '🔗'
      };
      grid.innerHTML = cards.map(c => `
        <div class="home-card" style="border-left-color:${c.color}" onclick="navigate('${c.tab}')">
          <div class="home-card-header" style="color:${c.color}">
            <span>${CARD_ICONS[c.tab]}</span> ${c.label}
          </div>
          <div class="home-card-body">
            ${c.items.length && !(c.items.length === 1 && c.items[0].text === 'Nothing yet')
              ? c.items.map(i => `<div class="home-card-item${i.urgent ? ' urgent' : ''}">${escHtml(i.text)}</div>`).join('')
              : `<div class="home-card-empty">Nothing yet</div>
                 <button class="home-card-add" onclick="event.stopPropagation();navigate('${c.tab}')">+ Add</button>`
            }
          </div>
        </div>
      `).join('');
    }
```

Replace with:
```js
    function renderHome() {
      updateClock();

      // Daily focus strip
      const focusStrip = document.getElementById('home-focus-strip');
      if (focusStrip) {
        const activeTasks = Store.get('wr_todos', []).filter(t => !t.done).length;
        const todayEvents = (Store.get('wr_calevents', {})[todayStr()] || []).length;
        const pills = [];
        if (activeTasks) pills.push(`${activeTasks} task${activeTasks !== 1 ? 's' : ''}`);
        if (todayEvents) pills.push(`${todayEvents} event${todayEvents !== 1 ? 's' : ''} today`);
        focusStrip.innerHTML = pills.length
          ? pills.map(p => `<span class="focus-pill">${p}</span>`).join('')
          : '<span class="focus-pill">All clear ✓</span>';
      }

      const grid = document.getElementById('home-grid');
      const cards = [
        { tab: 'tasks',     label: 'Tasks',     color: 'var(--blue)',   items: getTaskSummary()  },
        { tab: 'watchlist', label: 'Watchlist',  color: 'var(--purple)', items: getWatchSummary() },
        { tab: 'notes',     label: 'Notes',      color: 'var(--amber)',  items: getNotesSummary() },
        { tab: 'links',     label: 'Links',      color: 'var(--coral)',  items: getLinksSummary() },
      ];
      const CARD_ICONS = { tasks: '✓', watchlist: '👁', notes: '📝', links: '🔗' };
      grid.innerHTML = cards.map(c => `
        <div class="home-card" style="border-left-color:${c.color}" onclick="navigate('${c.tab}')">
          <div class="home-card-header" style="color:${c.color}">
            <span>${CARD_ICONS[c.tab]}</span> ${c.label}
          </div>
          <div class="home-card-body">
            ${c.items.length && !(c.items.length === 1 && c.items[0].text === 'Nothing yet')
              ? c.items.map(i => `<div class="home-card-item${i.urgent ? ' urgent' : ''}">${escHtml(i.text)}</div>`).join('')
              : `<div class="home-card-empty">Nothing yet</div>
                 <button class="home-card-add" onclick="event.stopPropagation();navigate('${c.tab}')">+ Add</button>`
            }
          </div>
        </div>
      `).join('');

      miniCalYear  = new Date().getFullYear();
      miniCalMonth = new Date().getMonth();
      renderMiniCalendar();
    }
```

- [ ] **Step 2: Commit**

```bash
git add warroom.html
git commit -m "feat: update renderHome — remove habits/calendar cards, add mini calendar render"
```

---

### Task 8: Manual test and merge

**Files:**
- Working directory: `C:\Users\samjl\OneDrive\Documents\War Room Planning\`

- [ ] **Step 1: Open warroom.html in browser and verify**

Open `../war-room-home-redesign/warroom.html` in a browser. Check:
- No Habits tab in the tab bar
- Home screen shows two-column layout: left (2×2 cards) + right (news)
- Mini calendar appears below the summary cards with month label and ◀ ▶ navigation
- Days with events show a dot beneath the number
- Today's date is highlighted
- Clicking ◀ / ▶ changes the month
- Clicking any day navigates to the Calendar tab
- Markets strip is full-width at the bottom
- Focus strip pills show tasks + events only (no habits)
- Both Midnight and Arctic themes render the mini calendar correctly

- [ ] **Step 2: Push branch to remote**

```bash
cd ../war-room-home-redesign
git push origin war-room-home-redesign
```

- [ ] **Step 3: Merge to master**

```bash
cd "C:\Users\samjl\OneDrive\Documents\War Room Planning"
git merge war-room-home-redesign
git push origin master
```

- [ ] **Step 4: Clean up worktree**

```bash
git worktree remove ../war-room-home-redesign
git branch -d war-room-home-redesign
```
