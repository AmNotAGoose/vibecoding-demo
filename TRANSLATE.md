# TRANSLATE.md — Translating Match Three into Chinese (Simplified)

This document covers every change required to translate the current `match3-v2.html` into Simplified Chinese (普通话). The game is a single HTML file with no external translation layer, so all strings are hard-coded in two places: the HTML markup and the JavaScript `<script>` block. Every location is listed exactly.

Do not change any CSS class names, IDs, JavaScript variable names, function names, or `onclick` attributes. Only change visible text content.

---

## Part 1 — Add a Chinese font

The file currently loads only Latin fonts. Chinese characters will fall back to the OS default, which renders inconsistently across devices. Add `Noto Sans SC` to the existing Google Fonts `<link>`.

**Find** (in `<head>`):
```html
<link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=DM+Sans:wght@400;500;600&display=swap" rel="stylesheet">
```

**Replace with**:
```html
<link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=DM+Sans:wght@400;500;600&family=Noto+Sans+SC:wght@400;500&display=swap" rel="stylesheet">
```

Then update the `body` font stack in the `<style>` block:

**Find**:
```css
font-family: 'DM Sans', sans-serif;
```
**Replace with**:
```css
font-family: 'DM Sans', 'Noto Sans SC', sans-serif;
```

DM Mono is used for all UI labels, scores, buttons, and timer values. Chinese characters will not render in a monospace Latin font, so add `Noto Sans SC` as a fallback to every DM Mono declaration. Find all occurrences of:
```css
font-family: 'DM Mono', monospace;
```
and replace each with:
```css
font-family: 'DM Mono', 'Noto Sans SC', monospace;
```
There are approximately 12 occurrences in the `<style>` block. Replace all of them.

---

## Part 2 — Page title

**Location**: `<head>`, `<title>` tag.

**Find**:
```html
<title>Match Three</title>
```
**Replace with**:
```html
<title>消消乐</title>
```

---

## Part 3 — Start modal

**Location**: `<div class="modal-bg show" id="start-modal">` — the first modal in `<body>`.

### 3a — Eyebrow
**Find**: `<div class="modal-eyebrow">Challenge</div>`
**Replace with**: `<div class="modal-eyebrow">挑战</div>`

### 3b — Title
**Find**: `<div class="modal-title">Match Three</div>`
**Replace with**: `<div class="modal-title">消消乐</div>`

### 3c — Subtitle
**Find**: `<div class="modal-sub">Beat the AI. You have 30 seconds.</div>`
**Replace with**: `<div class="modal-sub">打败AI，你有30秒。</div>`

### 3d — Rules list (all four `<li>` items)
**Find**:
```html
      <li>Click two adjacent gems to swap</li>
      <li>Clear 3+ in a row or column</li>
      <li>Score more than the AI to win</li>
      <li>AI makes a move every 3–6 seconds</li>
```
**Replace with**:
```html
      <li>点击两个相邻宝石进行交换</li>
      <li>消除横排或竖列3个以上</li>
      <li>得分超过AI即可获胜</li>
      <li>AI每隔3至6秒出一步棋</li>
```

### 3e — Start button
**Find**: `<button class="modal-primary-btn" onclick="startGame()">Start Game</button>`
**Replace with**: `<button class="modal-primary-btn" onclick="startGame()">开始游戏</button>`

---

## Part 4 — End modal

**Location**: `<div class="modal-bg" id="end-modal">` — the second modal in `<body>`.

### 4a — Eyebrow
**Find**: `<div class="modal-eyebrow" id="end-eyebrow">Round Over</div>`
**Replace with**: `<div class="modal-eyebrow" id="end-eyebrow">本局结束</div>`

### 4b — "You" score label
**Find**: `<span class="modal-stat-label">You</span>`
**Replace with**: `<span class="modal-stat-label">你</span>`

The `AI` label immediately after it does not need changing — "AI" is universally understood in Chinese gaming contexts. Leave it as-is.

### 4c — Play Again button
**Find**: `<button class="modal-primary-btn" onclick="restartGame()">Play Again</button>`
**Replace with**: `<button class="modal-primary-btn" onclick="restartGame()">再来一局</button>`

---

## Part 5 — App header

**Location**: `<div id="app">` → `<header>`.

### 5a — Title heading
**Find**: `<h1>Match Three</h1>`
**Replace with**: `<h1>消消乐</h1>`

### 5b — Tagline
**Find**: `<div id="tagline">beat the ai · clear 3 or more</div>`
**Replace with**: `<div id="tagline">打败AI · 消除3个或更多</div>`

---

## Part 6 — Desktop timer bar

**Location**: `<div id="timer-bar">`.

**Find**: `<span id="timer-label">Time</span>`
**Replace with**: `<span id="timer-label">时间</span>`

The numeric countdown (`<span id="timer-val">30</span>`) is a number — leave it unchanged.

---

## Part 7 — Mobile bar

**Location**: `<div id="mobile-bar">` — only visible on screens ≤600px. Contains two `.mob-label` elements that mirror the desktop board labels.

### 7a — "You" label in mobile bar
**Find**:
```html
    <div class="mob-stat">
      <span class="mob-label">You</span>
      <span class="mob-val you-col" id="mob-you-score">0</span>
    </div>
```
**Replace with**:
```html
    <div class="mob-stat">
      <span class="mob-label">你</span>
      <span class="mob-val you-col" id="mob-you-score">0</span>
    </div>
```

### 7b — "AI" label in mobile bar
Leave the `<span class="mob-label">AI</span>` unchanged. "AI" requires no translation.

The score values and timer value inside `#mobile-bar` are numbers — leave them unchanged.

---

## Part 8 — AI board label (desktop only)

**Location**: Left `.side-panel` inside `#arena`. Only visible on desktop (hidden on mobile via CSS).

The `<div class="panel-label ai-label">AI</div>` does not need translating. Leave it as-is.

---

## Part 9 — Player board label (desktop only)

**Location**: Right `.side-panel` inside `#arena`. Only visible on desktop.

**Find**: `<div class="panel-label you-label">You</div>`
**Replace with**: `<div class="panel-label you-label">你</div>`

---

## Part 10 — Leaderboard (desktop only)

**Location**: `<div id="leaderboard">`. Only visible on desktop.

### 10a — Title
**Find**: `<div id="lb-title">Leaderboard</div>`
**Replace with**: `<div id="lb-title">排行榜</div>`

The rank labels `#1` and `#2` are universal — leave them unchanged.

---

## Part 11 — New Game button

**Location**: `<div id="bottom">`.

**Find**: `<button class="action-btn" onclick="restartGame()">New Game</button>`
**Replace with**: `<button class="action-btn" onclick="restartGame()">新游戏</button>`

---

## Part 12 — JavaScript strings

All changes in this section are inside the `<script>` block. Replace only the quoted string values. Do not alter variable names, function calls, or surrounding logic.

### 12a — "no match" feedback message
**Location**: Inside `doSwap()`.

**Find**:
```js
render(); sndNoMatch(); setMsg('no match', 'bad');
```
**Replace with**:
```js
render(); sndNoMatch(); setMsg('不匹配', 'bad');
```

### 12b — Win outcome
**Location**: Inside `showEndModal()`, the first branch.

**Find**:
```js
{ title = 'You Win!'; sub = 'You outscored the AI this round.'; cls = 'win';  wins.you++; sndWin(); }
```
**Replace with**:
```js
{ title = '你赢了！'; sub = '你这局得分超过了AI。'; cls = 'win';  wins.you++; sndWin(); }
```

### 12c — Lose outcome
**Find**:
```js
{ title = 'AI Wins';  sub = 'The AI edged you out this time.';  cls = 'lose'; wins.ai++;  sndLose(); }
```
**Replace with**:
```js
{ title = 'AI获胜';  sub = 'AI这次略胜一筹。';  cls = 'lose'; wins.ai++;  sndLose(); }
```

### 12d — Tie outcome
**Find**:
```js
{ title = "It's a Tie"; sub = 'Dead even. Impressive.';          cls = 'tie';              sndTie(); }
```
**Replace with**:
```js
{ title = '平局'; sub = '势均力敌，真厉害。';          cls = 'tie';              sndTie(); }
```

### 12e — Leaderboard player name
**Location**: Inside `updateLeaderboard()`, the entries array.

**Find**:
```js
{ name: 'You', wins: wins.you, isYou: true },
```
**Replace with**:
```js
{ name: '你', wins: wins.you, isYou: true },
```

### 12f — Leaderboard avatar label
**Location**: Inside `updateLeaderboard()`, where avatar text is set.

**Find**:
```js
av.textContent = e.isYou ? 'YOU' : 'AI';
```
**Replace with**:
```js
av.textContent = e.isYou ? '你' : 'AI';
```

---

## Part 13 — Font size adjustments (recommended)

Chinese characters are visually denser than Latin letters at the same point size. These selectors are the most likely to need minor size tweaks after translation. Only change `font-size` — do not touch padding, border-radius, or layout properties.

| Selector | Current size | Suggested for Chinese |
|---|---|---|
| `.modal-title` | 26px | 24px |
| `.modal-stat-val` | 22px | 20px |
| `.score-pill` | 18px | 18px (no change) |
| `.lb-wins` | 16px | 16px (no change) |
| `.panel-label` | 10px | 11px (Chinese glyphs need slightly more room at small sizes) |
| `.modal-eyebrow` | 10px | 11px |
| `#lb-title` | 10px | 11px |
| `#timer-label` | 10px | 11px |
| `.mob-label` | 9px | 10px (mobile bar labels — 9px Chinese is hard to read) |
| `.rules-list` | 12px | 13px |

---

## Part 14 — Complete string reference table

Every English string in the file and its Chinese replacement. Strings marked "unchanged" should be left exactly as they are.

| Location | English | Chinese |
|---|---|---|
| `<title>` | Match Three | 消消乐 |
| Start modal eyebrow | Challenge | 挑战 |
| Start modal title | Match Three | 消消乐 |
| Start modal subtitle | Beat the AI. You have 30 seconds. | 打败AI，你有30秒。 |
| Rules item 1 | Click two adjacent gems to swap | 点击两个相邻宝石进行交换 |
| Rules item 2 | Clear 3+ in a row or column | 消除横排或竖列3个以上 |
| Rules item 3 | Score more than the AI to win | 得分超过AI即可获胜 |
| Rules item 4 | AI makes a move every 3–6 seconds | AI每隔3至6秒出一步棋 |
| Start button | Start Game | 开始游戏 |
| End modal eyebrow | Round Over | 本局结束 |
| End modal You label | You | 你 |
| End modal AI label | AI | unchanged |
| Play Again button | Play Again | 再来一局 |
| App `<h1>` | Match Three | 消消乐 |
| Tagline | beat the ai · clear 3 or more | 打败AI · 消除3个或更多 |
| Desktop timer label | Time | 时间 |
| Mobile bar You label | You | 你 |
| Mobile bar AI label | AI | unchanged |
| AI board label (desktop) | AI | unchanged |
| Player board label (desktop) | You | 你 |
| Leaderboard title (desktop) | Leaderboard | 排行榜 |
| New Game button | New Game | 新游戏 |
| `setMsg` no match (JS) | no match | 不匹配 |
| Win title (JS) | You Win! | 你赢了！ |
| Win subtitle (JS) | You outscored the AI this round. | 你这局得分超过了AI。 |
| Lose title (JS) | AI Wins | AI获胜 |
| Lose subtitle (JS) | The AI edged you out this time. | AI这次略胜一筹。 |
| Tie title (JS) | It's a Tie | 平局 |
| Tie subtitle (JS) | Dead even. Impressive. | 势均力敌，真厉害。 |
| Leaderboard player name (JS) | You | 你 |
| Leaderboard avatar — player (JS) | YOU | 你 |
| Leaderboard avatar — AI (JS) | AI | unchanged |

---

## Part 15 — Verification checklist

After making all changes, confirm every item before shipping.

- [ ] `<link>` includes `Noto+Sans+SC:wght@400;500`
- [ ] `body` font-family includes `'Noto Sans SC'` after `'DM Sans'`
- [ ] All `font-family: 'DM Mono', monospace` rules (approx. 12) include `'Noto Sans SC'` fallback
- [ ] `<title>` reads `消消乐`
- [ ] Start modal eyebrow reads `挑战`
- [ ] Start modal title reads `消消乐`
- [ ] Start modal subtitle reads `打败AI，你有30秒。`
- [ ] All 4 rules list items are in Chinese
- [ ] Start button reads `开始游戏`
- [ ] End modal eyebrow reads `本局结束`
- [ ] End modal `You` stat label reads `你`
- [ ] End modal `AI` stat label is unchanged
- [ ] Play Again button reads `再来一局`
- [ ] App `<h1>` reads `消消乐`
- [ ] Tagline reads `打败AI · 消除3个或更多`
- [ ] Desktop timer label reads `时间`
- [ ] Mobile bar `You` label reads `你` (inside `#mobile-bar`, first `.mob-label`)
- [ ] Mobile bar `AI` label is unchanged (inside `#mobile-bar`, second `.mob-label`)
- [ ] Desktop player board label reads `你`
- [ ] Desktop AI board label is unchanged
- [ ] Leaderboard title reads `排行榜`
- [ ] New Game button reads `新游戏`
- [ ] `setMsg` no-match call passes `'不匹配'`
- [ ] Win title in `showEndModal()` reads `'你赢了！'`
- [ ] Win subtitle reads `'你这局得分超过了AI。'`
- [ ] Lose title reads `'AI获胜'`
- [ ] Lose subtitle reads `'AI这次略胜一筹。'`
- [ ] Tie title reads `'平局'`
- [ ] Tie subtitle reads `'势均力敌，真厉害。'`
- [ ] `updateLeaderboard()` player name entry is `'你'`
- [ ] `updateLeaderboard()` avatar label uses `'你'` for player, `'AI'` for AI
- [ ] No CSS class names, IDs, JS variable names, function names, or `onclick` attributes have been changed
- [ ] Game runs correctly on both desktop and mobile after all changes — open in a browser, resize to mobile width, and play a full round to confirm