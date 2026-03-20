# Agent Prompt: Build a Match-3 Game (Exact Specification)

You are to produce a single self-contained HTML file that implements a fully functional Match-3 puzzle game. Follow every instruction below exactly. Do not add features, do not change the visual design, do not introduce external libraries beyond the fonts specified.

---

## Step 1 — File structure

Create one file: `match3.html`. It must contain, in order:
1. A standard HTML5 `<!DOCTYPE html>` boilerplate with `<head>` and `<body>`.
2. A `<link>` tag loading two Google Fonts: `DM Mono` (weights 400, 500) and `DM Sans` (weights 400, 500, 600).
3. A single `<style>` block containing ALL CSS.
4. A single `<div id="wrap">` containing ALL HTML markup.
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
--accent: #a78bfa
--accent-glow: rgba(167,139,250,0.25)
--cell-size: 52px
--gap: 4px
```

---

## Step 3 — Body and background

Style `body` as:
- `background: var(--bg)`
- `color: var(--text)`
- `font-family: 'DM Sans', sans-serif`
- `min-height: 100vh`
- `display: flex; align-items: center; justify-content: center`
- `padding: 2rem 1rem`

Add a `body::before` pseudo-element as a decorative grid overlay:
- `position: fixed; inset: 0; pointer-events: none`
- `background-image`: two layered `linear-gradient` calls that draw 1px lines at `rgba(255,255,255,0.02)` every 40px — one horizontal, one vertical (use `90deg`).
- `background-size: 40px 40px`

---

## Step 4 — HTML layout (write this markup inside `<body>`)

```html
<div id="wrap">

  <header>
    <h1>Match Three</h1>
    <div id="tagline">swap adjacent gems · clear 3 or more</div>
  </header>

  <div id="stats">
    <div class="stat">
      <span class="stat-label">Score</span>
      <span class="stat-value" id="score-val">0</span>
    </div>
    <div class="stat">
      <span class="stat-label">Moves</span>
      <span class="stat-value" id="moves-val">30</span>
    </div>
  </div>

  <div id="board-wrap">
    <div id="grid"></div>
    <div id="overlay">
      <div id="overlay-title">Game Over</div>
      <div id="overlay-score">0</div>
      <div id="overlay-sub">final score</div>
      <button class="overlay-btn" onclick="initGame()">Play Again</button>
    </div>
  </div>

  <div id="bottom">
    <div id="msg"></div>
    <button id="new-btn" onclick="initGame()">New Game</button>
  </div>

</div>
```

Important notes:
- `#grid` is empty — JavaScript fills it entirely at runtime.
- `#overlay` sits inside `#board-wrap` and is shown/hidden by adding/removing the CSS class `show`.
- `#msg` is an inline feedback area (e.g. "no match").

---

## Step 5 — CSS: layout containers

**`#wrap`**: `display: flex; flex-direction: column; align-items: center; gap: 20px; position: relative; z-index: 1`

**`header`**: `display: flex; flex-direction: column; align-items: center; gap: 4px`

**`h1`**: DM Mono, 13px, weight 500, `letter-spacing: 0.18em`, `text-transform: uppercase`, `color: var(--accent)`

**`#tagline`**: 12px, `color: var(--text-muted)`, `letter-spacing: 0.04em`

---

## Step 6 — CSS: stat bar

**`#stats`**: `display: flex; gap: 2px; background: var(--surface); border: 1px solid var(--border); border-radius: 12px; overflow: hidden`

**`.stat`**: `display: flex; flex-direction: column; align-items: center; padding: 10px 28px; gap: 2px`

**`.stat + .stat`**: `border-left: 1px solid var(--border)` (divider between the two stats)

**`.stat-label`**: DM Mono, 10px, `letter-spacing: 0.12em`, `text-transform: uppercase`, `color: var(--text-muted)`

**`.stat-value`**: DM Mono, 22px, weight 500, `color: var(--text)`, `line-height: 1`, `min-width: 3ch`, `text-align: center`

**`#score-val`**: override color to `var(--accent)` (score displays in purple, moves stays white)

---

## Step 7 — CSS: board

**`#board-wrap`**: `background: var(--surface); border: 1px solid var(--border); border-radius: 16px; padding: 14px; position: relative`
(The `position: relative` is required so the overlay positions against it.)

**`#grid`**: CSS Grid — `grid-template-columns: repeat(8, var(--cell-size)); grid-template-rows: repeat(8, var(--cell-size)); gap: var(--gap)`

---

## Step 8 — CSS: cells

**`.cell`** (base state):
```
width: var(--cell-size)
height: var(--cell-size)
border-radius: 10px
display: flex; align-items: center; justify-content: center
font-size: 24px
cursor: pointer
border: 1.5px solid transparent
background: var(--surface2)
transition: transform 0.12s ease, border-color 0.12s ease, background 0.12s ease
user-select: none
position: relative
```

**`.cell:hover`**: `transform: scale(1.07); background: #2a2a38; border-color: var(--border-strong)`

**`.cell.selected`**: `border-color: var(--accent); background: rgba(167,139,250,0.12); transform: scale(1.1); box-shadow: 0 0 0 3px var(--accent-glow)`

**`.cell.matched`**: Apply animation `pop 0.28s ease forwards`

```css
@keyframes pop {
  0%   { transform: scale(1); opacity: 1; }
  40%  { transform: scale(1.25); opacity: 0.6; }
  100% { transform: scale(0); opacity: 0; }
}
```

**`.cell.falling`**: Apply animation `fall 0.2s cubic-bezier(0.22, 1, 0.36, 1)`

```css
@keyframes fall {
  from { transform: translateY(calc(-1 * var(--cell-size))); opacity: 0.7; }
  to   { transform: translateY(0); opacity: 1; }
}
```

---

## Step 9 — CSS: bottom bar and message

**`#bottom`**: `display: flex; align-items: center; gap: 12px; width: 100%; justify-content: space-between`

**`#msg`**: DM Mono, 12px, `color: var(--text-muted)`, `letter-spacing: 0.05em`, `min-height: 16px`, `flex: 1`

**`#msg.bad`**: `color: #f87171` (red — used for "no match" feedback)

**`#msg.good`**: `color: #6ee7b7` (green — available for positive feedback if needed)

**`#new-btn`**:
```
font-family: DM Mono; font-size: 11px; font-weight: 500
letter-spacing: 0.1em; text-transform: uppercase
color: var(--accent); background: transparent
border: 1px solid rgba(167,139,250,0.35); border-radius: 8px
padding: 7px 16px; cursor: pointer
transition: background 0.15s, border-color 0.15s
white-space: nowrap
```

**`#new-btn:hover`**: `background: rgba(167,139,250,0.1); border-color: var(--accent)`

**`#new-btn:active`**: `transform: scale(0.97)`

---

## Step 10 — CSS: game-over overlay

**`#overlay`** (hidden by default):
```
display: none
position: absolute; inset: 0
border-radius: 15px
background: rgba(15,15,19,0.88)
backdrop-filter: blur(4px)
flex-direction: column; align-items: center; justify-content: center
gap: 12px
z-index: 10
```

**`#overlay.show`**: `display: flex` (this is how it becomes visible — JS adds class `show`)

**`#overlay-title`**: DM Mono, 13px, `letter-spacing: 0.2em`, `text-transform: uppercase`, `color: var(--accent)`

**`#overlay-score`**: DM Mono, 42px, weight 500, `color: var(--text)`, `line-height: 1`

**`#overlay-sub`**: 12px, `color: var(--text-muted)`, `margin-bottom: 4px`

**`.overlay-btn`**:
```
font-family: DM Mono; font-size: 11px; font-weight: 500
letter-spacing: 0.12em; text-transform: uppercase
color: var(--bg); background: var(--accent)
border: none; border-radius: 8px
padding: 10px 24px; cursor: pointer
transition: opacity 0.15s
```

**`.overlay-btn:hover`**: `opacity: 0.85`

---

## Step 11 — JavaScript: constants and state

At the top of the `<script>` block, declare:

```js
const GEMS = ['🔴','🟠','🟡','🟢','🔵','🟣'];
const COLS = 8, ROWS = 8;
let board = [], selected = null, score = 0, moves = 30, busy = false;
```

- `GEMS`: array of 6 emoji used as gem types. Each cell on the board holds one of these strings.
- `COLS` / `ROWS`: board dimensions, both 8.
- `board`: a 2D array `[ROWS][COLS]` where each entry is a gem string or `null` (null means cleared, awaiting gravity).
- `selected`: either `null` (nothing selected) or `[r, c]` (row/column of the currently highlighted cell).
- `score`: integer, starts at 0, incremented by 10 per gem cleared.
- `moves`: integer countdown from 30. Decremented once per valid swap only. Game ends when it reaches 0.
- `busy`: boolean lock. When `true`, all click handlers do nothing. Prevents the user from clicking during async animations.

---

## Step 12 — `initGame()`

**Purpose**: Reset all state and start a fresh game.

**Steps**:
1. Set `score = 0`, `moves = 30`, `selected = null`, `busy = false`.
2. Remove the CSS class `show` from `#overlay` (hides the game-over screen).
3. Generate a new board using a `do...while` loop:
   - Inside the loop: fill `board` as a 2D array using `Array.from({length: ROWS}, () => Array.from({length: COLS}, () => GEMS[Math.floor(Math.random() * GEMS.length)]))`.
   - Continue looping `while (findMatches().length > 0)`. This guarantees the starting board has zero pre-existing matches.
4. Call `render()`, `updateInfo()`, and `setMsg('')`.

---

## Step 13 — `render()`

**Purpose**: Fully re-draw the grid by clearing and rebuilding all DOM cells from the `board` array.

**Steps**:
1. Get the `#grid` element and set `grid.innerHTML = ''`.
2. Loop `r` from 0 to ROWS-1, `c` from 0 to COLS-1 (row-major order, so row 0 col 0 is top-left).
3. For each `[r, c]`, create a `<div>`:
   - `className = 'cell'`
   - `textContent = board[r][c]` (the emoji string)
   - `dataset.r = r`, `dataset.c = c` (stored for `getCell()` lookups)
   - `onclick = () => onCellClick(r, c)` (closure captures current r, c)
   - If `selected` is not null AND `selected[0] === r` AND `selected[1] === c`: add class `selected` to highlight it.
4. Append each cell to `#grid`.

**Important**: `render()` always destroys and recreates every cell. It is the single source of truth for what the grid looks like.

---

## Step 14 — `getCell(r, c)`

**Purpose**: Return the live DOM element for a given board position, used during animations.

**Implementation**:
```js
return document.querySelector(`[data-r="${r}"][data-c="${c}"]`);
```

This works because `render()` sets `data-r` and `data-c` on every cell.

---

## Step 15 — `onCellClick(r, c)`

**Purpose**: Handle all user click input. Implements a two-click selection model.

**Steps** (execute in this exact order):
1. If `busy` is true OR `moves <= 0`: return immediately (do nothing).
2. If `selected` is null: set `selected = [r, c]`, call `render()`, return. (First click — select the gem.)
3. Destructure `selected` into `[sr, sc]`.
4. If `sr === r && sc === c`: set `selected = null`, call `render()`, return. (Clicking the same cell deselects it.)
5. Compute Manhattan distance: `Math.abs(sr - r) + Math.abs(sc - c)`. If this is NOT exactly 1: set `selected = [r, c]`, call `render()`, return. (Non-adjacent cell — treat it as a new first selection.)
6. Set `selected = null`.
7. Call `doSwap(sr, sc, r, c)`. (Adjacent cell — attempt a swap.)

---

## Step 16 — `doSwap(r1, c1, r2, c2)` (async)

**Purpose**: Attempt a swap between two adjacent cells. Reverse it if no match results. Count the move and trigger match resolution if valid.

**Steps**:
1. Set `busy = true`.
2. Swap the two cells in `board`: `[board[r1][c1], board[r2][c2]] = [board[r2][c2], board[r1][c1]]`.
3. Call `render()` to show the swapped state immediately.
4. Call `findMatches()`. If the result array is empty (no matches produced):
   - Reverse the swap: `[board[r1][c1], board[r2][c2]] = [board[r2][c2], board[r1][c1]]`.
   - Call `render()`.
   - Call `setMsg('no match', 'bad')`.
   - After 900ms (via `setTimeout`), call `setMsg('')`.
   - Set `busy = false`.
   - Return early.
5. If matches were found:
   - Decrement `moves` by 1.
   - Call `updateInfo()`.
   - `await resolveMatches()`.
   - Set `busy = false`.
   - If `moves <= 0`, call `endGame()`.

**Key rule**: `moves` is only decremented for swaps that produce at least one match.

---

## Step 17 — `findMatches()`

**Purpose**: Scan the entire board and return an array of all `[r, c]` positions that are part of a match (3 or more identical adjacent gems in a row or column). Uses a `Set` of flat indices to automatically deduplicate cells belonging to both a row and column match.

**Steps**:
1. Create `const hit = new Set()`.
2. **Horizontal scan** — loop `r` from 0 to ROWS-1, `c` from 0 to COLS-3:
   - If `board[r][c]` is truthy AND equals `board[r][c+1]` AND equals `board[r][c+2]`:
     - Start a run counter `l = 3`.
     - Extend: while `c + l < COLS` and `board[r][c+l] === board[r][c]`, increment `l`.
     - Add flat indices: for `k = 0` to `l-1`, add `r * COLS + (c + k)` to `hit`.
3. **Vertical scan** — loop `c` from 0 to COLS-1, `r` from 0 to ROWS-3:
   - If `board[r][c]` is truthy AND equals `board[r+1][c]` AND equals `board[r+2][c]`:
     - Start a run counter `l = 3`.
     - Extend: while `r + l < ROWS` and `board[r+l][c] === board[r][c]`, increment `l`.
     - Add flat indices: for `k = 0` to `l-1`, add `(r + k) * COLS + c` to `hit`.
4. Convert `hit` to an array and map each flat index `i` back to `[Math.floor(i / COLS), i % COLS]`.
5. Return that array.

**Why flat indices**: A cell at the intersection of a row match and a column match (an L- or T-shape) would otherwise be added twice. The `Set` deduplicates by the integer key `r * COLS + c`.

---

## Step 18 — `resolveMatches()` (async)

**Purpose**: Run the full cascade loop — clear matches, apply gravity, re-render, repeat until no matches remain.

**Steps** (inside a `while` loop):
1. Call `findMatches()`. If the result is empty, exit the loop.
2. Add `matches.length * 10` to `score`. Call `updateInfo()`.
3. For each `[r, c]` in the matches array:
   - Call `getCell(r, c)` to get the DOM element and add class `matched` to it (triggers the pop animation).
   - Set `board[r][c] = null`.
4. `await delay(290)` — wait for the pop animation to finish before dropping gems.
5. Call `gravity()` — compacts the board data (no DOM changes yet).
6. Call `render()` — redraws the entire grid with the compacted data.
7. `await delay(210)` — brief pause after the fall before checking for new matches.
8. Loop back to step 1.

**Why the loop**: A single swap can trigger a cascade. After gravity fills the board, new 3-in-a-row matches may appear that must also be cleared without costing the player a move.

---

## Step 19 — `gravity()`

**Purpose**: Compact each column of `board` downward, filling cleared (`null`) cells, then fill the top of each column with new random gems.

**Steps** — for each column `c` from 0 to COLS-1:
1. Set a write pointer `empty = ROWS - 1` (starts at the bottom of the column).
2. Scan `r` from ROWS-1 down to 0:
   - If `board[r][c]` is not null:
     - Copy it to `board[empty][c]`.
     - If `empty !== r`, set `board[r][c] = null` (clear the old position).
     - Decrement `empty`.
3. After the scan, every index from `r = 0` to `r = empty` (inclusive) is still null (the top of the column is empty). Fill each with a new random gem: `board[r][c] = GEMS[Math.floor(Math.random() * GEMS.length)]`.

**Effect**: All non-null gems in a column are packed to the bottom in their original relative order. New random gems fill any remaining space at the top.

**Important**: `gravity()` only modifies the `board` data array. It does not touch the DOM. Always call `render()` after `gravity()` to reflect changes visually.

---

## Step 20 — `endGame()`

**Purpose**: Show the game-over overlay with the player's final score.

**Steps**:
1. Get `#overlay` element.
2. Set the `textContent` of `#overlay-score` to the current value of `score`.
3. Add class `show` to `#overlay`. (CSS rule `#overlay.show { display: flex }` makes it visible.)

---

## Step 21 — `updateInfo()`

**Purpose**: Sync the stat bar DOM to the current `score` and `moves` values.

```js
document.getElementById('score-val').textContent = score;
document.getElementById('moves-val').textContent = moves;
```

---

## Step 22 — `setMsg(text, cls = '')`

**Purpose**: Update the `#msg` feedback element with a message and optional CSS class.

```js
const el = document.getElementById('msg');
el.textContent = text;
el.className = cls;
```

- Call `setMsg('no match', 'bad')` to show a red error message.
- Call `setMsg('')` to clear it (empty string, no class).

---

## Step 23 — `delay(ms)`

**Purpose**: A simple promise-based timer used with `await` in async functions.

```js
function delay(ms) { return new Promise(r => setTimeout(r, ms)); }
```

---

## Step 24 — Initialization

The last line of the script must be:
```js
initGame();
```

This runs on page load and starts the first game.

---

## Step 25 — Verification checklist

Before outputting the file, confirm:

- [ ] Single HTML file, no external JS
- [ ] Google Fonts link present (`DM Mono` + `DM Sans`)
- [ ] All 10 CSS variables declared on `:root`
- [ ] Grid is `8×8` (COLS = 8, ROWS = 8)
- [ ] `GEMS` has exactly 6 emoji: 🔴🟠🟡🟢🔵🟣
- [ ] `initGame()` regenerates board until `findMatches()` returns empty
- [ ] `onCellClick` enforces Manhattan distance = 1 before swapping
- [ ] `doSwap` reverses the swap if no matches found and does NOT decrement moves
- [ ] `findMatches` uses a `Set` of flat indices to deduplicate
- [ ] `gravity()` packs gems downward and fills nulls from top with new random gems
- [ ] `resolveMatches` loops until no matches remain (cascade support)
- [ ] `#overlay` is hidden by default (`display: none`) and shown by adding class `show`
- [ ] `busy` flag prevents clicks during all async operations
- [ ] `initGame()` is called at the bottom of the script to auto-start