# AI Agent Prompt: Build a Match-3 Game (Single HTML File)

You are building a fully self-contained Match-3 puzzle game in a **single HTML file** (no external JS libraries, no build tools). Follow every step below exactly. Do not deviate from the specified logic, structure, or styling.

### Running and delivering the game

- The result is **static HTML**: open `match3.html` directly in a browser (double-click or drag into a tab). **No dev server, bundler, or build step is required** for the game to work.
- Keep the script as a **classic** `<script>` block (do **not** use `type="module"`). Inline handlers such as `onclick="initGame()"` expect `initGame` to exist as a **global** function; a top-level `function initGame() { … }` declaration achieves that. If you use only `const initGame = () => …`, you must also assign `window.initGame = initGame`.

---

## STEP 1 — File Structure

Create one file: `match3.html`. It must contain, in this order:

1. `<!DOCTYPE html>` declaration
2. `<head>` with charset (`<meta charset="UTF-8">`), viewport meta (`width=device-width, initial-scale=1`), Google Fonts `<link>`, and a `<style>` block
3. `<body>` with the full HTML layout
4. A single **non-module** `<script>` block at the bottom of `<body>` containing all game logic

Optional but reasonable: wrap the document in `<html lang="en">` … `</html>` for accessibility.

---

## STEP 2 — Fonts

Load from Google Fonts via `<link>` in `<head>`:

```
https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=DM+Sans:wght@400;500;600&display=swap
```

- **DM Mono** — used for the title, all stat labels, stat values, message text, and both buttons
- **DM Sans** — used as the `body` base font for everything else

---

## STEP 3 — CSS Variables

Declare these on `:root`:

```css
--bg: #0f0f13;
--surface: #1a1a22;
--surface2: #22222e;
--border: rgba(255,255,255,0.08);
--border-strong: rgba(255,255,255,0.18);
--text: #f0eff4;
--text-muted: #7c7a8e;
--accent: #a78bfa;
--accent-glow: rgba(167,139,250,0.25);
--cell-size: 52px;
--gap: 4px;
```

---

## STEP 4 — CSS Rules (apply exactly as described)

### Global reset
```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
```

### `body`
- `background: var(--bg)`
- `color: var(--text)`
- `font-family: 'DM Sans', sans-serif`
- `min-height: 100vh`
- `display: flex`, `align-items: center`, `justify-content: center`
- `padding: 2rem 1rem`

### `body::before` (decorative background grid)
- `content: ''`, `position: fixed`, `inset: 0`
- `background-image`: two overlapping 1px linear-gradient lines (horizontal and vertical) at `rgba(255,255,255,0.02)`, `background-size: 40px 40px`
- `pointer-events: none`

### `#wrap`
- `display: flex`, `flex-direction: column`, `align-items: center`, `gap: 20px`
- `position: relative`, `z-index: 1`

### `header`
- `display: flex`, `flex-direction: column`, `align-items: center`, `gap: 4px`

### `h1`
- Font: DM Mono, `font-size: 13px`, `font-weight: 500`
- `letter-spacing: 0.18em`, `text-transfo

rm: uppercase`
- `color: var(--accent)`

### `#tagline`
- `font-size: 12px`, `color: var(--text-muted)`, `letter-spacing: 0.04em`

### `#stats` (stat bar container)
- `display: flex`, `gap: 2px`
- `background: var(--surface)`, `border: 1px solid var(--border)`, `border-radius: 12px`, `overflow: hidden`

### `.stat` (individual stat block)
- `display: flex`, `flex-direction: column`, `align-items: center`
- `padding: 10px 28px`, `gap: 2px`

### `.stat + .stat`
- `border-left: 1px solid var(--border)` (divider between the two stat blocks)

### `.stat-label`
- Font: DM Mono, `font-size: 10px`, `letter-spacing: 0.12em`, `text-transform: uppercase`
- `color: var(--text-muted)`

### `.stat-value`
- Font: DM Mono, `font-size: 22px`, `font-weight: 500`, `color: var(--text)`
- `line-height: 1`, `min-width: 3ch`, `text-align: center`

### `#score-val`
- `color: var(--accent)` (overrides `.stat-value` color for the score number only)

### `#board-wrap`
- `background: var(--surface)`, `border: 1px solid var(--border)`, `border-radius: 16px`
- `padding: 14px`, `position: relative`

### `#grid`
- `display: grid`
- `grid-template-columns: repeat(8, var(--cell-size))`
- `grid-template-rows: repeat(8, var(--cell-size))`
- `gap: var(--gap)`

### `.cell`
- `width: var(--cell-size)`, `height: var(--cell-size)`, `border-radius: 10px`
- `display: flex`, `align-items: center`, `justify-content: center`
- `font-size: 24px`, `cursor: pointer`
- `border: 1.5px solid transparent`, `background: var(--surface2)`
- `transition: transform 0.12s ease, border-color 0.12s ease, background 0.12s ease`
- `user-select: none`, `position: relative`

### `.cell:hover`
- `transform: scale(1.07)`, `background: #2a2a38`, `border-color: var(--border-strong)`

### `.cell.selected`
- `border-color: var(--accent)`, `background: rgba(167,139,250,0.12)`
- `transform: scale(1.1)`, `box-shadow: 0 0 0 3px var(--accent-glow)`

### `.cell.matched` (pop-out animation on clear)
```css
animation: pop 0.28s ease forwards;

@keyframes pop {
  0%   { transform: scale(1); opacity: 1; }
  40%  { transform: scale(1.25); opacity: 0.6; }
  100% { transform: scale(0); opacity: 0; }
}
```

### `.cell.falling` (drop-in animation for new gems)
```css
animation: fall 0.2s cubic-bezier(0.22, 1, 0.36, 1);

@keyframes fall {
  from { transform: translateY(calc(-1 * var(--cell-size))); opacity: 0.7; }
  to   { transform: translateY(0); opacity: 1; }
}
```

> **Note:** Include this rule in the stylesheet as specified. The algorithm steps below do **not** require toggling the `falling` class unless you voluntarily extend the spec (e.g. mark newly spawned gems after `gravity()`). A correct minimal implementation can leave `falling` unused in JavaScript.

### `#bottom`
- `display: flex`, `align-items: center`, `gap: 12px`
- `width: 100%`, `justify-content: space-between`

### `#msg`
- Font: DM Mono, `font-size: 12px`, `color: var(--text-muted)`
- `letter-spacing: 0.05em`, `min-height: 16px`, `flex: 1`

### `#msg.bad` → `color: #f87171`
### `#msg.good` → `color: #6ee7b7`

### `#new-btn`
- Font: DM Mono, `font-size: 11px`, `font-weight: 500`, `letter-spacing: 0.1em`, `text-transform: uppercase`
- `color: var(--accent)`, `background: transparent`
- `border: 1px solid rgba(167,139,250,0.35)`, `border-radius: 8px`, `padding: 7px 16px`
- `cursor: pointer`, `transition: background 0.15s, border-color 0.15s`, `white-space: nowrap`

### `#new-btn:hover`
- `background: rgba(167,139,250,0.1)`, `border-color: var(--accent)`

### `#new-btn:active`
- `transform: scale(0.97)`

### `#overlay` (game-over panel, overlaid on top of the grid)
- `display: none` (hidden by default)
- `position: absolute`, `inset: 0`, `border-radius: 15px`
- `background: rgba(15,15,19,0.88)`, `backdrop-filter: blur(4px)`
- `flex-direction: column`, `align-items: center`, `justify-content: center`, `gap: 12px`
- `z-index: 10`

### `#overlay.show`
- `display: flex` (makes the overlay visible)

### `#overlay-title`
- Font: DM Mono, `font-size: 13px`, `letter-spacing: 0.2em`, `text-transform: uppercase`
- `color: var(--accent)`

### `#overlay-score`
- Font: DM Mono, `font-size: 42px`, `font-weight: 500`, `color: var(--text)`, `line-height: 1`

### `#overlay-sub`
- `font-size: 12px`, `color: var(--text-muted)`, `margin-bottom: 4px`

### `.overlay-btn`
- Font: DM Mono, `font-size: 11px`, `font-weight: 500`, `letter-spacing: 0.12em`, `text-transform: uppercase`
- `color: var(--bg)` (dark text on accent background)
- `background: var(--accent)`, `border: none`, `border-radius: 8px`, `padding: 10px 24px`
- `cursor: pointer`, `transition: opacity 0.15s`

### `.overlay-btn:hover`
- `opacity: 0.85`

---

## STEP 5 — HTML Layout (inside `<body>`)

Build this exact DOM structure:

```
#wrap
  header
    h1 > "Match Three"
    #tagline > "swap adjacent gems · clear 3 or more"
  #stats
    .stat
      .stat-label > "Score"
      .stat-value#score-val > "0"
    .stat
      .stat-label > "Moves"
      .stat-value#moves-val > "30"
  #board-wrap
    #grid   ← populated entirely by JavaScript, starts empty
    #overlay
      #overlay-title > "Game Over"
      #overlay-score > "0"
      #overlay-sub > "final score"
      button.overlay-btn [onclick="initGame()"] > "Play Again"
  #bottom
    #msg    ← empty initially
    button#new-btn [onclick="initGame()"] > "New Game"
```

**Concrete HTML details:**

- **`#tagline`:** Use a single element with `id="tagline"` (e.g. `<p id="tagline">` or `<div id="tagline">`). It only needs the text and the styles from Step 4.
- **Buttons:** Use `<button type="button" …>` for both `Play Again` and `New Game` so they never act as submit buttons if the layout is ever wrapped in a `<form>`.
- **`#board-wrap` children:** `#grid` must come **before** `#overlay` in the DOM so the grid paints underneath; `#overlay` uses `position: absolute` and a higher `z-index` to cover the board area inside the padded wrap.
- **IDs:** Keep `id` values exactly as named (`score-val`, `moves-val`, `overlay-score`, etc.) so `getElementById` in the script matches without ambiguity.

---

## STEP 6 — JavaScript: Constants and State

Declare these at the top of the `<script>` block:

```js
const GEMS = ['🔴','🟠','🟡','🟢','🔵','🟣'];  // 6 gem types
const COLS = 8, ROWS = 8;                        // 8×8 grid
let board = [];      // 2D array [row][col] → gem emoji string or null
let selected = null; // [row, col] of the currently selected cell, or null
let score = 0;       // current score (starts 0)
let moves = 30;      // moves remaining (starts 30)
let busy = false;    // true while async swap/resolve is in progress (blocks input)
```

**Coordinate system:** `board` is indexed `board[row][col]` with `row = 0` at the **top** of the grid and `row = ROWS - 1` at the **bottom**. Column `0` is the left edge. This matches typical 2D grid loops (`r` then `c`).

**Cell values:** After a match is processed, cleared cells are **`null`**. All other cells hold one of the `GEMS` strings. Do not leave cleared cells as `undefined` unless you treat them identically to `null` everywhere (the reference logic uses only `null`).

---

## STEP 7 — Function: `initGame()`

**Purpose:** Reset all state and start a fresh game.

**Steps:**
1. Set `score = 0`, `moves = 30`, `selected = null`, `busy = false`
2. Remove the `show` class from `#overlay` (hides the game-over panel)
3. Generate a random 8×8 board: use `Array.from` to create a ROWS-length array of COLS-length arrays, each cell filled with a random item from `GEMS` using `Math.floor(Math.random() * GEMS.length)`
4. Run this in a `do…while` loop: keep regenerating the entire board until `findMatches()` returns an empty array. This ensures the initial board has zero pre-existing matches.
   - In code, treat “empty” as **`findMatches().length === 0`** (or assign the result to a variable and check `.length`).
5. Call `render()`, `updateInfo()`, and `setMsg('')`

---

## STEP 8 — Function: `render()`

**Purpose:** Wipe and redraw the entire `#grid` DOM from the current `board` array.

**Steps:**
1. Get `#grid` and set `innerHTML = ''`
2. Loop `r` from 0 to ROWS-1, and inside that loop `c` from 0 to COLS-1
3. For each `[r, c]`, create a `<div>`:
   - `className = 'cell'`
   - `textContent = board[r][c]` — if the value can be `null` during intermediate states, coerce with **`board[r][c] ?? ''`** so the UI never shows the literal string `"null"`.
   - `dataset.r` and `dataset.c`: set from numeric `r` / `c` (the DOM stores them as strings). **`getCell`** must query with the same string form, e.g. `` `[data-r="${r}"][data-c="${c}"]` ``.
   - `onclick = () => onCellClick(r, c)` (closure captures current r, c)
   - If `selected` is not null AND `selected[0] === r` AND `selected[1] === c`, add class `'selected'`
4. Append each cell div to `#grid`

> **Important:** `render()` is a full repaint — it does not do partial DOM updates. Every call clears and rebuilds the entire grid.

---

## STEP 9 — Function: `getCell(r, c)`

**Purpose:** Return the DOM element for a specific grid position.

**Implementation:**
```js
return document.querySelector(`[data-r="${r}"][data-c="${c}"]`);
```

Used during match animation to add the `matched` class to the correct element before wiping the board.

**Timing:** In `resolveMatches`, call `getCell` and add `'matched'` **before** `await delay(290)`, and do **not** call `render()` in between. That way the nodes from the last `render()` still exist, the pop animation runs on screen, then `gravity()` + `render()` rebuild the grid.

---

## STEP 10 — Function: `onCellClick(r, c)`

**Purpose:** Handle the selection and swap logic when a user clicks a cell.

**Steps (execute in this exact order):**

1. If `busy === true` OR `moves <= 0` → return immediately (ignore the click)
2. If `selected === null` (nothing is selected yet):
   - Set `selected = [r, c]`
   - Call `render()` to show the selection highlight
   - Return
3. Destructure `selected` into `[sr, sc]`
4. If `sr === r && sc === c` (user clicked the already-selected cell):
   - Set `selected = null`
   - Call `render()` to remove the highlight
   - Return
5. Calculate Manhattan distance: `Math.abs(sr - r) + Math.abs(sc - c)`
6. If the distance is NOT equal to 1 (cells are not adjacent):
   - Set `selected = [r, c]` (move selection to the newly clicked cell instead)
   - Call `render()`
   - Return
7. If distance IS 1 (valid adjacent pair):
   - Set `selected = null`
   - Call `doSwap(sr, sc, r, c)` — **do not** `await` it here. The click handler stays synchronous; `doSwap` runs its async body in the background. Using `await` inside `onCellClick` would require making `onCellClick` async and changes the intended flow.

---

## STEP 11 — Function: `doSwap(r1, c1, r2, c2)` (async)

**Purpose:** Attempt to swap two adjacent cells. Reverse the swap if it produces no matches.

**Declaration:** Implement as **`async function doSwap(r1, c1, r2, c2) { … }`** so you can use **`await resolveMatches()`** inside the valid-swap branch.

**Steps:**

1. Set `busy = true`
2. Swap the two values in `board` using destructuring: `[board[r1][c1], board[r2][c2]] = [board[r2][c2], board[r1][c1]]`
3. Call `render()` so the player sees the swap visually
4. Call `findMatches()` and store the result (e.g. `matches`):
   - If **`matches.length === 0`** (invalid swap, no matches produced):
     - Swap the values back: `[board[r1][c1], board[r2][c2]] = [board[r2][c2], board[r1][c1]]`
     - Call `render()` to show the board restored
     - Call `setMsg('no match', 'bad')` to show red feedback text
     - After 900ms (`setTimeout`), call `setMsg('')` to clear the message
     - Set `busy = false`
     - Return (do not decrement moves)
5. If matches exist (valid swap):
   - Decrement `moves` by 1
   - Call `updateInfo()` to refresh the stat display
   - **`await resolveMatches()`** (handles chain reactions) — `resolveMatches` must be declared **`async`** for this to be valid
   - Set `busy = false` **only after** `resolveMatches` finishes (so input stays blocked through cascades)
   - If `moves <= 0`, call `endGame()` **after** resolution completes, so the final score includes all chain clears from that move

---

## STEP 12 — Function: `findMatches()`

**Purpose:** Scan the board and return all cell coordinates that are part of a match of 3 or more in a row (horizontal or vertical). Returns an array of `[row, col]` pairs.

**Implementation details:**

Use a `Set` called `hit` to store unique cell indices encoded as `row * COLS + col` (this prevents duplicates when a cell is part of both a horizontal and vertical match).

**Horizontal scan:**
- Loop `r` from 0 to ROWS-1
- Loop `c` from 0 to `COLS - 3` inclusive (so there are at least 3 cells to the right)
- Check: `board[r][c]` is truthy AND equals `board[r][c+1]` AND equals `board[r][c+2]`
- If match found, extend it: set `l = 3`, then **`while (c + l < COLS && board[r][c + l] === board[r][c]) l++`** — the “starting gem” for comparison is always **`board[r][c]`** (the leftmost cell of the run)
- Add all cells from column `c` to column `c+l-1` in row `r` to `hit` using the index formula

**Vertical scan:**
- Loop `c` from 0 to COLS-1
- Loop `r` from 0 to `ROWS - 3` inclusive
- Check: `board[r][c]` is truthy AND equals `board[r+1][c]` AND equals `board[r+2][c]`
- If match found, extend downward: set `l = 3`, then **`while (r + l < ROWS && board[r + l][c] === board[r][c]) l++`** — compare to **`board[r][c]`** (the topmost cell of the vertical run)
- Add all cells from row `r` to row `r+l-1` in column `c` to `hit`

**Return:** `[...hit].map(i => [Math.floor(i / COLS), i % COLS])`
- This decodes each flat index back to a `[row, col]` pair

---

## STEP 13 — Function: `resolveMatches()` (async)

**Purpose:** Clear all matched gems, apply gravity, and repeat until no more matches remain (handles chain reactions).

**Declaration:** **`async function resolveMatches() { … }`**. Use a loop such as **`while (true)`** with a **`break`** when `findMatches()` returns no cells, or an equivalent `do…while` — the important part is repeating the sequence until the board has no matches.

**Steps (loop until no matches):**

1. Call `findMatches()`. If the result is empty, exit the loop.
2. Add `matches.length * 10` to `score` (10 points per matched gem)
3. Call `updateInfo()`
4. For each `[r, c]` in matches:
   - Get the DOM element via `getCell(r, c)` and add class `'matched'` (triggers the pop animation)
   - Set `board[r][c] = null` (mark cell as empty in the data model)
5. `await delay(290)` — wait for the pop animation to finish before rebuilding the DOM
6. Call `gravity()` — fills in the null cells
7. Call `render()` — redraws the board
8. `await delay(210)` — brief pause before checking for new matches
9. Loop back to step 1

---

## STEP 14 — Function: `gravity()`

**Purpose:** After cells are set to `null` (cleared by matches), compact each column downward and fill gaps at the top with new random gems.

**Steps (process one column at a time):**

1. Loop `c` from 0 to COLS-1
2. For each column, set a pointer `empty = ROWS - 1` (starts at the bottom row)
3. Loop `r` from `ROWS - 1` downward to 0:
   - If **`board[r][c] !== null`** (the cell holds a gem; cleared cells are **`null`** only):
     - Copy it down: `board[empty][c] = board[r][c]`
     - If `empty !== r`, set `board[r][c] = null` (only clear the source if it actually moved)
     - Decrement `empty`
4. After the downward pass, rows **`0` through `empty` inclusive** are still empty at the top of that column (if any).
5. Fill those gaps: **`for (let r = empty; r >= 0; r--)`** assign `board[r][c] = GEMS[Math.floor(Math.random() * GEMS.length)]`.

> **How this works:** The `empty` pointer tracks the next available slot, starting from the bottom. Non-null gems are packed down to the `empty` slot, then `empty` moves up one. After the inner loop completes, any remaining positions above `empty` were null and get new random gems assigned.

> **Edge case:** If the column was full of gems, after packing, **`empty` is `-1`**. The fill loop **`for (r = empty; r >= 0; r--)`** runs **zero** iterations — correct. If the column was entirely cleared, **`empty === ROWS - 1`** before filling and you fill every row — also correct.

---

## STEP 15 — Function: `endGame()`

**Purpose:** Display the game-over overlay with the final score.

**Steps:**
1. Get the `#overlay` element
2. Set the text of `#overlay-score` to the final score — use **`String(score)`** (or template `` `${score}` ``) for `textContent`, since it always expects a string
3. Add class `'show'` to `#overlay` (CSS changes `display` from `none` to `flex`)

---

## STEP 16 — Function: `updateInfo()`

**Purpose:** Sync the stat bar display with current `score` and `moves` values.

**Steps:**
1. `document.getElementById('score-val').textContent = String(score)` (or coerce similarly)
2. `document.getElementById('moves-val').textContent = String(moves)`

Numbers assign fine to `textContent` in browsers, but string coercion avoids surprises if types ever change.

---

## STEP 17 — Function: `setMsg(text, cls = '')`

**Purpose:** Show or clear the feedback message below the board.

**Steps:**
1. Get `#msg`
2. Set `el.textContent = text`
3. Set `el.className = cls`

Pass `cls = 'bad'` for a red error message (no match). Pass no second argument (defaults to `''`) to clear styling. The CSS classes `bad` and `good` control the text color.

---

## STEP 18 — Function: `delay(ms)`

**Purpose:** Return a Promise that resolves after `ms` milliseconds. Used with `await` to pause async functions.

```js
function delay(ms) { return new Promise(r => setTimeout(r, ms)); }
```

---

## STEP 19 — Initialization

At the very bottom of the `<script>` block, after all function definitions, call:

```js
initGame();
```

This runs immediately when the script loads, populating the board and rendering the initial state.

**Suggested function order (avoid “used before initialization” errors):** Define helpers first (`delay`, `findMatches`, `gravity`, `getCell`), then anything that calls them (`resolveMatches`, `render`, `updateInfo`, `setMsg`, `endGame`, `doSwap`, `onCellClick`), then **`initGame`**, with the final **`initGame();`** call last. If you use `const name = () => {}` instead of `function name() {}`, strict order matters because `const` is not hoisted.

---

## STEP 20 — Verification Checklist

Before outputting the file, confirm every item below:

- [ ] Everything is in one `.html` file — no external `.js` or `.css` files; the file runs as static HTML (open directly in a browser)
- [ ] Script is not `type="module"` unless `initGame` is explicitly exposed on `window` for inline `onclick`
- [ ] The board is exactly 8 columns × 8 rows
- [ ] There are exactly 6 gem types: `['🔴','🟠','🟡','🟢','🔵','🟣']`
- [ ] The player starts with exactly 30 moves per game
- [ ] Invalid swaps (no resulting match) are reversed and cost 0 moves
- [ ] Valid swaps cost exactly 1 move and trigger full match resolution
- [ ] Chain reactions work: after clearing and refilling, the board is re-scanned and new matches also clear
- [ ] Scoring is 10 points per gem cleared (e.g. a 3-match = 30pts, a 5-match = 50pts)
- [ ] The game-over overlay appears when moves reach 0, showing the final score
- [ ] "New Game" button and "Play Again" button both call `initGame()` and fully reset all state
- [ ] The `busy` flag prevents any input during async swap/resolve operations
- [ ] The initial board is guaranteed to have no pre-existing matches (enforced by the `do…while` loop)
- [ ] All CSS uses the exact color hex values and variables defined in Step 3
- [ ] Fonts load from Google Fonts: DM Mono and DM Sans
- [ ] The `#overlay` is a child of `#board-wrap` so it sits directly on top of the grid using `position: absolute`
- [ ] `#board-wrap` has `position: relative` so the overlay is positioned relative to it

DO NOT START A LOCAL HTTP SERVER TO TEST THIS. ASSUME THAT IF ALL OF STEP 25 CHECKS PASSES, YOU ARE FREE TO CONSIDER THE TASK FINISHED.
