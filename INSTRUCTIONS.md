# Agent Prompt: Build a Match-3 Game (Exact Specification)

IGNORE THE CONTENTS OF match3.html FROM NOW ON. THEY ARE AN IRRELEVANT BETA VERSION.

You are to produce a single self-contained HTML file implementing a fully functional two-player Match-3 puzzle game — a human player vs an AI opponent — with a countdown timer, start/end modals, a persistent leaderboard, a Web Audio API sound engine, and full mobile responsiveness. Follow every instruction below exactly. Do not add features, do not change the visual design, do not introduce external libraries beyond the fonts specified.

---

## Step 1 — File structure

One file: `index.html`. Contents in order:
1. Standard HTML5 `<!DOCTYPE html>` boilerplate.
2. `<link>` loading two Google Fonts: `DM Mono` (weights 400, 500) and `DM Sans` (weights 400, 500, 600).
3. A single `<style>` block — ALL CSS, ending with the mobile `@media` block.
4. All HTML markup directly inside `<body>`: start modal, end modal, then `<div id="app">`.
5. A single `<script>` block at the end of `<body>` — ALL JavaScript.

No external JS libraries. No separate files.

---

## Step 2 — CSS variables and base reset

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
html { height: 100%; }
```

Declare on `:root`:
```
--bg: #0f0f13
--surface: #1a1a22
--surface2: #22222e
--border: rgba(255,255,255,0.08)
--border-strong: rgba(255,255,255,0.18)
--text: #f0eff4
--text-muted: #7c7a8e
--accent: #a78bfa
--accent-glow: rgba(167,139,250,0.25)
--ai-accent: #fb923c
--ai-glow: rgba(251,146,60,0.25)
--cell-size: clamp(28px, 10.5vw, 44px)
--gap: 2px
```

`--cell-size: clamp(28px, 10.5vw, 44px)` makes the player grid fluid — it shrinks automatically on narrow screens with no JavaScript. The AI mini-grid on mobile uses its own inline `clamp` value (see Step 13).

---

## Step 3 — Body and background

```css
body {
  background: var(--bg);
  color: var(--text);
  font-family: 'DM Sans', sans-serif;
  min-height: 100%;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding: 1rem 0.75rem 2rem;
  touch-action: manipulation;
}
```

`touch-action: manipulation` removes the 300ms tap delay on mobile. `align-items: flex-start` prevents vertical centering on short phone screens.

`body::before` decorative grid overlay: `position: fixed; inset: 0; pointer-events: none`, two overlapping `linear-gradient` lines at `rgba(255,255,255,0.02)`, `background-size: 40px 40px`.

---

## Step 4 — HTML layout

### 4a — Start modal (shown on load)

```html
<div class="modal-bg show" id="start-modal">
  <div class="modal">
    <div class="start-gems">🔴🟠🟡🟢🔵🟣</div>
    <div class="modal-eyebrow">Challenge</div>
    <div class="modal-title">Match Three</div>
    <div class="modal-sub">Beat the AI. You have 30 seconds.</div>
    <ul class="rules-list">
      <li>Click two adjacent gems to swap</li>
      <li>Clear 3+ in a row or column</li>
      <li>Score more than the AI to win</li>
      <li>AI makes a move every 3–6 seconds</li>
    </ul>
    <button class="modal-primary-btn" onclick="startGame()">Start Game</button>
  </div>
</div>
```

### 4b — End modal (hidden by default)

```html
<div class="modal-bg" id="end-modal">
  <div class="modal">
    <div class="modal-eyebrow" id="end-eyebrow">Round Over</div>
    <div class="modal-title" id="end-title">—</div>
    <div class="modal-sub" id="end-sub"></div>
    <div class="modal-scores">
      <div class="modal-stat">
        <span class="modal-stat-label">You</span>
        <span class="modal-stat-val you" id="end-you-score">0</span>
      </div>
      <div class="modal-stat">
        <span class="modal-stat-label">AI</span>
        <span class="modal-stat-val ai" id="end-ai-score">0</span>
      </div>
    </div>
    <button class="modal-primary-btn" onclick="restartGame()">Play Again</button>
  </div>
</div>
```

### 4c — Main app

```html
<div id="app">

  <header>
    <h1>Match Three</h1>
    <div id="tagline">beat the ai · clear 3 or more</div>
  </header>

  <!-- desktop timer bar (hidden on mobile) -->
  <div id="timer-bar">
    <span id="timer-label">Time</span>
    <div id="timer-track"><div id="timer-fill"></div></div>
    <span id="timer-val">30</span>
  </div>

  <!-- mobile-only top bar: your score | shrinking timer | AI score -->
  <div id="mobile-bar">
    <div class="mob-stat">
      <span class="mob-label">You</span>
      <span class="mob-val you-col" id="mob-you-score">0</span>
    </div>
    <div class="mob-timer">
      <div id="mob-timer-track"><div id="mob-timer-fill"></div></div>
      <span id="mob-timer-val">30</span>
    </div>
    <div class="mob-stat">
      <span class="mob-label">AI</span>
      <span class="mob-val ai-col" id="mob-ai-score">0</span>
    </div>
  </div>

  <!-- mobile-only compact leaderboard strip -->
  <div id="mob-lb">
    <span id="mob-lb-title">Leaderboard</span>
    <div class="mob-lb-entries">
      <div class="mob-lb-entry" id="mob-lb-e1">
        <span class="mob-lb-name" id="mob-lb-n1">—</span>
        <span class="mob-lb-wins" id="mob-lb-w1">0</span>
      </div>
      <div class="mob-lb-entry" id="mob-lb-e2">
        <span class="mob-lb-name" id="mob-lb-n2">—</span>
        <span class="mob-lb-wins" id="mob-lb-w2">0</span>
      </div>
    </div>
  </div>

  <div id="arena">

    <!-- AI board (left — hidden on mobile, replaced by #mob-ai-wrap) -->
    <div class="side-panel">
      <div class="panel-label ai-label">AI</div>
      <div class="score-pill ai-score" id="ai-score-val">0</div>
      <div class="board-wrap ai-board">
        <div class="grid" id="ai-grid"></div>
      </div>
    </div>

    <!-- leaderboard (centre — hidden on mobile, replaced by #mob-lb) -->
    <div id="leaderboard">
      <div id="lb-title">Leaderboard</div>
      <div class="lb-row" id="lb-row-1">
        <span class="lb-rank">#1</span>
        <div class="lb-avatar" id="lb-av-1"></div>
        <span class="lb-name" id="lb-name-1">—</span>
        <span class="lb-wins" id="lb-wins-1">0</span>
      </div>
      <div class="lb-divider">· · ·</div>
      <div class="lb-row" id="lb-row-2">
        <span class="lb-rank">#2</span>
        <div class="lb-avatar" id="lb-av-2"></div>
        <span class="lb-name" id="lb-name-2">—</span>
        <span class="lb-wins" id="lb-wins-2">0</span>
      </div>
    </div>

    <!-- player board (right — full width on mobile) -->
    <div class="side-panel">
      <div class="panel-label you-label">You</div>
      <div class="score-pill you-score" id="score-val">0</div>
      <div class="board-wrap you-board">
        <div class="grid" id="grid"></div>
      </div>
    </div>

  </div><!-- /#arena -->

  <!-- mobile-only AI mini-board (below player board) -->
  <div id="mob-ai-wrap">
    <span id="mob-ai-label">AI Board</span>
    <div id="mob-ai-board">
      <div id="mob-ai-grid"></div>
    </div>
  </div>

  <div id="bottom">
    <div id="msg"></div>
    <button class="action-btn" onclick="restartGame()">New Game</button>
  </div>

</div>
```

**Key notes:**
- `#mobile-bar`, `#mob-lb`, and `#mob-ai-wrap` are hidden by default in base CSS and revealed only inside the `@media` block.
- `#ai-grid`, `#grid`, and `#mob-ai-grid` are all empty — JavaScript fills them.
- The start modal has class `show` by default; the end modal does not.
- The desktop AI board in `#arena` remains in the DOM on mobile but is hidden via CSS. The mobile AI mini-grid (`#mob-ai-grid`) is a separate element rendered by a separate JS function.

---

## Step 5 — CSS: layout

**`#app`**: `display: flex; flex-direction: column; align-items: center; gap: 12px; position: relative; z-index: 1; width: 100%; max-width: 1100px`

**`header`**: `display: flex; flex-direction: column; align-items: center; gap: 4px`

**`h1`**: DM Mono, 13px, weight 500, `letter-spacing: 0.18em`, `text-transform: uppercase`, `color: var(--accent)`

**`#tagline`**: 11px, `color: var(--text-muted)`, `letter-spacing: 0.04em`

**`#arena`**: `display: flex; align-items: flex-start; gap: 12px; width: 100%; justify-content: center`

**`.side-panel`**: `display: flex; flex-direction: column; align-items: center; gap: 8px`

---

## Step 6 — CSS: desktop timer bar

**`#timer-bar`**: `display: flex; align-items: center; gap: 10px; font-family: DM Mono; width: 100%; max-width: 340px`

**`#timer-label`**: 10px, `letter-spacing: 0.12em`, `text-transform: uppercase`, `color: var(--text-muted)`, `white-space: nowrap`

**`#timer-track`**: `flex: 1; height: 6px; background: var(--surface2); border-radius: 99px; overflow: hidden; border: 1px solid var(--border)`

**`#timer-fill`**: `height: 100%; width: 100%; background: var(--accent); border-radius: 99px; transition: width 1s linear, background 0.4s`

**`#timer-fill.low`**: `background: #f87171`

**`#timer-val`**: DM Mono, 13px, weight 500, `color: var(--text)`, `min-width: 3ch`, `text-align: right`

---

## Step 7 — CSS: side panel labels and score pills

**`.panel-label`**: DM Mono, 10px, `letter-spacing: 0.14em`, `text-transform: uppercase`, `padding: 3px 10px`, `border-radius: 99px`, `border: 1px solid var(--border)`, `white-space: nowrap`

**`.panel-label.ai-label`**: `color: var(--ai-accent); border-color: rgba(251,146,60,0.3)`

**`.panel-label.you-label`**: `color: var(--accent); border-color: rgba(167,139,250,0.3)`

**`.score-pill`**: DM Mono, 18px, weight 500, `min-width: 4ch`, `text-align: center`

**`.score-pill.ai-score`**: `color: var(--ai-accent)`

**`.score-pill.you-score`**: `color: var(--accent)`

---

## Step 8 — CSS: boards and grids

**`.board-wrap`**: `background: var(--surface); border: 1px solid var(--border); border-radius: 14px; padding: 8px; position: relative`

**`.board-wrap.ai-board`**: `border-color: rgba(251,146,60,0.2)`

**`.board-wrap.you-board`**: `border-color: rgba(167,139,250,0.2)`

**`.grid`**: `display: grid; grid-template-columns: repeat(8, var(--cell-size)); grid-template-rows: repeat(8, var(--cell-size)); gap: var(--gap)`

---

## Step 9 — CSS: cells

**`.cell`** base:
```
width: var(--cell-size); height: var(--cell-size)
border-radius: 6px
display: flex; align-items: center; justify-content: center
font-size: clamp(13px, 4.5vw, 20px)
cursor: pointer
border: 1.5px solid transparent
background: var(--surface2)
transition: transform 0.12s ease, border-color 0.12s ease, background 0.12s ease
user-select: none
-webkit-tap-highlight-color: transparent
```

**`.cell:hover`**: `transform: scale(1.07); background: #2a2a38; border-color: var(--border-strong)`

**`.cell.selected`**: `border-color: var(--accent); background: rgba(167,139,250,0.12); transform: scale(1.1); box-shadow: 0 0 0 3px var(--accent-glow)`

**`.cell.ai-hint-a`**: `border-color: var(--ai-accent); background: rgba(251,146,60,0.1)`

**`.cell.ai-hint-b`**: `border-color: var(--ai-accent); background: rgba(251,146,60,0.2); transform: scale(1.08)`

**`.cell.matched`**: `animation: pop 0.28s ease forwards`

```css
@keyframes pop {
  0%   { transform: scale(1); opacity: 1; }
  40%  { transform: scale(1.25); opacity: 0.6; }
  100% { transform: scale(0); opacity: 0; }
}
```

---

## Step 10 — CSS: leaderboard (desktop)

**`#leaderboard`**: `background: var(--surface); border: 1px solid var(--border); border-radius: 14px; padding: 14px 12px; display: flex; flex-direction: column; gap: 10px; min-width: 120px; align-self: stretch`

**`#lb-title`**: DM Mono, 10px, `letter-spacing: 0.14em`, `text-transform: uppercase`, `color: var(--text-muted)`, `text-align: center`, `padding-bottom: 8px`, `border-bottom: 1px solid var(--border)`

**`.lb-row`**: `display: flex; align-items: center; gap: 6px; padding: 6px 8px; border-radius: 8px; background: var(--surface2); border: 1px solid var(--border)`

**`.lb-row.leader`**: `border-color: rgba(255,215,0,0.25); background: rgba(255,215,0,0.04)`

**`.lb-rank`**: DM Mono, 10px, `color: var(--text-muted)`, `min-width: 14px`

**`.lb-avatar`**: `width: 24px; height: 24px; border-radius: 50%; display: flex; align-items: center; justify-content: center; flex-shrink: 0; font-size: 10px; font-weight: 500; font-family: DM Mono`

**`.lb-avatar.ai-av`**: `background: rgba(251,146,60,0.15); color: var(--ai-accent)`

**`.lb-avatar.you-av`**: `background: rgba(167,139,250,0.15); color: var(--accent)`

**`.lb-name`**: 12px, weight 500, `flex: 1`

**`.lb-wins`**: DM Mono, 16px, weight 500

**`.lb-wins.ai-wins`**: `color: var(--ai-accent)`

**`.lb-wins.you-wins`**: `color: var(--accent)`

**`.lb-divider`**: DM Mono, 10px, `color: var(--text-muted)`, `text-align: center`, `letter-spacing: 0.08em`

**`.lb-crown`**: `font-size: 11px; margin-left: 2px`

---

## Step 11 — CSS: bottom bar and buttons

**`#bottom`**: `display: flex; align-items: center; gap: 12px; width: 100%; justify-content: space-between`

**`#msg`**: DM Mono, 11px, `color: var(--text-muted)`, `letter-spacing: 0.05em`, `min-height: 16px`, `flex: 1`

**`#msg.bad`**: `color: #f87171` · **`#msg.good`**: `color: #6ee7b7`

**`.action-btn`**: DM Mono, 11px, weight 500, `letter-spacing: 0.1em`, `text-transform: uppercase`, `color: var(--accent)`, `background: transparent`, `border: 1px solid rgba(167,139,250,0.35)`, `border-radius: 8px`, `padding: 8px 16px`, `cursor: pointer`, `transition: background 0.15s, border-color 0.15s`, `white-space: nowrap`, `-webkit-tap-highlight-color: transparent`

**`.action-btn:hover`**: `background: rgba(167,139,250,0.1); border-color: var(--accent)` · **`:active`**: `transform: scale(0.97)`

---

## Step 12 — CSS: modals

**`.modal-bg`**: `display: none; position: fixed; inset: 0; background: rgba(10,10,15,0.88); backdrop-filter: blur(6px); z-index: 100; align-items: center; justify-content: center; padding: 1rem`

**`.modal-bg.show`**: `display: flex`

**`.modal`**: `background: var(--surface); border: 1px solid var(--border-strong); border-radius: 20px; padding: 28px 24px; display: flex; flex-direction: column; align-items: center; gap: 12px; width: 100%; max-width: 340px; text-align: center`

**`.modal-eyebrow`**: DM Mono, 10px, `letter-spacing: 0.2em`, `text-transform: uppercase`, `color: var(--text-muted)`

**`.modal-title`**: DM Mono, 26px, weight 500, `color: var(--text)`, `line-height: 1.1`

**`.modal-title.win`**: `color: var(--accent)` · **`.lose`**: `color: #f87171` · **`.tie`**: `color: var(--ai-accent)`

**`.modal-sub`**: 13px, `color: var(--text-muted)`, `line-height: 1.6`, `margin-bottom: 2px`

**`.modal-scores`**: `display: flex; gap: 2px; background: var(--surface2); border: 1px solid var(--border); border-radius: 12px; overflow: hidden; width: 100%`

**`.modal-stat`**: `flex: 1; display: flex; flex-direction: column; align-items: center; padding: 10px 0; gap: 3px`

**`.modal-stat + .modal-stat`**: `border-left: 1px solid var(--border)`

**`.modal-stat-label`**: DM Mono, 10px, `letter-spacing: 0.1em`, `text-transform: uppercase`, `color: var(--text-muted)`

**`.modal-stat-val`**: DM Mono, 22px, weight 500 · **`.you`**: `color: var(--accent)` · **`.ai`**: `color: var(--ai-accent)`

**`.modal-primary-btn`**: DM Mono, 12px, weight 500, `letter-spacing: 0.12em`, `text-transform: uppercase`, `color: var(--bg)`, `background: var(--accent)`, `border: none`, `border-radius: 10px`, `padding: 14px 32px`, `cursor: pointer`, `transition: opacity 0.15s`, `width: 100%`, `margin-top: 4px`, `-webkit-tap-highlight-color: transparent`

**`.modal-primary-btn:hover`**: `opacity: 0.85`

**`.start-gems`**: `display: flex; gap: 6px; font-size: 20px; margin-bottom: 2px`

**`.rules-list`**: 12px, `color: var(--text-muted)`, `line-height: 2`, `list-style: none`, `text-align: left`, `width: 100%`. Each `li::before { content: '·  '; }`

---

## Step 13 — CSS: mobile responsive block

This is the entire mobile layout, placed at the very end of the `<style>` block.

### Default hidden state (outside the media query)

These three elements are always hidden until the media query activates them:

```css
#mobile-bar  { display: none; }
#mob-lb      { display: none; }
#mob-ai-wrap { display: none; }
```

### `@media (max-width: 600px)`

**Mobile layout order (top → bottom, no scrolling):**
1. Header (`<h1>` only, tagline hidden)
2. `#mobile-bar` — your score | timer bar | AI score
3. `#mob-lb` — compact horizontal leaderboard strip
4. `#arena` — player board only (full width)
5. `#mob-ai-wrap` — AI mini-board with small cells
6. `#bottom` — message + New Game button

```css
@media (max-width: 600px) {
  /* page shell */
  body { padding: 0.5rem 0.5rem 0.75rem; align-items: flex-start; }
  #app { gap: 6px; }

  /* hide desktop-only elements */
  #tagline                         { display: none; }
  #timer-bar                       { display: none; }
  #leaderboard                     { display: none; }
  .panel-label, .score-pill        { display: none; }
  #arena > .side-panel:first-child { display: none; }

  /* player board fills full width */
  #arena      { width: 100%; justify-content: center; }
  .side-panel { width: 100%; }
  .board-wrap { width: 100%; }
  .grid       { margin: 0 auto; }

  /* mobile top bar */
  #mobile-bar {
    display: flex; align-items: center; width: 100%;
    background: var(--surface); border: 1px solid var(--border);
    border-radius: 10px; padding: 6px 12px; gap: 10px;
  }
  .mob-stat { display: flex; flex-direction: column; align-items: center; gap: 1px; min-width: 40px; }
  .mob-label { font-family: 'DM Mono', monospace; font-size: 9px; letter-spacing: 0.12em; text-transform: uppercase; color: var(--text-muted); }
  .mob-val { font-family: 'DM Mono', monospace; font-size: 18px; font-weight: 500; line-height: 1; }
  .mob-val.you-col { color: var(--accent); }
  .mob-val.ai-col  { color: var(--ai-accent); }
  .mob-timer { flex: 1; display: flex; align-items: center; gap: 6px; }
  #mob-timer-track { flex: 1; height: 4px; background: var(--surface2); border-radius: 99px; overflow: hidden; border: 1px solid var(--border); }
  #mob-timer-fill { height: 100%; width: 100%; background: var(--accent); border-radius: 99px; transition: width 1s linear, background 0.4s; }
  #mob-timer-fill.low { background: #f87171; }
  #mob-timer-val { font-family: 'DM Mono', monospace; font-size: 12px; font-weight: 500; color: var(--text); min-width: 2ch; text-align: right; }

  /* compact leaderboard strip */
  #mob-lb {
    display: flex; align-items: center; justify-content: space-between;
    width: 100%; background: var(--surface); border: 1px solid var(--border);
    border-radius: 10px; padding: 6px 12px; gap: 8px;
  }
  #mob-lb-title { font-family: 'DM Mono', monospace; font-size: 9px; letter-spacing: 0.12em; text-transform: uppercase; color: var(--text-muted); white-space: nowrap; }
  .mob-lb-entries { display: flex; align-items: center; gap: 10px; flex: 1; justify-content: flex-end; }
  .mob-lb-entry { display: flex; align-items: center; gap: 5px; background: var(--surface2); border: 1px solid var(--border); border-radius: 8px; padding: 4px 8px; }
  .mob-lb-entry.mob-leader { border-color: rgba(255,215,0,0.3); background: rgba(255,215,0,0.04); }
  .mob-lb-name { font-size: 11px; font-weight: 500; color: var(--text); }
  .mob-lb-wins { font-family: 'DM Mono', monospace; font-size: 13px; font-weight: 500; }
  .mob-lb-wins.you-w { color: var(--accent); }
  .mob-lb-wins.ai-w  { color: var(--ai-accent); }

  /* AI mini-board */
  #mob-ai-wrap { display: flex; flex-direction: column; align-items: center; gap: 4px; width: 100%; }
  #mob-ai-label { font-family: 'DM Mono', monospace; font-size: 9px; letter-spacing: 0.12em; text-transform: uppercase; color: var(--ai-accent); }
  #mob-ai-board { background: var(--surface); border: 1px solid rgba(251,146,60,0.2); border-radius: 10px; padding: 5px; }

  /* AI grid sizing: 8 cells + 7 gaps of 2px must fit within screen width minus padding.
     Formula: cell = (100vw - 40px) / 8. Floor 18px, cap 28px. */
  #mob-ai-grid {
    display: grid;
    grid-template-columns: repeat(8, clamp(18px, calc((100vw - 40px) / 8), 28px));
    grid-template-rows:    repeat(8, clamp(18px, calc((100vw - 40px) / 8), 28px));
    gap: 2px;
  }
  #mob-ai-grid .cell {
    width:  clamp(18px, calc((100vw - 40px) / 8), 28px);
    height: clamp(18px, calc((100vw - 40px) / 8), 28px);
    font-size: clamp(9px, 2.8vw, 13px);
    border-radius: 4px;
    cursor: default;
  }
  /* disable hover on AI mini-grid — it is display-only */
  #mob-ai-grid .cell:hover { transform: none; background: var(--surface2); border-color: transparent; }

  #bottom { padding: 0 2px; }
}
```

**Overflow prevention:** The player grid uses `--cell-size: clamp(28px, 10.5vw, 44px)` — on a 390px screen that resolves to ~41px, giving `8 × 41 + 7 × 2 = ~342px` which fits inside the screen. The AI mini-grid uses `clamp(18px, calc((100vw - 40px) / 8), 28px)` — on a 390px screen that resolves to ~44px but is capped at 28px, giving `8 × 28 + 7 × 2 = 238px`. The compact UI strips (mobile-bar, mob-lb) are fixed at ~44px each. Total height budget on a 390×844 iPhone is comfortably under 800px with `#app { gap: 6px }` and `body { padding: 0.5rem }`.

---

## Step 14 — JavaScript: constants and state

```js
const GEMS = ['🔴','🟠','🟡','🟢','🔵','🟣'];
const COLS = 8, ROWS = 8, GAME_TIME = 30;

let board = [], selected = null, score = 0, busy = false, gameActive = false;
let aiBoard = [], aiScore = 0, aiBusy = false, aiTimer = null;
let timeLeft = GAME_TIME, timerInterval = null;
let wins = { you: 0, ai: 0 };
let comboCount = 0;
```

---

## Step 15 — Audio engine (Web Audio API, zero external files)

```js
const AC = new (window.AudioContext || window.webkitAudioContext)();
function resumeAC() { if (AC.state === 'suspended') AC.resume(); }
```

### `synth({ type, freq, freqEnd, startTime, duration, gainPeak, attack, decay, sustain, release })`

Creates an oscillator with an ADSR gain envelope. Connect `osc → gain → AC.destination`. Schedule frequency, apply envelope, start and stop.

### `noiseBlip(startTime, duration, gainPeak)`

Fills a mono `AudioBuffer` with random noise, routes through a `bandpass` `BiquadFilterNode` (180 Hz, Q 1.2), fades out with a `GainNode`.

### Sound events

| Function | Trigger |
|---|---|
| `sndStartJingle()` | First line of `startGame()` |
| `sndSelect()` | Player first-clicks or re-selects a gem |
| `sndSwap()` | Player confirms an adjacent swap |
| `sndNoMatch()` | Swap reversed, no matches |
| `sndMatch(count)` | Each cascade wave (player only); pitch rises with `comboCount` |
| `sndFall()` | After player gravity resolves |
| `sndTick()` | Each second when `timeLeft <= 8 && > 0` |
| `sndWin()` | Player wins the round |
| `sndLose()` | AI wins |
| `sndTie()` | Scores equal |

### `sndStartJingle()` — 2-second loud jingle

Base gain `G = 0.55`. Five simultaneous layers from `t = AC.currentTime`:
- **Bass hits** (triangle, 80→55 Hz, 0.35s): t+0, t+0.5, t+1.0, t+1.5
- **Arpeggio** (sawtooth, 0.22s): 8 notes C4→E6 spaced 0.18s
- **Chord stabs** (square, 0.38s): C major at t+0, F major at t+0.5, G major at t+1.0
- **Landing chord** (sine + detuned copy at `freq*1.004`, 0.55s): C5 E5 G5 C6 at t+1.5
- **Snare hits** (noiseBlip, 0.08s, gainPeak 0.35): t+0.25, t+0.75, t+1.25, t+1.75

---

## Step 16 — Game lifecycle

### `startGame()`
1. `sndStartJingle()` — must be first (user gesture unlocks AudioContext).
2. Remove class `show` from both modals.
3. Reset all state.
4. `board = freshBoard(); aiBoard = freshBoard()`.
5. `render(); renderAI(); renderMobAI(); updateScores(); startTimer(); scheduleAI(); setMsg('')`.

### `restartGame()`
`clearInterval(timerInterval); clearTimeout(aiTimer); startGame();`

### `freshBoard()`
Generate a random 8×8 board, re-generate until `findMatchesOn(b).length === 0`.

---

## Step 17 — Timer

### `startTimer()`
`timeLeft = GAME_TIME`, call `updateTimerUI()`, start `setInterval` at 1000ms. On each tick: decrement, call `updateTimerUI()`, call `sndTick()` if `timeLeft <= 8 && > 0`. When `timeLeft <= 0`: clear interval and `aiTimer`, set `gameActive = false; busy = true`, then `setTimeout(showEndModal, 400)`.

### `updateTimerUI()`

Updates **both** the desktop and mobile timer elements simultaneously:

```js
function updateTimerUI() {
  const pct = (timeLeft / GAME_TIME) * 100;
  document.getElementById('timer-val').textContent = timeLeft;
  const fill = document.getElementById('timer-fill');
  fill.style.width = pct + '%';
  fill.classList.toggle('low', timeLeft <= 8);
  const mf = document.getElementById('mob-timer-fill');
  const mv = document.getElementById('mob-timer-val');
  if (mf) { mf.style.width = pct + '%'; mf.classList.toggle('low', timeLeft <= 8); }
  if (mv) mv.textContent = timeLeft;
}
```

---

## Step 18 — Player board

### `render()`
Clear `#grid`, rebuild all cells row-major. Each cell: `class='cell'`, `textContent`, `dataset.r/c`, `onclick`. Add `selected` class if matches `selected` variable.

### `getCell(r, c)`
```js
return document.querySelector(`#grid [data-r="${r}"][data-c="${c}"]`);
```
The `#grid` scope prefix is mandatory — the two AI grids share the same `data-r`/`data-c` attributes.

### `onCellClick(r, c)`
1. Guard: `!gameActive || busy`.
2. No selection: `sndSelect()`, set `selected`, `render()`, return.
3. Same cell: deselect, `render()`, return.
4. Manhattan ≠ 1: `sndSelect()`, re-select, `render()`, return.
5. `selected = null; doSwap(sr, sc, r, c)`.

### `doSwap(r1, c1, r2, c2)` (async)
`busy=true; sndSwap()`. Swap. `render()`. If no matches: reverse, `render()`, `sndNoMatch()`, `setMsg('no match','bad')`, clear after 900ms, `busy=false`, return. Otherwise: `comboCount=0; await resolveMatchesOn(board,'player'); comboCount=0; busy=false`.

---

## Step 19 — AI boards

### `renderAI()`
Same structure as `render()` targeting `#ai-grid`. No `onclick` handlers. This renders the **desktop** AI grid.

### `renderMobAI()`
Same structure targeting `#mob-ai-grid`. Guards with `if (!grid) return`. No `onclick` handlers. This renders the **mobile** AI mini-grid. The CSS handles the smaller cell size — the JS simply writes the same `aiBoard` data.

```js
function renderMobAI() {
  const grid = document.getElementById('mob-ai-grid');
  if (!grid) return;
  grid.innerHTML = '';
  for (let r = 0; r < ROWS; r++) {
    for (let c = 0; c < COLS; c++) {
      const cell = document.createElement('div');
      cell.className = 'cell';
      cell.textContent = aiBoard[r][c];
      cell.dataset.r = r; cell.dataset.c = c;
      grid.appendChild(cell);
    }
  }
}
```

### `getAICell(r, c)`
```js
return document.querySelector(`#ai-grid [data-r="${r}"][data-c="${c}"]`);
```
Scoped to `#ai-grid` (the desktop grid). The mobile grid is display-only and does not need per-cell lookup.

### `scheduleAI()`
`aiTimer = setTimeout(() => aiTakeTurn(), 3000 + Math.random() * 3000)` — only if `gameActive`.

### `aiTakeTurn()` (async)
Guard `!gameActive || aiBusy`. Flash `ai-hint-a` (300ms) then `ai-hint-b` (250ms) on desktop cells. Swap. `renderAI(); renderMobAI()`. `await resolveMatchesOn(aiBoard, 'ai')`. Re-schedule.

### `findBestAIMove()`
Test all right/down adjacent pairs by temporarily swapping and calling `findMatchesOn(aiBoard)`. Return the pair with the highest match count, or `null`.

---

## Step 20 — Shared match logic

### `findMatchesOn(b)`
Board-agnostic. `Set<number>` of flat indices (`r * COLS + c`) for deduplication. Horizontal then vertical runs of 3+. Returns `[[r,c],...]`.

### `resolveMatchesOn(b, who)` (async)
Loop while matches exist. Score the correct variable. Animate matched cells. `await delay(290)`. `gravityOn(b)`. Render: player → `sndFall(); render()`. AI → `renderAI(); renderMobAI()`. `await delay(180)`. Loop.

### `gravityOn(b)`
Per column, pack non-null gems downward, fill top nulls with random gems. Data only — no DOM access.

---

## Step 21 — End game

### `showEndModal()`
Write scores to end modal. Determine win/lose/tie, increment `wins`, call the appropriate sound. Set `#end-title` class (`modal-title win|lose|tie`). Add `show` to `#end-modal`. Call `updateLeaderboard()`.

---

## Step 22 — Leaderboard

### `updateLeaderboard()`

Updates **both** the desktop leaderboard and the mobile leaderboard strip from the same sorted `entries` array.

```js
function updateLeaderboard() {
  const entries = [
    { name: 'You', wins: wins.you, isYou: true },
    { name: 'AI',  wins: wins.ai,  isYou: false }
  ];
  entries.sort((a, b) => b.wins - a.wins);

  // desktop
  for (let i = 0; i < 2; i++) {
    const e = entries[i];
    const av  = document.getElementById(`lb-av-${i+1}`);
    const nm  = document.getElementById(`lb-name-${i+1}`);
    const wn  = document.getElementById(`lb-wins-${i+1}`);
    const row = document.getElementById(`lb-row-${i+1}`);
    av.textContent = e.isYou ? 'YOU' : 'AI';
    av.className   = 'lb-avatar ' + (e.isYou ? 'you-av' : 'ai-av');
    wn.textContent = e.wins;
    wn.className   = 'lb-wins ' + (e.isYou ? 'you-wins' : 'ai-wins');
    const hasLead  = entries[0].wins > 0 && i === 0;
    row.classList.toggle('leader', hasLead);
    nm.innerHTML   = e.name + (hasLead ? ' <span class="lb-crown">👑</span>' : '');
  }

  // mobile strip
  for (let i = 0; i < 2; i++) {
    const e  = entries[i];
    const en = document.getElementById(`mob-lb-e${i+1}`);
    const nm = document.getElementById(`mob-lb-n${i+1}`);
    const wn = document.getElementById(`mob-lb-w${i+1}`);
    if (!en) continue;
    const hasLead = entries[0].wins > 0 && i === 0;
    en.className   = 'mob-lb-entry' + (hasLead ? ' mob-leader' : '');
    nm.textContent = e.name + (hasLead ? ' 👑' : '');
    wn.textContent = e.wins;
    wn.className   = 'mob-lb-wins ' + (e.isYou ? 'you-w' : 'ai-w');
  }
}
```

Call once at script end (initial display at 0–0) and once inside `showEndModal()`.

---

## Step 23 — UI helpers

### `updateScores()`
Updates all four score elements — desktop side-panels and mobile bar:
```js
function updateScores() {
  document.getElementById('score-val').textContent = score;
  document.getElementById('ai-score-val').textContent = aiScore;
  const my = document.getElementById('mob-you-score');
  const ma = document.getElementById('mob-ai-score');
  if (my) my.textContent = score;
  if (ma) ma.textContent = aiScore;
}
```

### `setMsg(text, cls = '')`
```js
const el = document.getElementById('msg');
el.textContent = text; el.className = cls;
```

### `delay(ms)`
```js
function delay(ms) { return new Promise(r => setTimeout(r, ms)); }
```

---

## Step 24 — Initialization

Last line of the script:
```js
updateLeaderboard();
```

---

## Step 25 — Verification checklist

- [ ] Single HTML file, no external JS
- [ ] Google Fonts: `DM Mono` (400, 500) and `DM Sans` (400, 500, 600)
- [ ] `<meta name="viewport" content="width=device-width, initial-scale=1.0">` present
- [ ] `--cell-size: clamp(28px, 10.5vw, 44px)` and `--gap: 2px` on `:root`
- [ ] `touch-action: manipulation` on `body`
- [ ] `-webkit-tap-highlight-color: transparent` on `.cell`, `.action-btn`, `.modal-primary-btn`
- [ ] `GAME_TIME = 30`
- [ ] `GEMS` has exactly 6 emoji: 🔴🟠🟡🟢🔵🟣
- [ ] Start modal has class `show` by default; end modal does not
- [ ] Three mobile-only elements hidden by default outside the media query: `#mobile-bar`, `#mob-lb`, `#mob-ai-wrap`
- [ ] `@media (max-width: 600px)` hides: `#tagline`, `#timer-bar`, `#leaderboard`, `.panel-label`, `.score-pill`, `#arena > .side-panel:first-child`
- [ ] `@media` shows: `#mobile-bar`, `#mob-lb`, `#mob-ai-wrap`
- [ ] `#mob-ai-grid` uses `clamp(18px, calc((100vw - 40px) / 8), 28px)` for both `grid-template-columns` and `grid-template-rows`
- [ ] `#mob-ai-grid .cell` overrides `width`, `height`, `font-size`, `border-radius`, `cursor: default`
- [ ] `#mob-ai-grid .cell:hover` disables transform and border — it is display-only
- [ ] `renderMobAI()` is called alongside every `renderAI()` call: in `startGame()`, in `aiTakeTurn()` after swap, and in `resolveMatchesOn()` for `'ai'`
- [ ] `getCell` scopes to `#grid`; `getAICell` scopes to `#ai-grid` — `#mob-ai-grid` is never queried by JS
- [ ] `updateTimerUI()` syncs both desktop (`#timer-fill`, `#timer-val`) and mobile (`#mob-timer-fill`, `#mob-timer-val`)
- [ ] `updateScores()` syncs all four elements: `#score-val`, `#ai-score-val`, `#mob-you-score`, `#mob-ai-score`
- [ ] `updateLeaderboard()` updates both desktop rows (`#lb-*`) and mobile strip entries (`#mob-lb-*`)
- [ ] `findMatchesOn(b)` is board-agnostic; uses flat-index `Set` deduplication
- [ ] `resolveMatchesOn(b, who)` scores the correct variable; sounds only for `'player'`
- [ ] `gravityOn(b)` is data-only; render always called after
- [ ] `sndStartJingle()` is the first call in `startGame()`
- [ ] `comboCount` resets before and after each player swap; increments per cascade wave
- [ ] `wins` persists across rounds
- [ ] `updateLeaderboard()` called at script end and inside `showEndModal()`
- [ ] `busy` and `gameActive` both enforced in `onCellClick()`

DO NOT START A LOCAL HTTP SERVER TO TEST THIS. ASSUME THAT IF ALL OF STEP 25 CHECKS PASSES, YOU ARE FREE TO CONSIDER THE TASK FINISHED.
