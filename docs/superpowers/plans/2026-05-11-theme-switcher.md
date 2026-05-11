# Theme Switcher Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the binary light/dark toggle with two named themes (Midnight and Arctic) selectable via colour swatches on the Home screen, with choice synced to Supabase.

**Architecture:** All code lives in `warroom.html`. The site already uses `html[data-theme]` CSS custom properties — we update the colour values and replace the tab-bar button with Home screen swatches. The existing `Store` helper handles Supabase sync automatically.

**Tech Stack:** Vanilla JS, CSS custom properties, existing Supabase Store helper.

---

### Task 1: Set up worktree

**Files:**
- Working directory: `C:\Users\samjl\OneDrive\Documents\War Room Planning\`

- [ ] **Step 1: Create the worktree branch**

```bash
cd "C:\Users\samjl\OneDrive\Documents\War Room Planning"
git worktree add ../war-room-themes -b war-room-themes
```

- [ ] **Step 2: Verify worktree exists**

```bash
git worktree list
```

Expected output includes: `../war-room-themes  [war-room-themes]`

- [ ] **Step 3: All subsequent steps work in the worktree folder**

```bash
cd ../war-room-themes
```

---

### Task 2: Update CSS theme colour values

The existing CSS has `html[data-theme="light"]` and `html[data-theme="dark"]` blocks. This task updates both to the Midnight and Arctic palette.

**Files:**
- Modify: `warroom.html` — CSS blocks at lines ~27-52

- [ ] **Step 1: Replace the light theme block**

Find:
```css
    /* Light theme */
    html[data-theme="light"] {
      --bg:         #ffffff;
      --surface:    #f1f5f9;
      --surface2:   #dde3ec;
      --border:     #e2e8f0;
      --border2:    #cbd5e1;
      --text:       #0f172a;
      --text2:      #64748b;
      --text3:      #94a3b8;
      --accent:     #3b82f6;
      --accent-bg:  #eff6ff;
    }
```
Replace with:
```css
    /* Arctic theme (light) */
    html[data-theme="light"] {
      --bg:         #f8fafc;
      --surface:    #f1f5f9;
      --surface2:   #e2e8f0;
      --border:     #e2e8f0;
      --border2:    #cbd5e1;
      --text:       #0f172a;
      --text2:      #475569;
      --text3:      #94a3b8;
      --accent:     #3b82f6;
      --accent-bg:  #eff6ff;
    }
```

- [ ] **Step 2: Add theme swatch CSS**

Find the closing `  </style>` tag and insert before it:

```css
    /* ── Theme Swatches ── */
    #theme-swatches {
      display: flex;
      align-items: center;
      gap: 8px;
      margin-top: 10px;
    }
    .theme-swatch-label {
      font-size: 10px;
      font-weight: 700;
      letter-spacing: 1px;
      color: var(--text3);
    }
    .theme-swatch {
      width: 20px;
      height: 20px;
      border-radius: 50%;
      border: 2px solid transparent;
      cursor: pointer;
      padding: 0;
      transition: transform 0.15s, outline 0.15s;
      outline: 2px solid transparent;
      outline-offset: 2px;
    }
    .theme-swatch[data-theme="dark"]  { background: #18181b; border-color: #a855f7; }
    .theme-swatch[data-theme="light"] { background: #f8fafc; border-color: #3b82f6; }
    .theme-swatch.active { outline-color: var(--accent); }
    .theme-swatch:hover  { transform: scale(1.15); }
```

- [ ] **Step 3: Commit**

```bash
git add warroom.html
git commit -m "style: update Arctic light theme palette and add theme swatch CSS"
```

---

### Task 3: Replace theme toggle button with Home screen swatches

**Files:**
- Modify: `warroom.html` — tab bar HTML and home-hero HTML

- [ ] **Step 1: Remove the theme toggle button from the tab bar**

Find and delete this line (~line 790):
```html
    <button id="theme-toggle" onclick="toggleTheme()" title="Toggle theme">🌙</button>
```

- [ ] **Step 2: Add theme swatches to the Home hero**

Find the `#home-hero-left` block:
```html
        <div id="home-hero-left">
          <div id="home-clock">00:00:00</div>
          <div id="home-date"></div>
        </div>
```
Replace with:
```html
        <div id="home-hero-left">
          <div id="home-clock">00:00:00</div>
          <div id="home-date"></div>
          <div id="theme-swatches">
            <span class="theme-swatch-label">THEME</span>
            <button class="theme-swatch" data-theme="dark"  onclick="setTheme('dark')"  title="Midnight"></button>
            <button class="theme-swatch" data-theme="light" onclick="setTheme('light')" title="Arctic"></button>
          </div>
        </div>
```

- [ ] **Step 3: Commit**

```bash
git add warroom.html
git commit -m "feat: replace tab-bar theme toggle with Home screen swatches"
```

---

### Task 4: Replace theme JS with setTheme function

**Files:**
- Modify: `warroom.html` — the `/* ── THEME ── */` JS section (~lines 937-955)

- [ ] **Step 1: Replace the entire theme JS block**

Find this entire block:
```js
    /* ── THEME ── */
    (function () {
      const saved = localStorage.getItem('wr_theme');
      const preferred = window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
      const active = saved || preferred;
      document.documentElement.dataset.theme = active;
      document.addEventListener('DOMContentLoaded', () => {
        const btn = document.getElementById('theme-toggle');
        if (btn) btn.textContent = active === 'dark' ? '☀️' : '🌙';
      });
    })();

    function toggleTheme() {
      const next = document.documentElement.dataset.theme === 'dark' ? 'light' : 'dark';
      document.documentElement.dataset.theme = next;
      localStorage.setItem('wr_theme', next);
      const btn = document.getElementById('theme-toggle');
      if (btn) btn.textContent = next === 'dark' ? '☀️' : '🌙';
    }
```

Replace with:
```js
    /* ── THEME ── */
    (function initTheme() {
      const saved     = localStorage.getItem('wr_theme');
      const preferred = window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
      let active;
      try { active = saved ? JSON.parse(saved) : null; } catch { active = null; }
      document.documentElement.dataset.theme = active || preferred;
    })();

    function setTheme(name) {
      document.documentElement.dataset.theme = name;
      Store.set('wr_theme', name);
      document.querySelectorAll('.theme-swatch').forEach(s => {
        s.classList.toggle('active', s.dataset.theme === name);
      });
    }

    function initThemeSwatches() {
      const current = document.documentElement.dataset.theme;
      document.querySelectorAll('.theme-swatch').forEach(s => {
        s.classList.toggle('active', s.dataset.theme === current);
      });
    }
```

- [ ] **Step 2: Commit**

```bash
git add warroom.html
git commit -m "feat: replace toggleTheme with setTheme, sync via Store to Supabase"
```

---

### Task 5: Call initThemeSwatches on INIT

**Files:**
- Modify: `warroom.html` — INIT block (~line 1751)

- [ ] **Step 1: Add initThemeSwatches call**

Find:
```js
    navigate('home');
```
Add after it:
```js
    initThemeSwatches();
```

- [ ] **Step 2: Commit**

```bash
git add warroom.html
git commit -m "feat: call initThemeSwatches on init to highlight active swatch"
```

---

### Task 6: Manual test and merge

**Files:**
- Working directory: `C:\Users\samjl\OneDrive\Documents\War Room Planning\`

- [ ] **Step 1: Open warroom.html in browser and verify**

Open `../war-room-themes/warroom.html` in a browser. Check:
- No theme toggle button in the tab bar
- Two circular swatches visible on the Home screen below the clock/date
- Clicking Midnight swatch applies dark theme instantly
- Clicking Arctic swatch applies light theme — white/slate background, blue accent
- Active swatch shows a visible ring/outline
- Refreshing the page preserves the selected theme
- Opening browser devtools → Application → Local Storage: `wr_theme` key exists with correct value
- Supabase table editor shows `wr_theme` row with the selected theme value

- [ ] **Step 2: Push branch to remote**

```bash
cd ../war-room-themes
git push origin war-room-themes
```

- [ ] **Step 3: Merge to master**

```bash
cd "C:\Users\samjl\OneDrive\Documents\War Room Planning"
git merge war-room-themes
git push origin master
```

- [ ] **Step 4: Clean up worktree**

```bash
git worktree remove ../war-room-themes
git branch -d war-room-themes
```
