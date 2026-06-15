# 🏸 จัดก๊วนแบดมินตัน - Badminton Player Grouping App

## Project Overview

This is a **Thai-language web application** for managing badminton player groups and automatically creating fair match pairings. The app ensures every player gets to play equally while maintaining game competition balance through a sophisticated pairing algorithm.

**Live Demo:** https://pongpatsi56.github.io/gwan-bad-new/

---

## Purpose & Features

### Core Purpose
- **Group Management**: Add and manage badminton players with gender information
- **Fair Pairing**: Automatically pair 4 players into 2 teams for each match
- **Defender System**: Winners stay to defend their position against new challengers
- **Play Balance**: Track who played how many games and who's still waiting, ensuring everyone plays equally
- **Statistics Tracking**: Record wins, losses, play count, win percentage for each player
- **History Recording**: Log all matches with results, streaks, and rotation-outs

### Key Features
1. **Setup Phase**
   - Add players with names and gender (Male/Female)
   - Minimum 4 players required to start a game
   - Test data button for quick testing

2. **Game Phase**
   - Real-time match display with current round number
   - Two game modes:
     - **Normal Mode** (First match): Fair random selection of 4 players → best pairing
     - **Defending Mode** (Subsequent matches): Winners stay as defending team vs best challengers
   - Waiting queue showing players not in current match
   - Win buttons for recording match results

3. **Rotation System**
   - **Streak Limit**: If a defending team wins 2 consecutive matches, they rotate out and a fresh fair draw happens
   - This prevents the same players from dominating and ensures everyone gets fair chances

4. **Statistics**
   - Player stats table showing: Games Played, Wins, Wait Count, Win %
   - Color-coded by team/status (Defender, Challenger, Waiting)

5. **History Log**
   - Records all matches chronologically
   - Shows winners, losers, streaks, and rotation rotations
   - Displays match time stamps

6. **Data Persistence**
   - Automatically saves to browser's localStorage under key 'badminton-v3'
   - Data persists between sessions
   - Old 'badminton-v2' data can be automatically migrated

---

## How It Works

### Algorithm Details

#### 1. **Pair Scoring Function** (`scorePair`)
When selecting challengers, the algorithm scores each potential pair against the defending team:
- **Urgency Factor**: Players who waited longer or played fewer games get priority
  - `waitCount * 3 + max(0, avgGames - gamesPlayed)`
- **Mixed Doubles Bonus**: +4 points for pairing opposite genders
- **Partner Avoidance**: -4 points if they partnered last time
- **Rematch Avoidance**: -5 points if they faced the same defender pair last match
- Small random noise (0-0.4) for variety

#### 2. **First Match Generation** (`genFirstMatch`)
- Scores all players by urgency
- Selects top 4 players fairly
- Forms best pairing from these 4 players using `formPairs()`
- Remaining players go to waiting queue

#### 3. **Pair Formation** (`formPairs`)
For any 4 selected players, tries all 3 possible pairings:
- `[[0,1], [2,3]]` - First vs Last
- `[[0,2], [1,3]]` - Criss-cross
- `[[0,3], [1,2]]` - Cross
Picks the best based on gender mix and partner/opponent history.

#### 4. **Next Match Generation** (`genNextMatch`)
- Winners stay as defending team
- Pool = losers + waiting players
- Algorithm picks best challenger pair from pool
- Remaining pool becomes new waiting queue
- If defending team hits max streak (2 wins), call `genFirstMatch()` instead to rotate them out

---

## Technical Stack

### Frontend
- **HTML5** - Single-page application structure
- **CSS3** - Modern responsive design with:
  - CSS Grid for layout
  - CSS Variables for theming (green, blue, pink, amber, purple colors)
  - Smooth animations (fadeIn, pulse effects)
  - Mobile-responsive breakpoint (max-width: 420px)
  - Glass-morphism inspired cards with shadows

### JavaScript (Vanilla)
- **No frameworks** - Pure vanilla JS for maximum performance
- **State Management** - Single object `S` with:
  - `players[]` - Player list with stats
  - `match` - Current match data
  - `history[]` - All recorded matches
  - `round` - Current round number
- **LocalStorage API** - Client-side persistence
- **UUID Generation** - For unique player IDs (fallback to timestamp)

### Design
- **Color Scheme**: Green (primary), Blue (Team 1), Pink (Team 2), Amber (Defending), Purple (Challenger)
- **Typography**: System font stack (Segoe UI, system-ui, sans-serif)
- **Responsive**: Works on desktop and mobile (420px breakpoint)
- **Accessibility**: Good contrast, semantic HTML

---

## Data Structure

### Player Object
```javascript
{
  id: "uuid or timestamp",
  name: "Player Name",
  gender: "M" or "F",
  gamesPlayed: 0,
  wins: 0,
  waitCount: 0,
  lastPartnerId: "uuid or null",
  lastOpponentIds: ["uuid1", "uuid2"]
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
  defenderSide: 1 or 2  // only when isDefending=true
}
```

### History Record
```javascript
{
  round: 1,
  t1Names: ["Name1", "Name2"],
  t2Names: ["Name3", "Name4"],
  winner: 1 or 2,
  isDefending: false,
  defenderWon: false,
  oldStreak: 0,
  newStreak: 1,
  rotateOut: false,
  time: "14:30"
}
```

---

## Game Flow

1. **Setup** → Add 4+ players with gender
2. **Start Game** → First match created (fair 4-player draw + best pairing)
3. **Record Win** → Update stats, check if rotation needed
4. **Generate Next** → Either:
   - Fresh fair draw (if defenders hit 2-win limit) 🔄
   - Or: Defenders stay vs best challengers from pool ⚔️
5. **Repeat** → Continue indefinitely
6. **View Stats** → See all-time player statistics
7. **View History** → See past match results and streaks

---

## UI Screens

### Setup Screen
- Header: "🏸 จัดก๊วนแบดมินตัน"
- Tagline: "แชมป์อยู่ต่อ · จับคู่หลากหลาย · เล่นแฟร์ทุกคน"
- Add player form (name input + M/F toggle)
- Player list with gender badges
- Start Game button (disabled until 4+ players)

### Game Screen
- Round pill (e.g., "เกมที่ 1")
- Match card showing:
  - Current matchup with teams and genders
  - Streak banner if defending
  - Win buttons
- Waiting queue
- Tabs: Player Stats | Match History

### Interactive Elements
- **Add Form**: Input name + gender radio + Add button
- **Win Buttons**: Click to record which team won
- **Tab Navigation**: Switch between stats and history
- **Edit/Clear**: Edit players or clear all data

---

## Localization

- **Language**: Thai (ภาษาไทย)
- **UI Text Examples**:
  - "🏸 จัดก๊วนแบดมินตัน" = "Badminton Grouping"
  - "เกม" = "Game"
  - "เล่น" = "Play"
  - "รักษาแชมป์" = "Defend Champion"
  - "โค่นแชมป์" = "Overthrow Champion"
- **Gender**: 
  - "♂ ชาย" = Male
  - "♀ หญิง" = Female
- **Pair Types**:
  - "★ คู่ผสม" = Mixed Doubles
  - "ชายคู่" = Men's Doubles
  - "หญิงคู่" = Women's Doubles

---

## Usage Instructions

1. **Open the app** in any modern browser
2. **Enter player names** - minimum 4 required
3. **Select gender** for each player
4. **Click "เริ่มเกม" (Start Game)**
5. **See the first match** - 4 fairly selected players paired optimally
6. **Click winning team** - records result
7. **Next match** appears with defenders vs best challengers
8. **View stats** - see who's played most, won most, etc.
9. **View history** - see all past match results

---

## Project Files

- **index.html** - Complete single-file application (HTML + CSS + JavaScript)
- **README.md** - Simple link to live demo
- **PROJECT_SUMMARY.md** - This file, comprehensive documentation

---

## Browser Support

- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Any modern browser with:
  - ES6 JavaScript support
  - LocalStorage API
  - CSS Grid & Flexbox
  - CSS Custom Properties (variables)

---

## Notes for AI

When reading this codebase, understand:
1. The app is **purely client-side** (no server)
2. All data is stored in **browser localStorage**
3. The **pairing algorithm** prioritizes fairness over randomness
4. **Defending mode** is a game mechanic to make winners stay until they hit a streak limit
5. The UI uses **emoji heavily** for visual language (Thai-friendly)
6. All styling is **inline CSS** in one HTML file (no external dependencies)
7. The algorithm balances:
   - **Play count fairness** (equal games for everyone)
   - **Gender mixing** (prefer mixed doubles)
   - **Partner variety** (avoid repeating pairs)
   - **Competitive balance** (good matchups)
   - **Streak limit** (prevent domination, ensure rotation)

