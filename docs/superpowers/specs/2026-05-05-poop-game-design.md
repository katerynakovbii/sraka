# Poop Catcher — Design Spec

**Date:** 2026-05-05  
**Status:** Approved

## Overview

Browser-based arcade game. Poop 💩 falls from sky. Player controls toilet paper roll 🧻 along the bottom of the screen to catch it. Miss 3 — game over. Speed increases as score climbs. Single self-contained `index.html`, no dependencies.

---

## Architecture

**Delivery:** Single `index.html` file. Vanilla JS + Canvas 2D. No build tool, no dependencies.

**Structure:** Three JS objects in one `<script>` block:

- `Game` — owns state (score, lives, speed multiplier, active poops, clouds), drives the game loop via `requestAnimationFrame`, manages state transitions, handles spawning
- `Poop` — data object `{ x, y, speed }`, falls straight down each frame, despawns on catch or miss
- `Player` — data object `{ x, width }`, moves horizontally along canvas bottom

**Canvas:** Full-screen, resizes on window resize. Dark night-sky background (#1a1a2e).

---

## Game Rules

| Rule | Detail |
|------|--------|
| Lives | 3. Displayed as ❤️❤️❤️ top-right |
| Miss | Poop exits bottom without catch → lose 1 life → screen flashes red |
| Catch | Bounding-box: poop bottom edge overlaps player top edge within player width |
| Score | +1 per catch, top-left |
| Speed scaling | Every 10 catches: fall speed increases by 15%, spawn interval decreases by 10% |
| Max simultaneous | 1 poop early game → up to 3 at high scores |
| Game over | 0 lives → gameover screen |

---

## Feel & Effects

- **Catch:** brief 💥 emoji flash at catch point for 300ms
- **Miss:** canvas border flashes red for 400ms
- **Score bump:** score text does a quick scale-up bounce on increment
- **Background:** 3–5 ☁️ clouds drift slowly left-to-right across sky (loop)
- **Player emoji:** 🧻, ~60px wide, sits at canvas bottom with small padding

---

## Controls

| Input | Action |
|-------|--------|
| ← / A | Move player left |
| → / D | Move player right |
| Mouse move | Player X tracks cursor X directly |
| Touch drag | Player X tracks touch X directly |

Player speed for keyboard: 6px per frame.

---

## Screen States

```
start → playing → gameover → playing
```

### `start`
- Centered: title "💩 POOP CATCHER 🧻"
- Subtitle: "catch the poop before it hits the ground!"
- Prompt: "Press any key or tap to start"

### `playing`
- Score top-left
- Lives top-right
- Clouds drifting
- Poops falling
- Player at bottom

### `gameover`
- "GAME OVER" large centered
- Final score below
- "Play Again" — click or any key resets all state → `playing`

---

## File Structure

```
index.html        # entire game — HTML + CSS + JS
docs/
  superpowers/
    specs/
      2026-05-05-poop-game-design.md
```

---

## Out of Scope

- Sound / audio
- High score persistence (localStorage)
- Mobile-specific UI (score/lives scaling)
- Power-ups
