# Audio & Visual Update — Design Spec

**Date:** 2026-05-05
**Status:** Approved

## Overview

Add Web Audio API 8-bit sounds (background music + catch/miss SFX) and update visual effects: catch emoji → 💧, new 💦 splash on miss. All changes stay within single `index.html`, no external dependencies.

---

## Audio System

**Approach:** Web Audio API — procedural, no files, no CDN.

**AudioContext:** Created once on first user gesture (keydown/click/touchstart) to satisfy browser autoplay policy. Stored as module-level `let audioCtx = null`. All sound functions create it lazily if null.

**Sounds:**

| Sound | Trigger | Notes | Wave | Duration |
|-------|---------|-------|------|----------|
| `playCatch()` | poop caught | C5 (523Hz) → E5 (659Hz) | square | 80ms each note |
| `playMiss()` | poop missed | G4 (392Hz) → D4 (294Hz) | square | 100ms each note |
| `playMusic()` | game starts | 8-note loop: C5-E5-G5-E5-C5-G4-E5-C5 | triangle | 150ms/note |

**Music loop:** Recursive `setTimeout` sequences notes. Gain ~0.08 (quiet under SFX). Tracks current timeout ID in `musicTimeout`. `stopMusic()` clears timeout. `startMusic()` called in `startGame()`. `stopMusic()` called on transition to `gameover`.

**Note helper:**
```javascript
function playNote(freq, type, duration, gain, startTime) {
  const osc = audioCtx.createOscillator();
  const gainNode = audioCtx.createGain();
  osc.connect(gainNode);
  gainNode.connect(audioCtx.destination);
  osc.type = type;
  osc.frequency.value = freq;
  gainNode.gain.setValueAtTime(gain, startTime);
  gainNode.gain.exponentialRampToValueAtTime(0.001, startTime + duration);
  osc.start(startTime);
  osc.stop(startTime + duration);
}
```

---

## Visual Changes

**Catch effect:** Change drawn emoji from `💥` to `💧`. Same `effects` array, same 300ms timer, same position (`poop.x`, `canvas.height - 80`). One-line change in `drawPlaying`.

**Miss effect:** New `missEffects` array `{ x, y, timer }`. On miss, push `{ x: poop.x, y: canvas.height - 60, timer: 400 }`. Draw `💦` at each entry. `missEffects` filtered/ticked in `updateEffects(dt)` same as `effects`. Reset to `[]` in `startGame()`.

---

## Integration Points

| Change | Location |
|--------|----------|
| `let audioCtx`, `let musicTimeout` | top-level vars |
| `initAudio()`, `playNote()`, `playCatch()`, `playMiss()`, `playMusic()`, `stopMusic()`, `startMusic()` | new functions before input listeners |
| Call `initAudio()` | inside keydown, click, touchstart handlers (before state check) |
| Call `startMusic()` | inside `startGame()` |
| Call `stopMusic()` | inside `updateEffects` when `lives <= 0` triggers gameover |
| `let missEffects = []` | game state vars |
| Push to `missEffects` | miss branch in `updatePoops` |
| Tick/filter `missEffects` | `updateEffects(dt)` |
| Reset `missEffects` | `startGame()` |
| Draw `💦` | `drawPlaying()` |
| `💥` → `💧` | `drawPlaying()` effects forEach |

---

## Out of Scope

- Volume controls
- Mute button
- Sound on start screen
- Different music per difficulty level
