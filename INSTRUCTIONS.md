# Agent Prompt: Build a Match-3 Game (Exact Specification)

You are to produce a single self-contained HTML file that implements a fully functional two-player Match-3 puzzle game — a human player vs an AI opponent — with a countdown timer, start/end modals, a persistent leaderboard, and a Web Audio API sound engine. Follow every instruction below exactly. Do not add features, do not change the visual design, do not introduce external libraries beyond the fonts specified.

---

## Step 1 — File structure

Create one file: `match3.html`. It must contain, in order:
1. A standard HTML5 `<!DOCTYPE html>` boilerplate with `<head>` and `<body>`.
2. A `<link>` tag loading two Google Fonts: `DM Mono` (weights 400, 500) and `DM Sans` (weights 400, 500, 600).
3. A single `<style>` block containing ALL CSS.
4. All HTML markup directly inside `<body>` (two modal divs, then `<div id="app">`).
5. A single `<script>` block at the end of `<body>` containing ALL JavaScript.

No external JS libraries. No separate CSS files.

---

## Step 2 — CSS variables and base reset

At the top of the `<style>` block, write a universal reset:
```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
```

Then declare these CSS custom properties on `:root`:
```
--bg: #0f0f13
--surface: #1a1a22
--surface2: #22222e
--border: rgba(255,255,255,0.08)
--border-strong: rgba(255,255,255,0.18)
--text: #f0eff4
--text-muted: #7c7a8e
--accent: #a78bfa              (purple — player colour)
--accent-glow: rgba(167,139,250,0.25)
--ai-accent: #fb923c           (orange — AI colour)
--ai-glow: rgba(251,146,60,0.25)
--cell-size: 44px
--gap: 3px
```

---

## Step 3 — Body and background

Style `body` as:
- `background: var(--bg)`
- `color: var(--text)`
- `font-family: 'DM Sans', sans-serif`
- `min-height: 100vh`
- `display: flex; align-items: center; justify-content: center`
- `padding: 1.5rem 1rem`

Add a `body::before` pseudo-element as a decorative grid overlay:
- `position: fixed; inset: 0; pointer-events: none`
- `background-image`: two layered `linear-gradient` calls that draw 1px lines at `rgba(255,255,255,0.02)` every 40px — one horizontal, one vertical (`90deg`).
- `background-size: 40px 40px`

---

## Step 4 — HTML layout

### 4a — Start modal (shown on page load)

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
      <li>AI makes a move every 3-6 seconds</li>
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

  <div id="timer-bar">
    <span id="timer-label">Time</span>
    <div id="timer-track"><div id="timer-fill"></div></div>
    <span id="timer-val">30</span>
  </div>

  <div id="arena">

    <!-- AI board (left) -->
    <div class="side-panel">
      <div class="panel-label ai-label">AI</div>
      <div class="score-pill ai-score" id="ai-score-val">0</div>
      <div class="board-wrap ai-board">
        <div class="grid" id="ai-grid"></div>
      </div>
    </div>

    <!-- Leaderboard (centre) -->
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

    <!-- Player board (right) -->
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

Important notes:
- Both `#grid` and `#ai-grid` are empty — JavaScript fills them at runtime.
- The start modal has class `show` by default; the end modal does not.
- The leaderboard sits between the two boards inside `#arena`.

---

## Step 5 — CSS: layout

**`#app`**: `display: flex; flex-direction: column; align-items: center; gap: 16px; position: relative; z-index: 1`

**`header`**: `display: flex; flex-direction: column; align-items: center; gap: 4px`

**`h1`**: DM Mono, 13px, weight 500, `letter-spacing: 0.18em`, `text-transform: uppercase`, `color: var(--accent)`

**`#tagline`**: 12px, `color: var(--text-muted)`, `letter-spacing: 0.04em`

**`#arena`**: `display: flex; align-items: flex-start; gap: 16px`

**`.side-panel`**: `display: flex; flex-direction: column; align-items: center; gap: 10px`

---

## Step 6 — CSS: timer bar

**`#timer-bar`**: `display: flex; align-items: center; gap: 10px; font-family: DM Mono`

**`#timer-label`**: 10px, `letter-spacing: 0.12em`, `text-transform: uppercase`, `color: var(--text-muted)`

**`#timer-track`**: `width: 200px; height: 6px; background: var(--surface2); border-radius: 99px; overflow: hidden; border: 1px solid var(--border)`

**`#timer-fill`**: `height: 100%; width: 100%; background: var(--accent); border-radius: 99px; transition: width 1s linear, background 0.4s`

**`#timer-fill.low`**: `background: #f87171` (turns red when timeLeft <= 8)

**`#timer-val`**: DM Mono, 13px, weight 500, `color: var(--text)`, `min-width: 3ch; text-align: right`

---

## Step 7 — CSS: side panel labels and score pills

**`.panel-label`**: DM Mono, 10px, `letter-spacing: 0.14em`, `text-transform: uppercase`, `padding: 4px 12px`, `border-radius: 99px`, `border: 1px solid var(--border)`

**`.panel-label.ai-label`**: `color: var(--ai-accent); border-color: rgba(251,146,60,0.3)`

**`.panel-label.you-label`**: `color: var(--accent); border-color: rgba(167,139,250,0.3)`

**`.score-pill`**: DM Mono, 20px, weight 500, `min-width: 5ch; text-align: center`

**`.score-pill.ai-score`**: `color: var(--ai-accent)`

**`.score-pill.you-score`**: `color: var(--accent)`

---

## Step 8 — CSS: board wraps and grids

**`.board-wrap`**: `background: var(--surface); border: 1px solid var(--border); border-radius: 16px; padding: 12px; position: relative`

**`.board-wrap.ai-board`**: `border-color: rgba(251,146,60,0.2)`

**`.board-wrap.you-board`**: `border-color: rgba(167,139,250,0.2)`

**`.grid`**: CSS Grid — `grid-template-columns: repeat(8, var(--cell-size)); grid-template-rows: repeat(8, var(--cell-size)); gap: var(--gap)`

---

## Step 9 — CSS: cells

**`.cell`** (base state):
```
width: var(--cell-size); height: var(--cell-size)
border-radius: 8px
display: flex; align-items: center; justify-content: center
font-size: 20px
cursor: pointer
border: 1.5px solid transparent
background: var(--surface2)
transition: transform 0.12s ease, border-color 0.12s ease, background 0.12s ease
user-select: none
```

**`.cell:hover`**: `transform: scale(1.07); background: #2a2a38; border-color: var(--border-strong)`

**`.cell.selected`**: `border-color: var(--accent); background: rgba(167,139,250,0.12); transform: scale(1.1); box-shadow: 0 0 0 3px var(--accent-glow)`

**`.cell.ai-hint-a`**: `border-color: var(--ai-accent); background: rgba(251,146,60,0.1)` (first flash phase)

**`.cell.ai-hint-b`**: `border-color: var(--ai-accent); background: rgba(251,146,60,0.2); transform: scale(1.08)` (second flash phase)

**`.cell.matched`**: animation `pop 0.28s ease forwards`

```css
@keyframes pop {
  0%   { transform: scale(1); opacity: 1; }
  40%  { transform: scale(1.25); opacity: 0.6; }
  100% { transform: scale(0); opacity: 0; }
}
```

---

## Step 10 — CSS: leaderboard

**`#leaderboard`**: `background: var(--surface); border: 1px solid var(--border); border-radius: 16px; padding: 16px 14px; display: flex; flex-direction: column; gap: 12px; min-width: 140px; align-self: stretch`

**`#lb-title`**: DM Mono, 10px, `letter-spacing: 0.14em`, `text-transform: uppercase`, `color: var(--text-muted)`, `text-align: center`, `padding-bottom: 10px`, `border-bottom: 1px solid var(--border)`

**`.lb-row`**: `display: flex; align-items: center; gap: 8px; padding: 8px 10px; border-radius: 10px; background: var(--surface2); border: 1px solid var(--border)`

**`.lb-row.leader`**: `border-color: rgba(255,215,0,0.25); background: rgba(255,215,0,0.04)` (gold highlight for the leading player)

**`.lb-rank`**: DM Mono, 11px, `color: var(--text-muted)`, `min-width: 14px`

**`.lb-avatar`**: `width: 28px; height: 28px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 11px; font-weight: 500; flex-shrink: 0`

**`.lb-avatar.ai-av`**: `background: rgba(251,146,60,0.15); color: var(--ai-accent)`

**`.lb-avatar.you-av`**: `background: rgba(167,139,250,0.15); color: var(--accent)`

**`.lb-name`**: 13px, weight 500, `flex: 1`

**`.lb-wins`**: DM Mono, 18px, weight 500

**`.lb-wins.ai-wins`**: `color: var(--ai-accent)`

**`.lb-wins.you-wins`**: `color: var(--accent)`

**`.lb-divider`**: DM Mono, 10px, `color: var(--text-muted)`, `text-align: center`, `letter-spacing: 0.08em`

---

## Step 11 — CSS: bottom bar

**`#bottom`**: `display: flex; align-items: center; gap: 12px; width: 100%; justify-content: space-between`

**`#msg`**: DM Mono, 12px, `color: var(--text-muted)`, `letter-spacing: 0.05em`, `min-height: 16px`, `flex: 1`

**`#msg.bad`**: `color: #f87171`

**`#msg.good`**: `color: #6ee7b7`

**`.action-btn`**: DM Mono, 11px, weight 500, `letter-spacing: 0.1em`, `text-transform: uppercase`, `color: var(--accent)`, `background: transparent`, `border: 1px solid rgba(167,139,250,0.35)`, `border-radius: 8px`, `padding: 7px 16px`, `cursor: pointer`, `transition: background 0.15s, border-color 0.15s`, `white-space: nowrap`

**`.action-btn:hover`**: `background: rgba(167,139,250,0.1); border-color: var(--accent)`

**`.action-btn:active`**: `transform: scale(0.97)`

---

## Step 12 — CSS: modals

**`.modal-bg`** (shared wrapper):
```
display: none
position: fixed; inset: 0
background: rgba(10,10,15,0.85)
backdrop-filter: blur(6px)
z-index: 100
align-items: center; justify-content: center
```

**`.modal-bg.show`**: `display: flex`

**`.modal`** (inner card):
```
background: var(--surface)
border: 1px solid var(--border-strong)
border-radius: 20px
padding: 36px 40px
display: flex; flex-direction: column; align-items: center
gap: 14px
min-width: 300px; max-width: 360px
text-align: center
```

**`.modal-eyebrow`**: DM Mono, 10px, `letter-spacing: 0.2em`, `text-transform: uppercase`, `color: var(--text-muted)`

**`.modal-title`**: DM Mono, 28px, weight 500, `color: var(--text)`, `line-height: 1.1`

**`.modal-title.win`**: `color: var(--accent)`

**`.modal-title.lose`**: `color: #f87171`

**`.modal-title.tie`**: `color: var(--ai-accent)`

**`.modal-sub`**: 13px, `color: var(--text-muted)`, `line-height: 1.6`, `margin-bottom: 4px`

**`.modal-scores`**: `display: flex; gap: 2px; background: var(--surface2); border: 1px solid var(--border); border-radius: 12px; overflow: hidden; width: 100%`

**`.modal-stat`**: `flex: 1; display: flex; flex-direction: column; align-items: center; padding: 10px 0; gap: 3px`

**`.modal-stat + .modal-stat`**: `border-left: 1px solid var(--border)`

**`.modal-stat-label`**: DM Mono, 10px, `letter-spacing: 0.1em`, `text-transform: uppercase`, `color: var(--text-muted)`

**`.modal-stat-val`**: DM Mono, 22px, weight 500

**`.modal-stat-val.you`**: `color: var(--accent)`

**`.modal-stat-val.ai`**: `color: var(--ai-accent)`

**`.modal-primary-btn`**: DM Mono, 12px, weight 500, `letter-spacing: 0.12em`, `text-transform: uppercase`, `color: var(--bg)`, `background: var(--accent)`, `border: none`, `border-radius: 10px`, `padding: 12px 32px`, `cursor: pointer`, `transition: opacity 0.15s`, `width: 100%`, `margin-top: 4px`

**`.modal-primary-btn:hover`**: `opacity: 0.85`

**`.start-gems`**: `display: flex; gap: 6px; font-size: 22px; margin-bottom: 4px`

**`.rules-list`**: 12px, `color: var(--text-muted)`, `line-height: 2`, `list-style: none`. Each `li::before` uses `content: '·  '`.

---

## Step 13 — JavaScript: constants and state

```js
const GEMS = ['🔴','🟠','🟡','🟢','🔵','🟣'];
const COLS = 8, ROWS = 8, GAME_TIME = 30;

// player
let board = [], selected = null, score = 0, busy = false, gameActive = false;

// ai
let aiBoard = [], aiScore = 0, aiBusy = false, aiTimer = null;

// timer
let timeLeft = GAME_TIME, timerInterval = null;

// leaderboard — persists across rounds, resets only on full page reload
let wins = { you: 0, ai: 0 };

// combo counter for audio pitch escalation
let comboCount = 0;
```

- `GAME_TIME = 30`: the round lasts exactly 30 seconds.
- `gameActive`: when `false`, all player input is blocked.
- `busy`: blocks player clicks during async match resolution.
- `aiBusy`: prevents the AI from overlapping its own turns.
- `wins`: never resets between rounds.
- `comboCount`: incremented each cascade wave during a player swap; reset to 0 before and after each swap.

---

## Step 14 — Audio engine (Web Audio API, no external files)

Create a single shared AudioContext:

```js
const AC = new (window.AudioContext || window.webkitAudioContext)();
function resumeAC() { if (AC.state === 'suspended') AC.resume(); }
```

Call `resumeAC()` at the top of every sound function — browsers suspend the context until a user gesture unlocks it.

### `synth({ type, freq, freqEnd, startTime, duration, gainPeak, attack, decay, sustain, release })`

General-purpose oscillator with ADSR-style gain envelope. All sound functions build on this.

1. Create `OscillatorNode` + `GainNode`; connect `osc → gain → AC.destination`.
2. Set `osc.type = type`, schedule `osc.frequency = freq` at `startTime`.
3. If `freqEnd` is defined, `exponentialRampToValueAtTime(freqEnd, startTime + duration)`.
4. `rel = release ?? duration * 0.4`.
5. Gain envelope:
   - `0` at `startTime`
   - `gainPeak` at `startTime + attack`
   - `gainPeak * sustain` at `startTime + attack + decay`
   - Hold `gainPeak * sustain` until `startTime + duration - rel`
   - `0` at `startTime + duration`
6. `osc.start(startTime)`, `osc.stop(startTime + duration + 0.01)`.

### `noiseBlip(startTime, duration, gainPeak)`

Generates a short filtered noise burst (used for snare hits and thud effects).

1. Create a mono `AudioBuffer` of `AC.sampleRate * duration` samples; fill with `Math.random() * 2 - 1`.
2. Route through a `BiquadFilterNode` (`type: 'bandpass'`, `frequency: 180`, `Q: 1.2`).
3. Apply a `GainNode` that fades from `gainPeak` to `0` over `duration`.

### Sound functions — when and how to call each

| Function | Trigger | What it sounds like |
|---|---|---|
| `sndStartJingle()` | First call inside `startGame()` | Loud 2-second multi-layer jingle — see full spec below |
| `sndSelect()` | Player selects a gem (first click or re-select of non-adjacent) | Short descending sine tick: 880→740 Hz, 0.08s, gainPeak 0.07 |
| `sndSwap()` | Player confirms an adjacent swap (before match check) | Two detuned pitch-swept sines: 300→520 Hz and 310→530 Hz, 0.12s |
| `sndNoMatch()` | Swap reverses because no matches found | Low triangle thud 120→60 Hz + noiseBlip at gainPeak 0.06 |
| `sndMatch(count)` | Each cascade wave during player resolution | Major chord chime; root pitch rises with comboCount; sparkle if count >= 5 |
| `sndFall()` | After gravityOn() is called for the player board | Soft triangle thump: 200→140 Hz, 0.1s, gainPeak 0.06 |
| `sndTick()` | Each timer tick when `timeLeft <= 8 && timeLeft > 0` | Square-wave blip: 660 Hz, 0.06s, gainPeak 0.05 |
| `sndWin()` | Player wins the round (inside `showEndModal`) | 4-note ascending arpeggio: C5 E5 G5 C6, staggered 0.1s, gainPeak 0.13 |
| `sndLose()` | AI wins the round | 4-note descending minor motif: A4 G4 F4 D4, staggered 0.11s, gainPeak 0.11 |
| `sndTie()` | Scores are equal | Two identical sine pings at 440 Hz, 0.25s apart, gainPeak 0.1 |

### `sndStartJingle()` — full specification

Fires immediately when `startGame()` is called (the button click is the AudioContext unlock gesture). Total audible duration ~2 seconds. Base gain multiplier `G = 0.55` (loud). Five simultaneous layers:

**Layer 1 — Bass hits** (triangle, 80→55 Hz, duration 0.35s, gainPeak `G * 0.9`):
Fire at `t + 0`, `t + 0.5`, `t + 1.0`, `t + 1.5`.

**Layer 2 — Ascending arpeggio** (sawtooth, duration 0.22s, gainPeak `G * 0.35`):
8 notes starting 0.18s apart: C4, E4, G4, C5, E5, G5, C6, E6
(Hz: 261.63, 329.63, 392, 523.25, 659.26, 783.99, 1046.5, 1318.5)

**Layer 3 — Chord stabs** (square wave, duration 0.38s, gainPeak `G * 0.22`):
Three chords, each 0.5s apart:
- `t + 0.0`: C major — 261.63, 329.63, 392 Hz
- `t + 0.5`: F major — 349.23, 440, 523.25 Hz
- `t + 1.0`: G major with octave — 392, 493.88, 587.33, 783.99 Hz

**Layer 4 — Landing chord** (sine + detuned shimmer copy):
At `t + 1.5`, play C5 E5 G5 C6 (523.25, 659.26, 783.99, 1046.5 Hz) staggered 0.03s each, duration 0.55s, gainPeak `G * 0.45`. Each note also has a second oscillator at `freq * 1.004`, gainPeak `G * 0.18`, for chorus width.

**Layer 5 — Snare hits** (`noiseBlip`, duration 0.08s, gainPeak 0.35):
Fire at offbeats: `t + 0.25`, `t + 0.75`, `t + 1.25`, `t + 1.75`.

### `sndMatch(count)` — pitch escalation detail

```js
const semis = Math.min(comboCount, 5);
const base = 523.25 * Math.pow(2, semis / 12); // C5 + semis semitones
```

Play root, major third, perfect fifth above `base` as three sine tones staggered 0.02s apart, duration 0.22s, gainPeak 0.11. If `count >= 5`, add an extra sine at `base * 2` (one octave up) at gainPeak 0.07, startTime offset 0.05s.

---

## Step 15 — `startGame()`

Called by both the Start Game button and the Play Again button.

1. Call `sndStartJingle()` — must be first so the AudioContext is unlocked by the user gesture.
2. Remove class `show` from both `#start-modal` and `#end-modal`.
3. Reset: `score = 0; aiScore = 0; selected = null; busy = false; aiBusy = false; gameActive = true`.
4. `board = freshBoard(); aiBoard = freshBoard()`.
5. Call `render()`, `renderAI()`, `updateScores()`, `startTimer()`, `scheduleAI()`, `setMsg('')`.

### `restartGame()`

`clearInterval(timerInterval); clearTimeout(aiTimer); startGame();`

### `freshBoard()`

```js
let b;
do {
  b = Array.from({length: ROWS}, () =>
    Array.from({length: COLS}, () => GEMS[Math.floor(Math.random() * GEMS.length)]));
} while (findMatchesOn(b).length > 0);
return b;
```

Guarantees no pre-existing matches on the initial board.

---

## Step 16 — Timer

### `startTimer()`

1. `timeLeft = GAME_TIME`. Call `updateTimerUI()`. Clear any existing interval.
2. `setInterval` every 1000ms:
   - `timeLeft--; updateTimerUI()`.
   - If `timeLeft <= 8 && timeLeft > 0`: call `sndTick()`.
   - If `timeLeft <= 0`: clear interval, clear `aiTimer`, `gameActive = false`, `busy = true`, `setTimeout(showEndModal, 400)`.

### `updateTimerUI()`

```js
document.getElementById('timer-val').textContent = timeLeft;
const pct = (timeLeft / GAME_TIME) * 100;
const fill = document.getElementById('timer-fill');
fill.style.width = pct + '%';
fill.classList.toggle('low', timeLeft <= 8);
```

---

## Step 17 — Player board

### `render()`

Clears `#grid` and rebuilds from `board` in row-major order. For each `[r, c]`: create `<div class="cell">`, set `textContent`, `dataset.r`, `dataset.c`, `onclick`. Add class `selected` if it matches `selected`. Append to `#grid`.

### `getCell(r, c)`

```js
return document.querySelector(`#grid [data-r="${r}"][data-c="${c}"]`);
```

The `#grid` scope prefix is required — AI cells share the same `data-r`/`data-c` attribute names.

### `onCellClick(r, c)`

1. If `!gameActive || busy`: return.
2. If `!selected`: `sndSelect()`, `selected = [r, c]`, `render()`, return.
3. `[sr, sc] = selected`.
4. If same cell: `selected = null`, `render()`, return.
5. If Manhattan distance ≠ 1: `sndSelect()`, `selected = [r, c]`, `render()`, return.
6. `selected = null`. Call `doSwap(sr, sc, r, c)`.

### `doSwap(r1, c1, r2, c2)` (async)

1. `busy = true`. Call `sndSwap()`.
2. Swap `board[r1][c1]` and `board[r2][c2]`. Call `render()`.
3. If `findMatchesOn(board).length === 0`: reverse swap, `render()`, `sndNoMatch()`, `setMsg('no match', 'bad')`, clear message after 900ms, `busy = false`, return.
4. `comboCount = 0`.
5. `await resolveMatchesOn(board, 'player')`.
6. `comboCount = 0; busy = false`.

---

## Step 18 — AI board

### `renderAI()`

Same structure as `render()` but targets `#ai-grid`. No `onclick` handlers — cells are display-only.

### `getAICell(r, c)`

```js
return document.querySelector(`#ai-grid [data-r="${r}"][data-c="${c}"]`);
```

### `scheduleAI()`

If `!gameActive`, return. Otherwise: `aiTimer = setTimeout(() => aiTakeTurn(), 3000 + Math.random() * 3000)`.

### `aiTakeTurn()` (async)

1. If `!gameActive || aiBusy`: return. `aiBusy = true`.
2. `move = findBestAIMove()`. If null: `aiBusy = false; scheduleAI(); return`.
3. `[[r1,c1],[r2,c2]] = move`.
4. Get DOM refs `a = getAICell(r1,c1)`, `b2 = getAICell(r2,c2)`.
5. Add `ai-hint-a` to both. `await delay(300)`. Remove `ai-hint-a`, add `ai-hint-b`. `await delay(250)`.
6. Swap `aiBoard[r1][c1]` and `aiBoard[r2][c2]`. Call `renderAI()`.
7. `await resolveMatchesOn(aiBoard, 'ai')`.
8. `aiBusy = false`. If `gameActive`: `scheduleAI()`.

### `findBestAIMove()`

Tests every right-and-down adjacent pair on `aiBoard`. For each pair: temporarily swap, call `findMatchesOn(aiBoard)`, restore. Collect all pairs that produce at least one match into `candidates` with their match count. Sort descending by match count. Return `[candidates[0][0], candidates[0][1]]` or `null` if none found.

---

## Step 19 — Shared match logic

### `findMatchesOn(b)`

Board-agnostic (accepts player board or AI board). Uses `Set<number>` of flat indices (`r * COLS + c`) to deduplicate cells at match intersections.

Horizontal scan: for each row `r`, scan `c` from 0 to COLS-3. When 3+ identical non-null gems are found, extend the run as far as it goes, add all flat indices to `hit`.

Vertical scan: same logic per column.

Return `[...hit].map(i => [Math.floor(i / COLS), i % COLS])`.

### `resolveMatchesOn(b, who)` (async)

Loops until `findMatchesOn(b)` returns empty:

1. `matches = findMatchesOn(b)`. If empty, exit.
2. Score update:
   - `'player'`: `score += matches.length * 10; updateScores(); sndMatch(matches.length); comboCount++`.
   - `'ai'`: `aiScore += matches.length * 10; updateScores()`. No sound.
3. For each `[r, c]` in matches: add class `matched` to the appropriate cell element, set `b[r][c] = null`.
4. `await delay(290)` (pop animation duration).
5. `gravityOn(b)`.
6. Player: `sndFall(); render()`. AI: `renderAI()`.
7. `await delay(180)`. Loop.

### `gravityOn(b)`

For each column `c`: use a write pointer `empty = ROWS - 1` scanning downward. Non-null gems are packed to the bottom in their original relative order. Remaining top slots are filled with `GEMS[Math.floor(Math.random() * GEMS.length)]`. Never touches the DOM.

---

## Step 20 — End game

### `showEndModal()`

1. Set `#end-you-score` and `#end-ai-score` text content.
2. Determine outcome and update wins:
   - `score > aiScore`: `title = 'You Win!'`, `cls = 'win'`, `wins.you++`, `sndWin()`.
   - `aiScore > score`: `title = 'AI Wins'`, `cls = 'lose'`, `wins.ai++`, `sndLose()`.
   - Equal: `title = "It's a Tie"`, `cls = 'tie'`, `sndTie()`.
3. Set `#end-title` textContent and className (`modal-title win/lose/tie`).
4. Set `#end-sub` textContent with a flavour sentence.
5. Add class `show` to `#end-modal`.
6. Call `updateLeaderboard()`.

---

## Step 21 — Leaderboard

### `updateLeaderboard()`

1. Build entries: `[{ name: 'You', wins: wins.you, isYou: true }, { name: 'AI', wins: wins.ai, isYou: false }]`.
2. Sort descending by `wins`.
3. For each position `i` (0 and 1), update `#lb-row-{i+1}`, `#lb-av-{i+1}`, `#lb-name-{i+1}`, `#lb-wins-{i+1}`:
   - Avatar text content: `'YOU'` or `'AI'`.
   - Avatar class: `lb-avatar you-av` or `lb-avatar ai-av`.
   - Wins class: `lb-wins you-wins` or `lb-wins ai-wins`.
4. If `entries[0].wins > 0`, add class `leader` to `#lb-row-1` and inject a 👑 into the name via `innerHTML`. Otherwise remove `leader`.

Call once at script end (before any game starts) and once inside `showEndModal()`.

---

## Step 22 — UI helpers

### `updateScores()`
```js
document.getElementById('score-val').textContent = score;
document.getElementById('ai-score-val').textContent = aiScore;
```

### `setMsg(text, cls = '')`
```js
const el = document.getElementById('msg');
el.textContent = text;
el.className = cls;
```

Use `setMsg('no match', 'bad')` for errors, `setMsg('')` to clear.

### `delay(ms)`
```js
function delay(ms) { return new Promise(r => setTimeout(r, ms)); }
```

---

## Step 23 — Initialization

The last line of the script must be:
```js
updateLeaderboard();
```

This populates the leaderboard at 0–0 before the first game. The game does not begin until the user clicks "Start Game" on the modal.

---

## Step 24 — Verification checklist

Before outputting the file, confirm every item:

- [ ] Single HTML file, no external JS libraries
- [ ] Google Fonts link: `DM Mono` (400, 500) and `DM Sans` (400, 500, 600)
- [ ] All CSS variables on `:root`, including `--ai-accent: #fb923c` and `--ai-glow`
- [ ] `--cell-size: 44px`, `--gap: 3px`, `GAME_TIME = 30`
- [ ] `GEMS` has exactly 6 emoji: 🔴🟠🟡🟢🔵🟣
- [ ] Start modal has class `show` by default; end modal does not
- [ ] `#arena` order: AI side-panel → `#leaderboard` → player side-panel
- [ ] Both grids use class `grid` with separate IDs `#ai-grid` and `#grid`
- [ ] `getCell` scopes to `#grid`, `getAICell` scopes to `#ai-grid` — no cross-contamination
- [ ] `findMatchesOn(b)` is board-agnostic; uses flat-index `Set` for deduplication
- [ ] `resolveMatchesOn(b, who)` updates the correct score; plays sounds only for `'player'`
- [ ] `gravityOn(b)` only modifies data; render is always called after it
- [ ] AI two-phase flash: `ai-hint-a` (300ms) then `ai-hint-b` (250ms) before swap executes
- [ ] `findBestAIMove()` picks the swap that clears the most gems
- [ ] Timer bar turns red (`fill.classList.toggle('low', ...)`) at `timeLeft <= 8`
- [ ] `sndTick()` fires each second while `timeLeft <= 8 && timeLeft > 0`
- [ ] `sndStartJingle()` is the first call in `startGame()` so AudioContext is unlocked by the gesture
- [ ] `comboCount` resets to 0 before and after each player swap; increments once per cascade wave
- [ ] `wins` persists across rounds — only `score` and `aiScore` reset per game
- [ ] `updateLeaderboard()` called at script end and inside `showEndModal()`
- [ ] `#lb-row-1` gets class `leader` and 👑 in name when `entries[0].wins > 0`
- [ ] `busy` blocks player clicks during all async operations
- [ ] `gameActive` blocks player clicks when game has not started or has ended