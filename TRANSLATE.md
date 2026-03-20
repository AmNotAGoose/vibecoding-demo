# TRANSLATE.md — Translating Match Three into Chinese (Simplified)

This document covers every change required to translate `match3-v2.html` from English into Simplified Chinese (普通话). The game is a single HTML file with no external translation layer, so all strings are hard-coded. Every location is listed exactly — element type, ID or class, and the precise string to replace.

Do not change any CSS class names, IDs, JavaScript variable names, or function names. Only change the visible text content shown to the user.

---

## Part 1 — Add a Chinese font

The current file loads only Latin fonts. Chinese characters will fall back to the system default, which may look inconsistent. Add Noto Sans SC (Google Fonts) to the existing `<link>` tag.

**Find** (in `<head>`):
```html
<link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=DM+Sans:wght@400;500;600&display=swap" rel="stylesheet">
```

**Replace with**:
```html
<link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=DM+Sans:wght@400;500;600&family=Noto+Sans+SC:wght@400;500&display=swap" rel="stylesheet">
```

Then update the `body` font stack in the `<style>` block to include the Chinese font as a fallback.

**Find**:
```css
font-family: 'DM Sans', sans-serif;
```

**Replace with**:
```css
font-family: 'DM Sans', 'Noto Sans SC', sans-serif;
```

DM Mono is used for all UI labels and buttons. Chinese characters will not render correctly in a monospace Latin font, so add Noto Sans SC as a fallback there too. Find every occurrence of:
```css
font-family: 'DM Mono', monospace;
```
and replace with:
```css
font-family: 'DM Mono', 'Noto Sans SC', monospace;
```
There are approximately 12 occurrences in the `<style>` block. Replace all of them.

---

## Part 2 — Page title

**Location**: `<head>`, the `<title>` tag.

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

**Location**: `<div class="modal-bg show" id="start-modal">` — the first modal block in `<body>`.

### 3a — Eyebrow label
**Find**:
```html
<div class="modal-eyebrow">Challenge</div>
```
**Replace with**:
```html
<div class="modal-eyebrow">挑战</div>
```

### 3b — Game title
**Find**:
```html
<div class="modal-title">Match Three</div>
```
**Replace with**:
```html
<div class="modal-title">消消乐</div>
```

### 3c — Subtitle
**Find**:
```html
<div class="modal-sub">Beat the AI. You have 30 seconds.</div>
```
**Replace with**:
```html
<div class="modal-sub">打败AI，你有30秒。</div>
```

### 3d — Rules list (all four items)
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
**Find**:
```html
<button class="modal-primary-btn" onclick="startGame()">Start Game</button>
```
**Replace with**:
```html
<button class="modal-primary-btn" onclick="startGame()">开始游戏</button>
```

---

## Part 4 — End modal

**Location**: `<div class="modal-bg" id="end-modal">` — the second modal block in `<body>`.

### 4a — Eyebrow label
**Find**:
```html
<div class="modal-eyebrow" id="end-eyebrow">Round Over</div>
```
**Replace with**:
```html
<div class="modal-eyebrow" id="end-eyebrow">本局结束</div>
```

### 4b — "You" score label inside the end modal score panel
**Find**:
```html
        <span class="modal-stat-label">You</span>
```
**Replace with**:
```html
        <span class="modal-stat-label">你</span>
```

The `AI` label immediately below it does not need to change — "AI" is universally understood in Chinese gaming contexts. Leave it as-is.

### 4c — Play again button
**Find**:
```html
<button class="modal-primary-btn" onclick="restartGame()">Play Again</button>
```
**Replace with**:
```html
<button class="modal-primary-btn" onclick="restartGame()">再来一局</button>
```

---

## Part 5 — Main app header

**Location**: `<div id="app">` → `<header>`.

### 5a — Game title heading
**Find**:
```html
<h1>Match Three</h1>
```
**Replace with**:
```html
<h1>消消乐</h1>
```

### 5b — Tagline
**Find**:
```html
<div id="tagline">beat the ai · clear 3 or more</div>
```
**Replace with**:
```html
<div id="tagline">打败AI · 消除3个或更多</div>
```

---

## Part 6 — Timer bar

**Location**: `<div id="timer-bar">`.

**Find**:
```html
<span id="timer-label">Time</span>
```
**Replace with**:
```html
<span id="timer-label">时间</span>
```

---

## Part 7 — AI board label

**Location**: Left `.side-panel` inside `#arena`.

**Find**:
```html
<div class="panel-label ai-label">AI</div>
```

Leave this as `AI` — it is a universally recognised abbreviation in Chinese digital contexts and does not need translation.

---

## Part 8 — Player board label

**Location**: Right `.side-panel` inside `#arena`.

**Find**:
```html
<div class="panel-label you-label">You</div>
```
**Replace with**:
```html
<div class="panel-label you-label">你</div>
```

---

## Part 9 — Leaderboard

**Location**: `<div id="leaderboard">`.

### 9a — Title
**Find**:
```html
<div id="lb-title">Leaderboard</div>
```
**Replace with**:
```html
<div id="lb-title">排行榜</div>
```

The rank labels `#1` and `#2` are numeric and universal — leave them unchanged.

---

## Part 10 — New Game button

**Location**: `<div id="bottom">`.

**Find**:
```html
<button class="action-btn" onclick="restartGame()">New Game</button>
```
**Replace with**:
```html
<button class="action-btn" onclick="restartGame()">新游戏</button>
```

---

## Part 11 — JavaScript strings

All of these are inside the `<script>` block. Find each string precisely as written and replace only the quoted text value. Do not alter variable names, function calls, or surrounding code.

### 11a — "no match" feedback message
**Location**: Inside `doSwap()`.

**Find**:
```js
render(); sndNoMatch(); setMsg('no match', 'bad');
```
**Replace with**:
```js
render(); sndNoMatch(); setMsg('不匹配', 'bad');
```

### 11b — Win title and subtitle
**Location**: Inside `showEndModal()`.

**Find**:
```js
title = 'You Win!'; sub = 'You outscored the AI this round.'; cls = 'win';
```
**Replace with**:
```js
title = '你赢了！'; sub = '你这局得分超过了AI。'; cls = 'win';
```

### 11c — Lose title and subtitle
**Find**:
```js
title = 'AI Wins'; sub = 'The AI edged you out this time.'; cls = 'lose';
```
**Replace with**:
```js
title = 'AI获胜'; sub = 'AI这次略胜一筹。'; cls = 'lose';
```

### 11d — Tie title and subtitle
**Find**:
```js
title = "It's a Tie"; sub = 'Dead even. Impressive.'; cls = 'tie'; sndTie();
```
**Replace with**:
```js
title = '平局'; sub = '势均力敌，真厉害。'; cls = 'tie'; sndTie();
```

### 11e — Leaderboard player name
**Location**: Inside `updateLeaderboard()`, in the entries array.

**Find**:
```js
{ name: 'You', wins: wins.you, isYou: true },
```
**Replace with**:
```js
{ name: '你', wins: wins.you, isYou: true },
```

### 11f — Leaderboard avatar label
**Location**: Inside `updateLeaderboard()`, where avatar text content is set.

**Find**:
```js
av.textContent  = e.isYou ? 'YOU' : 'AI';
```
**Replace with**:
```js
av.textContent  = e.isYou ? '你' : 'AI';
```

---

## Part 12 — Font size adjustments (recommended)

Chinese characters are visually denser than Latin letters at the same point size. Some labels may feel cramped or oversized after translation. These are the elements most likely to need fine-tuning:

| Element / Selector | Current size | Recommended Chinese size |
|---|---|---|
| `.modal-title` | 28px | 26px |
| `.stat-value` | 22px | 20px |
| `.modal-stat-val` | 22px | 20px |
| `.lb-wins` | 18px | 18px (no change needed) |
| `.panel-label` | 10px | 11px (Chinese glyphs at 10px are hard to read) |
| `.modal-eyebrow` | 10px | 11px |
| `#lb-title` | 10px | 11px |
| `.stat-label` | 10px | 11px |
| `.rules-list` | 12px | 13px |

Adjust only the `font-size` values for the affected selectors. Do not change padding, border-radius, gap, or any layout properties — the game layout is designed to be tight and these adjustments are small enough not to break it.

---

## Part 13 — Complete string reference table

A single consolidated list of every English string and its Chinese replacement, for fast verification after editing.

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
| End modal AI label | AI | AI（不变） |
| Play Again button | Play Again | 再来一局 |
| App `<h1>` | Match Three | 消消乐 |
| Tagline | beat the ai · clear 3 or more | 打败AI · 消除3个或更多 |
| Timer label | Time | 时间 |
| AI board label | AI | AI（不变） |
| Player board label | You | 你 |
| Leaderboard title | Leaderboard | 排行榜 |
| New Game button | New Game | 新游戏 |
| `setMsg` no match (JS) | no match | 不匹配 |
| Win title (JS) | You Win! | 你赢了！ |
| Win subtitle (JS) | You outscored the AI this round. | 你这局得分超过了AI。 |
| Lose title (JS) | AI Wins | AI获胜 |
| Lose subtitle (JS) | The AI edged you out this time. | AI这次略胜一筹。 |
| Tie title (JS) | It's a Tie | 平局 |
| Tie subtitle (JS) | Dead even. Impressive. | 势均力敌，真厉害。 |
| Leaderboard player name (JS) | You | 你 |
| Avatar label — player (JS) | YOU | 你 |
| Avatar label — AI (JS) | AI | AI（不变） |

---

## Part 14 — Verification checklist

After making all changes, confirm every item before shipping:

- [ ] `<link>` includes `Noto+Sans+SC:wght@400;500`
- [ ] `body` font-family includes `'Noto Sans SC'` as fallback after `'DM Sans'`
- [ ] All ~12 occurrences of `font-family: 'DM Mono', monospace` include `'Noto Sans SC'` as fallback
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
- [ ] Timer label reads `时间`
- [ ] Player board label reads `你`
- [ ] Leaderboard title reads `排行榜`
- [ ] New Game button reads `新游戏`
- [ ] `setMsg` no-match call passes `'不匹配'`
- [ ] Win title and subtitle are in Chinese in `showEndModal()`
- [ ] Lose title and subtitle are in Chinese in `showEndModal()`
- [ ] Tie title and subtitle are in Chinese in `showEndModal()`
- [ ] `updateLeaderboard()` player name entry uses `'你'`
- [ ] `updateLeaderboard()` avatar label uses `'你'` for player and `'AI'` for AI
- [ ] No CSS class names, IDs, JS variable names, or function names have been altered
- [ ] Game runs correctly after all changes — open in browser and play a full round to confirm