# Agent Prompt: Build a Match-3 Game (Exact Specification)

You are to produce a single self-contained HTML file implementing a fully functional two-player Match-3 puzzle game — a human player vs an AI opponent — with a countdown timer, start/end modals, a persistent leaderboard, a Web Audio API sound engine, and full mobile responsiveness. Follow every instruction below exactly. Do not add features, do not change the visual design, do not introduce external libraries beyond the fonts specified.

---

## Step 1 — File structure

One file: `match3.html`. Contents in order:
1. Standard HTML5 `<!DOCTYPE html>` boilerplate.
2. `<link>` loading two Google Fonts: `DM Mono` (weights 400, 500) and `DM Sans` (weights 400, 500, 600).
3. A single `<style>` block — ALL CSS, including the mobile media query at the bottom.
4. HTML markup directly inside `<body>`: start modal, end modal, then `<div id="app">`.
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

`--cell-size` uses `clamp` so cells shrink proportionally on narrow screens without any JavaScript.

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

`touch-action: manipulation` disables the 300ms tap delay on mobile. `align-items: flex-start` prevents the layout from vertically centering on short phones.

`body::before` decorative grid:
- `position: fixed; inset: 0; pointer-events: none`
- Two overlapping `linear-gradient` lines at `rgba(255,255,255,0.02)`, horizontal and vertical, `background-size: 40px 40px`

---

## Step 4 — HTML layout

### 4a — Start modal

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

### 4b — End modal

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

  <!-- mobile-only bar: your score | shrinking timer | AI score -->
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

  <div id="arena">

    <!-- AI board (left — hidden on mobile) -->
    <div class="side-panel">
      <div class="panel-label ai-label">AI</div>
      <div class="score-pill ai-score" id="ai-score-val">0</div>
      <div class="board-wrap ai-board">
        <div class="grid" id="ai-grid"></div>
      </div>
    </div>

    <!-- leaderboard (centre — hidden on mobile) -->
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

  </div>

  <div id="bottom">
    <div id="msg"></div>
    <button class="action-btn" onclick="restartGame()">New Game</button>
  </div>

</div>
```

Key notes:
- `#mobile-bar` is hidden by default via `#mobile-bar { display: none; }` in the base CSS, shown only inside the `@media` block.
- Both grids (`#grid` and `#ai-grid`) are empty — JavaScript fills them.
- The start modal carries class `show` by default; the end modal does not.

---

## Step 5 — CSS: layout containers

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

`font-size: clamp(13px, 4.5vw, 20px)` scales emoji with the viewport so they always fit inside their cell.

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

## Step 10 — CSS: leaderboard

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

**`#msg.bad`**: `color: #f87171`

**`#msg.good`**: `color: #6ee7b7`

**`.action-btn`**: DM Mono, 11px, weight 500, `letter-spacing: 0.1em`, `text-transform: uppercase`, `color: var(--accent)`, `background: transparent`, `border: 1px solid rgba(167,139,250,0.35)`, `border-radius: 8px`, `padding: 8px 16px`, `cursor: pointer`, `transition: background 0.15s, border-color 0.15s`, `white-space: nowrap`, `-webkit-tap-highlight-color: transparent`

**`.action-btn:hover`**: `background: rgba(167,139,250,0.1); border-color: var(--accent)`

**`.action-btn:active`**: `transform: scale(0.97)`

---

## Step 12 — CSS: modals

**`.modal-bg`**: `display: none; position: fixed; inset: 0; background: rgba(10,10,15,0.88); backdrop-filter: blur(6px); z-index: 100; align-items: center; justify-content: center; padding: 1rem`

**`.modal-bg.show`**: `display: flex`

**`.modal`**: `background: var(--surface); border: 1px solid var(--border-strong); border-radius: 20px; padding: 28px 24px; display: flex; flex-direction: column; align-items: center; gap: 12px; width: 100%; max-width: 340px; text-align: center`

**`.modal-eyebrow`**: DM Mono, 10px, `letter-spacing: 0.2em`, `text-transform: uppercase`, `color: var(--text-muted)`

**`.modal-title`**: DM Mono, 26px, weight 500, `color: var(--text)`, `line-height: 1.1`

**`.modal-title.win`**: `color: var(--accent)`

**`.modal-title.lose`**: `color: #f87171`

**`.modal-title.tie`**: `color: var(--ai-accent)`

**`.modal-sub`**: 13px, `color: var(--text-muted)`, `line-height: 1.6`, `margin-bottom: 2px`

**`.modal-scores`**: `display: flex; gap: 2px; background: var(--surface2); border: 1px solid var(--border); border-radius: 12px; overflow: hidden; width: 100%`

**`.modal-stat`**: `flex: 1; display: flex; flex-direction: column; align-items: center; padding: 10px 0; gap: 3px`

**`.modal-stat + .modal-stat`**: `border-left: 1px solid var(--border)`

**`.modal-stat-label`**: DM Mono, 10px, `letter-spacing: 0.1em`, `text-transform: uppercase`, `color: var(--text-muted)`

**`.modal-stat-val`**: DM Mono, 22px, weight 500

**`.modal-stat-val.you`**: `color: var(--accent)`

**`.modal-stat-val.ai`**: `color: var(--ai-accent)`

**`.modal-primary-btn`**: DM Mono, 12px, weight 500, `letter-spacing: 0.12em`, `text-transform: uppercase`, `color: var(--bg)`, `background: var(--accent)`, `border: none`, `border-radius: 10px`, `padding: 14px 32px`, `cursor: pointer`, `transition: opacity 0.15s`, `width: 100%`, `margin-top: 4px`, `-webkit-tap-highlight-color: transparent`

**`.modal-primary-btn:hover`**: `opacity: 0.85`

**`.start-gems`**: `display: flex; gap: 6px; font-size: 20px; margin-bottom: 2px`

**`.rules-list`**: 12px, `color: var(--text-muted)`, `line-height: 2`, `list-style: none`, `text-align: left`, `width: 100%`. Each `li::before { content: '·  '; }`

---

## Step 13 — CSS: mobile responsive block

The entire mobile layout is handled by a single `@media` block. Place this at the very end of the `<style>` block.

First, outside the media query, hide the mobile bar by default:
```css
#mobile-bar { display: none; }
```

Then the media query:

```css
@media (max-width: 600px) {
  body { padding: 0.75rem 0.5rem 1.5rem; align-items: flex-start; }
  #app { gap: 10px; }

  /* hide desktop-only elements */
  #tagline        { display: none; }
  #timer-bar      { display: none; }
  #arena > .side-panel:first-child { display: none; }
  #leaderboard    { display: none; }
  .panel-label, .score-pill { display: none; }

  /* player board fills full width */
  #arena      { width: 100%; justify-content: center; }
  .side-panel { width: 100%; }
  .board-wrap { width: 100%; }
  .grid       { margin: 0 auto; }

  /* mobile info bar */
  #mobile-bar {
    display: flex;
    align-items: center;
    width: 100%;
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 8px 14px;
    gap: 10px;
  }

  .mob-stat {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1px;
    min-width: 44px;
  }

  .mob-label {
    font-family: 'DM Mono', monospace;
    font-size: 9px;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    color: var(--text-muted);
  }

  .mob-val {
    font-family: 'DM Mono', monospace;
    font-size: 20px;
    font-weight: 500;
    line-height: 1;
  }

  .mob-val.you-col { color: var(--accent); }
  .mob-val.ai-col  { color: var(--ai-accent); }

  .mob-timer {
    flex: 1;
    display: flex;
    align-items: center;
    gap: 7px;
  }

  #mob-timer-track {
    flex: 1;
    height: 5px;
    background: var(--surface2);
    border-radius: 99px;
    overflow: hidden;
    border: 1px solid var(--border);
  }

  #mob-timer-fill {
    height: 100%;
    width: 100%;
    background: var(--accent);
    border-radius: 99px;
    transition: width 1s linear, background 0.4s;
  }

  #mob-timer-fill.low { background: #f87171; }

  #mob-timer-val {
    font-family: 'DM Mono', monospace;
    font-size: 13px;
    font-weight: 500;
    color: var(--text);
    min-width: 2ch;
    text-align: right;
  }

  #bottom { padding: 0 2px; }
}
```

**What this achieves on mobile (≤600px):**
- The tagline, desktop timer bar, AI board side-panel, leaderboard, and all `.panel-label`/`.score-pill` elements are hidden.
- The player board stretches to fill the full screen width. Cell size shrinks automatically via the `clamp` variable already set on `:root`.
- `#mobile-bar` appears at the top of the game area showing: your score (left) | shrinking timer bar + countdown (centre) | AI score (right).
- Desktop (>600px) is completely unaffected — no overrides needed.

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

- `GAME_TIME = 30`: one round lasts 30 seconds.
- `gameActive`: `false` blocks all player input.
- `busy`: blocks player clicks during async match resolution.
- `aiBusy`: prevents the AI overlapping its own turns.
- `wins`: never resets between rounds — persists for the full page session.
- `comboCount`: incremented each cascade wave during one player swap; reset before and after each swap.

---

## Step 15 — Audio engine (Web Audio API, zero external files)

```js
const AC = new (window.AudioContext || window.webkitAudioContext)();
function resumeAC() { if (AC.state === 'suspended') AC.resume(); }
```

Call `resumeAC()` at the top of every sound function — browsers suspend the context until a user gesture.

### `synth({ type, freq, freqEnd, startTime, duration, gainPeak, attack, decay, sustain, release })`

1. Create `OscillatorNode` + `GainNode`; connect `osc → gain → AC.destination`.
2. `osc.type = type`; schedule `freq` at `startTime`. If `freqEnd` defined: `exponentialRampToValueAtTime(freqEnd, startTime + duration)`.
3. `rel = release ?? duration * 0.4`.
4. Gain envelope: `0` → `gainPeak` (attack) → `gainPeak * sustain` (decay) → hold → `0` (release).
5. `osc.start(startTime)`; `osc.stop(startTime + duration + 0.01)`.

### `noiseBlip(startTime, duration, gainPeak)`

Fill a mono `AudioBuffer` with `Math.random() * 2 - 1`. Route through a `BiquadFilterNode` (`bandpass`, 180 Hz, Q 1.2). Apply a `GainNode` fading from `gainPeak` to `0`.

### Sound event table

| Function | Trigger | Description |
|---|---|---|
| `sndStartJingle()` | First line of `startGame()` | Loud 2-second multi-layer jingle — full spec below |
| `sndSelect()` | Player first-clicks a gem or re-selects non-adjacent | Descending sine: 880→740 Hz, 0.08s, gainPeak 0.07 |
| `sndSwap()` | Player confirms an adjacent swap | Two detuned swept sines: 300→520 and 310→530 Hz, 0.12s |
| `sndNoMatch()` | Swap reversed, no matches | Triangle thud 120→60 Hz + noiseBlip gainPeak 0.06 |
| `sndMatch(count)` | Each cascade wave, player only | Major chord chime, root climbs with comboCount; sparkle if count ≥ 5 |
| `sndFall()` | After player gravity resolves | Triangle thump 200→140 Hz, 0.1s, gainPeak 0.06 |
| `sndTick()` | Each second when timeLeft ≤ 8 and > 0 | Square blip 660 Hz, 0.06s, gainPeak 0.05 |
| `sndWin()` | Player wins | Ascending arpeggio C5 E5 G5 C6, staggered 0.1s each |
| `sndLose()` | AI wins | Descending minor A4 G4 F4 D4, staggered 0.11s each |
| `sndTie()` | Scores equal | Two sine pings at 440 Hz, 0.25s apart |

### `sndStartJingle()` spec

Base gain `G = 0.55`. Five simultaneous layers starting from `t = AC.currentTime`:

**Layer 1 — Bass hits** (triangle, 80→55 Hz, 0.35s, `gainPeak G*0.9`): fire at t+0, t+0.5, t+1.0, t+1.5.

**Layer 2 — Arpeggio** (sawtooth, 0.22s, `gainPeak G*0.35`): 8 notes spaced 0.18s — C4 E4 G4 C5 E5 G5 C6 E6 (261.63, 329.63, 392, 523.25, 659.26, 783.99, 1046.5, 1318.5 Hz).

**Layer 3 — Chord stabs** (square, 0.38s, `gainPeak G*0.22`): C major at t+0, F major at t+0.5, G major+octave at t+1.0.

**Layer 4 — Landing chord** (sine + detuned copy at `freq*1.004`, `gainPeak G*0.45` and `G*0.18`): C5 E5 G5 C6 at t+1.5, staggered 0.03s, duration 0.55s.

**Layer 5 — Snare hits** (`noiseBlip`, 0.08s, gainPeak 0.35): t+0.25, t+0.75, t+1.25, t+1.75.

### `sndMatch(count)` pitch escalation

```js
const base = 523.25 * Math.pow(2, Math.min(comboCount, 5) / 12);
```
Play root + major third + fifth as three sine tones staggered 0.02s. If `count >= 5`, add octave sparkle at `base*2`.

---

## Step 16 — Game lifecycle

### `startGame()`
1. `sndStartJingle()` — must be first (unlocks AudioContext via user gesture).
2. Remove class `show` from both modals.
3. Reset: `score=0; aiScore=0; selected=null; busy=false; aiBusy=false; gameActive=true`.
4. `board = freshBoard(); aiBoard = freshBoard()`.
5. `render(); renderAI(); updateScores(); startTimer(); scheduleAI(); setMsg('')`.

### `restartGame()`
`clearInterval(timerInterval); clearTimeout(aiTimer); startGame();`

### `freshBoard()`
Generate a random 8×8 board, re-generating until `findMatchesOn(b).length === 0`. Returns the board.

---

## Step 17 — Timer

### `startTimer()`
Set `timeLeft = GAME_TIME`, call `updateTimerUI()`, clear any existing interval, start `setInterval` at 1000ms:
- `timeLeft--; updateTimerUI()`.
- If `timeLeft <= 8 && timeLeft > 0`: `sndTick()`.
- If `timeLeft <= 0`: clear interval, clear `aiTimer`, `gameActive=false; busy=true`, then `setTimeout(showEndModal, 400)`.

### `updateTimerUI()`

Updates **both** the desktop timer bar and the mobile timer bar simultaneously:

```js
function updateTimerUI() {
  const pct = (timeLeft / GAME_TIME) * 100;

  // desktop
  document.getElementById('timer-val').textContent = timeLeft;
  const fill = document.getElementById('timer-fill');
  fill.style.width = pct + '%';
  fill.classList.toggle('low', timeLeft <= 8);

  // mobile
  const mf = document.getElementById('mob-timer-fill');
  const mv = document.getElementById('mob-timer-val');
  if (mf) { mf.style.width = pct + '%'; mf.classList.toggle('low', timeLeft <= 8); }
  if (mv) mv.textContent = timeLeft;
}
```

The `if` guards are defensive — both elements always exist in the markup, but this protects against order-of-execution edge cases.

---

## Step 18 — Player board

### `render()`
Clear `#grid`, rebuild all cells from `board` row-major. Each cell: `className='cell'`, `textContent=board[r][c]`, `dataset.r`, `dataset.c`, `onclick`. Add `selected` class if it matches `selected`.

### `getCell(r, c)`
```js
return document.querySelector(`#grid [data-r="${r}"][data-c="${c}"]`);
```
The `#grid` scope prefix is mandatory — AI cells share the same `data-r`/`data-c` attributes.

### `onCellClick(r, c)`
1. If `!gameActive || busy`: return.
2. If `!selected`: `sndSelect()`, set `selected`, `render()`, return.
3. Destructure `[sr, sc] = selected`.
4. Same cell: deselect, `render()`, return.
5. Manhattan distance ≠ 1: `sndSelect()`, re-select new cell, `render()`, return.
6. `selected = null; doSwap(sr, sc, r, c)`.

### `doSwap(r1, c1, r2, c2)` (async)
1. `busy=true; sndSwap()`.
2. Swap cells. `render()`.
3. No matches: reverse, `render()`, `sndNoMatch()`, `setMsg('no match','bad')`, clear after 900ms, `busy=false`, return.
4. `comboCount=0; await resolveMatchesOn(board,'player'); comboCount=0; busy=false`.

---

## Step 19 — AI board

### `renderAI()`
Same structure as `render()` targeting `#ai-grid`. No `onclick` handlers.

### `getAICell(r, c)`
```js
return document.querySelector(`#ai-grid [data-r="${r}"][data-c="${c}"]`);
```

### `scheduleAI()`
If `!gameActive` return. `aiTimer = setTimeout(() => aiTakeTurn(), 3000 + Math.random() * 3000)`.

### `aiTakeTurn()` (async)
1. Guard: `if (!gameActive || aiBusy) return`. `aiBusy=true`.
2. `move = findBestAIMove()`. If null: `aiBusy=false; scheduleAI(); return`.
3. Flash `ai-hint-a` on both cells (300ms), then `ai-hint-b` (250ms).
4. Swap cells. `renderAI()`. `await resolveMatchesOn(aiBoard, 'ai')`.
5. `aiBusy=false`. If `gameActive`: `scheduleAI()`.

### `findBestAIMove()`
Test all right/down adjacent pairs by temporarily swapping and calling `findMatchesOn(aiBoard)`. Collect pairs with ≥1 match, sort by match count descending, return the top pair or `null`.

---

## Step 20 — Shared match logic

### `findMatchesOn(b)`
Board-agnostic. Uses `Set<number>` of flat indices (`r * COLS + c`) to deduplicate intersections. Horizontal then vertical runs of 3+. Returns `[[r,c],...]`.

### `resolveMatchesOn(b, who)` (async)
Loop while matches exist:
1. Score: player `score += matches.length * 10; updateScores(); sndMatch(matches.length); comboCount++`. AI: `aiScore += matches.length * 10; updateScores()`.
2. Add `matched` class to each cell element. Set `b[r][c] = null`.
3. `await delay(290)`. `gravityOn(b)`.
4. Player: `sndFall(); render()`. AI: `renderAI()`.
5. `await delay(180)`. Loop.

### `gravityOn(b)`
Per column: write pointer starting at `ROWS-1`, scan downward packing non-null gems, fill top nulls with new random gems. Data only — no DOM access.

---

## Step 21 — End game

### `showEndModal()`
1. Write `score` and `aiScore` to `#end-you-score` and `#end-ai-score`.
2. Determine outcome, set title/sub/cls, increment `wins.you` or `wins.ai`, call `sndWin()`/`sndLose()`/`sndTie()`.
3. Set `#end-title` text and class (`modal-title win|lose|tie`). Set `#end-sub`.
4. Add `show` to `#end-modal`. Call `updateLeaderboard()`.

---

## Step 22 — Leaderboard

### `updateLeaderboard()`
1. Build two entries, sort descending by wins.
2. For each position: set avatar text (`'YOU'`/`'AI'`), avatar class, wins text, wins class.
3. If `entries[0].wins > 0`: add `leader` class to `#lb-row-1`, inject `👑` into name via `innerHTML`.

Call once at script end (initial display) and once inside `showEndModal()`.

---

## Step 23 — UI helpers

### `updateScores()`
Updates **all four** score elements — desktop side-panels and mobile bar:

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
el.textContent = text;
el.className = cls;
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

Populates the leaderboard at 0–0 before any game starts. The game waits for the user to click "Start Game".

---

## Step 25 — Verification checklist

- [ ] Single HTML file, no external JS
- [ ] Google Fonts: `DM Mono` (400, 500) and `DM Sans` (400, 500, 600)
- [ ] `<meta name="viewport" content="width=device-width, initial-scale=1.0">` present
- [ ] All CSS variables on `:root` including `--cell-size: clamp(28px, 10.5vw, 44px)` and `--gap: 2px`
- [ ] `touch-action: manipulation` on `body`
- [ ] `-webkit-tap-highlight-color: transparent` on `.cell`, `.action-btn`, `.modal-primary-btn`
- [ ] `GAME_TIME = 30`, `COLS = 8`, `ROWS = 8`
- [ ] `GEMS` has exactly 6 emoji: 🔴🟠🟡🟢🔵🟣
- [ ] Start modal has class `show` by default; end modal does not
- [ ] `#mobile-bar { display: none; }` declared outside the media query (hidden by default)
- [ ] `@media (max-width: 600px)` block hides: `#tagline`, `#timer-bar`, first `.side-panel` in `#arena`, `#leaderboard`, `.panel-label`, `.score-pill`
- [ ] Mobile block shows `#mobile-bar` with `display: flex`
- [ ] `#arena > .side-panel:first-child` targets the AI panel specifically
- [ ] `--cell-size: clamp(28px, 10.5vw, 44px)` makes the board self-sizing — no JS needed for mobile grid width
- [ ] `font-size: clamp(13px, 4.5vw, 20px)` on `.cell` scales emoji with the grid
- [ ] `updateTimerUI()` writes to both `#timer-fill`/`#timer-val` AND `#mob-timer-fill`/`#mob-timer-val`
- [ ] `updateScores()` writes to both `#score-val`/`#ai-score-val` AND `#mob-you-score`/`#mob-ai-score`
- [ ] `getCell` scopes to `#grid`, `getAICell` scopes to `#ai-grid` — no cross-contamination
- [ ] `findMatchesOn(b)` is board-agnostic with flat-index `Set` deduplication
- [ ] `resolveMatchesOn(b, who)` scores the correct variable; sounds only for `'player'`
- [ ] `gravityOn(b)` is data-only; render called after
- [ ] AI two-phase flash: `ai-hint-a` (300ms) then `ai-hint-b` (250ms)
- [ ] `sndStartJingle()` is the **first** call in `startGame()` — AudioContext unlock requires the user gesture
- [ ] `comboCount` resets before and after each player swap; increments per cascade wave
- [ ] `wins` persists across rounds
- [ ] `updateLeaderboard()` called at script end and inside `showEndModal()`
- [ ] `busy` and `gameActive` flags both enforced in `onCellClick()`