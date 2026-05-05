# Audio & Visual Update Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Web Audio API 8-bit sounds (catch SFX, miss SFX, looping background music) and update visuals (catch emoji 💥→💧, new 💦 miss splash effect) in `index.html`.

**Architecture:** All changes in single `index.html`. AudioContext created lazily on first user gesture. Music runs as a recursive setTimeout note sequencer. Visual changes extend the existing `effects` pattern with a new `missEffects` array.

**Tech Stack:** Web Audio API (OscillatorNode, GainNode), vanilla JS, HTML5 Canvas

---

## File Structure

| File | Change |
|------|--------|
| `index.html` | Add audio vars + 6 functions, wire into game events, add missEffects, update draw |

---

### Task 1: Audio infrastructure — initAudio, playNote, playCatch, playMiss

**Files:**
- Modify: `index.html` (add after `// --- Game State ---` block, before `// --- Helpers ---`)

- [ ] **Step 1: Add audio state variables**

After `let flashTimer = 0;` (line 56), add:

```javascript
    // --- Audio ---
    let audioCtx = null;
    let musicTimeout = null;
```

- [ ] **Step 2: Add initAudio, playNote, playCatch, playMiss**

After the two audio variable lines, add:

```javascript
    function initAudio() {
      if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }

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
      osc.stop(startTime + duration + 0.01);
    }

    function playCatch() {
      if (!audioCtx) return;
      const t = audioCtx.currentTime;
      playNote(523, 'square', 0.08, 0.3, t);
      playNote(659, 'square', 0.08, 0.3, t + 0.08);
    }

    function playMiss() {
      if (!audioCtx) return;
      const t = audioCtx.currentTime;
      playNote(392, 'square', 0.1, 0.3, t);
      playNote(294, 'square', 0.1, 0.3, t + 0.1);
    }
```

- [ ] **Step 3: Verify structure (no browser test yet — just syntax check)**

Open browser console on `index.html`. Expected: no errors on load. Type `initAudio()` — no error. Type `playCatch()` — should hear two ascending beeps. Type `playMiss()` — two descending beeps.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Web Audio API infrastructure and SFX functions"
```

---

### Task 2: Background music — playMusic, stopMusic, startMusic

**Files:**
- Modify: `index.html` (add after `playMiss` function)

- [ ] **Step 1: Add playMusic, stopMusic, startMusic**

After `playMiss()`, add:

```javascript
    const MELODY = [523, 659, 784, 659, 523, 392, 659, 523];

    function playMusic(noteIndex) {
      if (!audioCtx) return;
      playNote(MELODY[noteIndex], 'triangle', 0.13, 0.08, audioCtx.currentTime);
      musicTimeout = setTimeout(() => {
        playMusic((noteIndex + 1) % MELODY.length);
      }, 150);
    }

    function stopMusic() {
      clearTimeout(musicTimeout);
      musicTimeout = null;
    }

    function startMusic() {
      stopMusic();
      if (!audioCtx) return;
      playMusic(0);
    }
```

- [ ] **Step 2: Verify in browser console**

Type `initAudio()` then `startMusic()` — looping 8-note chiptune melody plays. Type `stopMusic()` — music stops.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add background music sequencer"
```

---

### Task 3: Visual changes — 💥→💧 and missEffects 💦

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add missEffects variable**

After `let effects = [];` (currently line 54), add:

```javascript
    let missEffects = [];
```

- [ ] **Step 2: Push to missEffects on miss**

In `updatePoops`, find the miss branch:

```javascript
        if (p.y >= canvas.height + 50) {
          if (lives > 0) { lives--; flashTimer = 400; }
          return false;
        }
```

Replace with:

```javascript
        if (p.y >= canvas.height + 50) {
          if (lives > 0) { lives--; flashTimer = 400; }
          missEffects.push({ x: p.x, y: canvas.height - 60, timer: 400 });
          return false;
        }
```

- [ ] **Step 3: Tick and filter missEffects in updateEffects**

In `updateEffects`, after the `effects` filter line:

```javascript
      effects = effects.filter(e => { e.timer -= dt; return e.timer > 0; });
```

Add:

```javascript
      missEffects = missEffects.filter(e => { e.timer -= dt; return e.timer > 0; });
```

- [ ] **Step 4: Reset missEffects in startGame**

In `startGame()`, after `effects = [];`, add:

```javascript
      missEffects = [];
```

- [ ] **Step 5: Update drawPlaying — change 💥 to 💧 and draw 💦**

Find in `drawPlaying`:

```javascript
      effects.forEach(e => ctx.fillText('💥', e.x, e.y));
```

Replace with:

```javascript
      effects.forEach(e => ctx.fillText('💧', e.x, e.y));
      missEffects.forEach(e => ctx.fillText('💦', e.x, e.y));
```

- [ ] **Step 6: Verify in browser**

Start game, catch poop → 💧 appears. Let poop fall → 💦 appears at bottom. Both fade after ~300-400ms.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: update catch emoji to 💧, add 💦 miss splash effect"
```

---

### Task 4: Wire audio into game events

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Call initAudio in all user gesture handlers**

Find the keydown listener:

```javascript
    document.addEventListener('keydown', e => {
      keys[e.key] = true;
      if (state === 'start') startGame();
      else if (state === 'gameover') startGame();
    });
```

Replace with:

```javascript
    document.addEventListener('keydown', e => {
      keys[e.key] = true;
      initAudio();
      if (state === 'start') startGame();
      else if (state === 'gameover') startGame();
    });
```

Find the click listener:

```javascript
    canvas.addEventListener('click', () => {
      if (state === 'start') startGame();
      else if (state === 'gameover') startGame();
    });
```

Replace with:

```javascript
    canvas.addEventListener('click', () => {
      initAudio();
      if (state === 'start') startGame();
      else if (state === 'gameover') startGame();
    });
```

Find the touchstart listener:

```javascript
    canvas.addEventListener('touchstart', e => {
      e.preventDefault();
      if (state === 'start') startGame();
      else if (state === 'gameover') startGame();
    });
```

Replace with:

```javascript
    canvas.addEventListener('touchstart', e => {
      e.preventDefault();
      initAudio();
      if (state === 'start') startGame();
      else if (state === 'gameover') startGame();
    });
```

- [ ] **Step 2: Call startMusic in startGame**

In `startGame()`, after `state = 'playing';`, add:

```javascript
      startMusic();
```

- [ ] **Step 3: Call stopMusic when game over**

In `updateEffects`, find:

```javascript
      if (lives <= 0) state = 'gameover';
```

Replace with:

```javascript
      if (lives <= 0) { stopMusic(); state = 'gameover'; }
```

- [ ] **Step 4: Call playCatch on catch**

In `updatePoops`, find the catch branch:

```javascript
          score++;
          scoreBounce = 1;
          effects.push({ x: p.x, y: canvas.height - 80, timer: 300 });
          updateDifficulty();
```

Replace with:

```javascript
          score++;
          scoreBounce = 1;
          effects.push({ x: p.x, y: canvas.height - 80, timer: 300 });
          updateDifficulty();
          playCatch();
```

- [ ] **Step 5: Call playMiss on miss**

In `updatePoops`, find the miss branch:

```javascript
          if (lives > 0) { lives--; flashTimer = 400; }
          missEffects.push({ x: p.x, y: canvas.height - 60, timer: 400 });
```

Replace with:

```javascript
          if (lives > 0) { lives--; flashTimer = 400; playMiss(); }
          missEffects.push({ x: p.x, y: canvas.height - 60, timer: 400 });
```

- [ ] **Step 6: Full end-to-end verification**

Open `index.html`. Check each:

| Action | Expected |
|--------|----------|
| Press key / click to start | Music begins looping |
| Catch poop | 💧 appears + ascending 2-note beep |
| Miss poop | 💦 appears at bottom + descending 2-note beep + red flash |
| 3 misses | Music stops, GAME OVER screen |
| Press key to restart | Music restarts, all state reset |
| Music volume | Quiet (doesn't drown out SFX) |

- [ ] **Step 7: Commit and push**

```bash
git add index.html
git commit -m "feat: wire audio into game events — catch/miss SFX and music start/stop"
git push origin main
```

---

## Self-Review

- `initAudio` called before `startGame` in all 3 gesture handlers ✓
- `startMusic` calls `stopMusic` first (idempotent restart) ✓  
- `playMiss` only called when `lives > 0` (no sound on already-dead state) ✓
- `missEffects` reset in `startGame` ✓
- `MELODY` constant defined before `playMusic` uses it ✓
- All function names consistent across tasks ✓
