# 🏸 จัดก๊วนแบดมินตัน - Badminton Player Grouping App

## Project Overview

This is a **Thai-language, single-file web application** for managing badminton player groups and automatically creating fair match pairings. The entire app lives in `index.html` (HTML + CSS + JS, no build step, no dependencies).

**Live Demo:** https://pongpatsi56.github.io/gwan-bad-new/

---

## Purpose & Features

### Core Purpose
- **Group Management**: Add and manage badminton players with gender (M/F)
- **Fair Pairing**: Automatically pick 4 players and form 2 teams each round
- **Defender System**: Winners stay on court to defend against new challengers
- **Play Balance**: Track `gamesPlayed` and `waitCount` so everyone gets equal time
- **Score Tracking**: Real-time per-match score with undo and court-side switching
- **Statistics & History**: Wins, wait count, win %, full match log

---

## Game Flow

```
Setup → Start Game
          ↓
    genFirstMatch()          ← fair draw from all players
          ↓
   [Match in progress]       ← score tracking, side switching
          ↓
    recordWin(team)
          ↓
    defenderWon AND streak >= MAX_STREAK (2)?
      YES → genFirstMatch()  ← rotate out; full fresh fair draw
      NO  → genNextMatch()   ← winners stay, pick best challengers from pool
          ↓
   [Repeat indefinitely]
```

---

## Algorithm Details

### Player Selection for First Match (`genFirstMatch`)

Scores every player:
```
score = waitCount × 3 + max(0, avgGamesPlayed − gamesPlayed) + random(0, 0.5)
```
Top 4 play, the rest wait. `waitCount` is reset to 0 for the 4 who get to play.

### Challenger Selection (`pickChallengers`)

Pool = losers + previous waiting players. Tries every pair combination from the pool and picks the highest-scoring pair via `scorePair()`.

### Pair Scoring (`scorePair`)

```
score = (a.waitCount × 3 + max(0, avg − a.gamesPlayed))
      + (b.waitCount × 3 + max(0, avg − b.gamesPlayed))
      + 4   if mixed gender (M+F)
      − 4   if they were recent partners
      − 5   if they already faced defenderIds[0] recently
      − 5   if they already faced defenderIds[1] recently   ← checks BOTH defenders
      + random(0, 2)                                        ← enough variance to avoid deterministic repeats
```

Both defenders are checked for rematch avoidance (bug fix: previously only `defenderIds[0]` was checked).

### Team Split Scoring (`scoreSplit` / `formPairs`)

Given 4 chosen players, tries all 3 possible splits into 2 teams:
- `[0,1] vs [2,3]`
- `[0,2] vs [1,3]`
- `[0,3] vs [1,2]`

Each split is scored: `+4` per mixed-gender pair, `−4` per repeated partner pair, `+random(0, 1.5)`.

### Champion Streak Rule

- `MAX_STREAK = 2` (hardcoded)
- If defenders win 2 times in a row → `rotateOut = true` → `genFirstMatch()` (fresh fair draw)
- Champions who rotate out have high `gamesPlayed` and low `waitCount` → naturally low priority in next draw

---

## Data Structure

### State Object `S`
```javascript
{
  players: [...],   // array of Player objects
  match: {...},     // current Match object (null before game starts)
  history: [...],   // array of HistoryRecord, newest first
  round: 0,         // current round number
  started: false    // whether game phase is active
}
```

### Player Object
```javascript
{
  id: "uuid",
  name: "Player Name",
  gender: "M" | "F",
  gamesPlayed: 0,
  wins: 0,
  waitCount: 0,
  lastPartnerIds: ["uuid", "uuid"],   // last 2 partners (array, not single value)
  lastOpponentIds: ["uuid", ...]      // last 4 opponents
}
```

### Match Object
```javascript
{
  team1: ["player_id1", "player_id2"],
  team2: ["player_id3", "player_id4"],
  waiting: ["player_id5", ...],
  isDefending: false,
  streak: 0,
  defenderSide: 1 | 2,   // only meaningful when isDefending=true
  score1: 0,              // score for team1
  score2: 0,              // score for team2
  courtSide: 1 | 2,       // 1 = team1 at bottom, 2 = team2 at bottom
  scoreHistory: [...]     // stack of {score1, score2} for undo
}
```

### History Record
```javascript
{
  round: 1,
  t1Names: ["Name1", "Name2"],
  t2Names: ["Name3", "Name4"],
  winner: 1 | 2,
  isDefending: false,
  defenderWon: false,
  oldStreak: 0,
  newStreak: 1,
  rotateOut: false,
  time: "14:30"
}
```

---

## UI & Screens

### Setup Screen
- Add player form: text input + M/F gender toggle
- Player list with colored gender badges (`badge-M` = blue solid, `badge-F` = pink solid)
- Player count summary: `"N คน (M3 F2) — พร้อมเล่นได้เลย!"`
- Start Game button (disabled until ≥ 4 players)
- Load mock data button (10 test players)

### Game Screen
- **Round pill** at top
- **Match card**: current teams with gender icons, streak banner, win buttons
  - Win recording shows a confirm modal before committing
- **Score card** with SVG badminton court:
  - Real court lines: outer boundary, singles alleys, long/short service lines, center line, net with posts
  - Background dark green (`#2e6b25`)
  - **team1 = red** (`#fecaca` score, `#fca5a5` name), **team2 = blue** (`#bfdbfe` score, `#93c5fd` name)
  - Colors follow the team when sides are swapped (not the position)
  - Side-swap button triggers a 3D flip animation (`rotateX` 0→90°→ swap content →−90°→0°)
  - Score buttons: `+1 ฝั่งบน`, `+1 ฝั่งล่าง`, `↩ ยกเลิก +1` (undo, disabled when no history), `รีเซ็ตคะแนน`
- **Waiting queue** with chip cards per player
- **Stats table** and **History log** below

### Modals
- `#confirm-modal` — used by `requestRecordWin()` to confirm match result before committing
- `#app-modal` — general-purpose confirm dialog used by `showConfirm()` (clearAll, load mock data)

---

## Key Functions Reference

| Function | Purpose |
|---|---|
| `genFirstMatch()` | Fair player selection + team formation for first round or after rotate-out |
| `genNextMatch(winnerIds, loserIds, waiting, streak, side)` | Defenders stay; pick best challengers from pool |
| `recordWin(team)` | Update all stats, history, decide next match type |
| `scorePair(a, b, defenderIds, avg)` | Score a potential challenger pair |
| `scoreSplit(split, ids)` | Score one of 3 possible team splits |
| `formPairs(ids)` | Pick best team split from 4 player IDs |
| `pickChallengers(pool, defenderIds)` | Find best 2-player pair from pool |
| `hasRecentPartner(a, b)` | Check if a and b partnered in last 2 rounds |
| `incrementScore(side)` | Add 1 to top/bottom side, saves to scoreHistory |
| `undoScore()` | Pop last score state from scoreHistory |
| `switchCourtSide()` | Flip courtSide 1↔2 with 3D animation |
| `requestRecordWin(team)` | Open confirm modal before recording win |
| `confirmRecordWin()` | Execute recordWin after modal confirmation |
| `showConfirm(msg, cb, title, confirmText, cancelText)` | General-purpose confirm dialog using #app-modal |
| `renderGame()` | Re-render full game screen (court, teams, stats, history) |
| `save()` / `load()` | Persist/restore `S` from localStorage key `'badminton-v3'` |

---

## Technical Stack

- **Single HTML file** — no build step, no npm, no external dependencies
- **Vanilla JS** — ES6+, no frameworks
- **CSS** — CSS variables, Grid, Flexbox, 3D transforms, SVG for court
- **localStorage** — key `'badminton-v3'`, JSON-serialized state
- **State** — single mutable object `S`, cloned via `cloneState()` for undo

---

## Notes for AI

1. **Single file** — all HTML, CSS, and JS are in `index.html`. No other source files.
2. **State is global** — `S` is the single source of truth. `prevState` holds one level of match-level undo (for undoing the entire last match result, separate from `scoreHistory` which is for per-point undo).
3. **`renderGame()` is called after every state change** — it re-renders the full game view from scratch.
4. **Court colors are team-based, not position-based** — red = team1 always, blue = team2 always; `isBottomTeam1` determines which color goes top/bottom.
5. **Gender uses M/F strings** — not symbols. `gender === 'M'` or `gender === 'F'`.
6. **`lastPartnerIds` is an array of up to 2 IDs** — not a single value (old docs said `lastPartnerId`).
7. **Algorithm balances** (in order of weight):
   - Play count fairness (waitCount × 3 dominates)
   - Games-below-average factor
   - Mixed gender preference (+4)
   - Partner/opponent repeat avoidance (−4/−5)
   - Random noise (0–2) to prevent identical outcomes each session
8. **Streak limit is 2** (`MAX_STREAK = 2`) — after 2 consecutive defending wins, rotate out via `genFirstMatch()`.
9. **Both defenders are checked for rematch avoidance** — `scorePair` penalizes challenger pairs that previously faced either `defenderIds[0]` or `defenderIds[1]`.
10. **Score undo is per-point** — `scoreHistory` stack in match object; separate from `prevState` which is whole-match undo.
