# Retro Arcade — Design & Development Guide

This guide defines the consistent style and technical practices for retro mobile games. Each game is a single HTML file (vanilla HTML/CSS/JS, no dependencies). Games are accessed through a shared menu.

---

## 1. Visual Identity

### CRT Retro Theme
All games and the menu follow a unified CRT phosphor aesthetic:

- **Background:** `radial-gradient(ellipse at center, #0d0d0d 0%, #000 80%)`
- **Scanline effect:** `::before` pseudo-element — `repeating-linear-gradient` every 2px, `rgba(0,0,0,0.08)`
- **Vignette:** `::after` pseudo-element — `radial-gradient(ellipse at center, transparent 55%, rgba(0,0,0,0.65) 100%)`
- Both: `position: fixed; inset: 0; pointer-events: none; z-index: 1000+`

### Color Palette (CSS Variables)
```css
:root {
  --g: #33ff33;          /* Primary color — green phosphor */
  --gd: #1a8c1a;         /* Dimmed green — borders, inactive */
  --bg: #070707;         /* Game area background */
  --glow: rgba(51,255,51,0.12);  /* Glow effect */
  --amber: #ffaa00;      /* Accent color — scores, numbers, active selections */
}
```

Colors stay the same across all games. Green shades can be used for variety within the game area:
`#33ff33, #33dd33, #00ffaa, #66ff66, #00cc66, #22ff88, #44ffcc`

### Typography
- **Font:** `'Press Start 2P', monospace` (Google Fonts)
- **Titles:** 14–22px, `letter-spacing: 4–5px`, titleGlow animation
- **Scores/values:** `--amber` color, `text-shadow: 0 0 4–5px var(--amber)`
- **Labels:** 5–7px, `opacity: 0.6`, `letter-spacing: 1–2px`
- **Hints:** 5px, `opacity: 0.3`

### Animations
```css
@keyframes titleGlow {
  0%, 100% { text-shadow: 0 0 2px #fff, 0 0 8px var(--g), 0 0 20px rgba(51,255,51,0.25) }
  50% { text-shadow: 0 0 4px #fff, 0 0 12px var(--g), 0 0 30px rgba(51,255,51,0.35) }
}
@keyframes blink {
  0%, 100% { opacity: 1 } 50% { opacity: 0 }
}
```
- Game title: `titleGlow 3s ease-in-out infinite`
- "Tap to start" hints: `blink 1.2s step-end infinite`
- **Overlay h1:** `color: #d0ffd0` — slightly lighter than the base green; combined with white inner glow, letters stay legible against the background glow

**Note:** Using `var(--g)` as a text-shadow on same-colored text creates a blurry glowing blob. White close-glow (`0 0 2px #fff`) sharpens the letterforms; outer layers use transparent green.

### Element Styles
- **Canvas/game area:** `border: 2px solid var(--gd)`, `box-shadow: 0 0 15px var(--glow), inset 0 0 25px rgba(0,0,0,0.4)`, `image-rendering: pixelated`
- **Small panels (preview etc.):** `border: 1px solid var(--gd)`, `box-shadow: 0 0 4px var(--glow)`
- **Overlay screens:** `background: rgba(0,0,0,0.88)`, centered flexbox, `z-index: 500`
- **Pause button:** In the topbar between left and right sections (not inside boardWrap), text always `PAUSE` (not `| |` or other symbols), `color: var(--gd)`, `border: 1px solid var(--gd)`, `background: none`, no position tricks

### Drawing Style (Canvas Blocks)
When the game draws grid blocks on the canvas:
```
- Filled block: 1px padding on each side
- Light highlight lines on top-left corner (2px wide)
- Dark shadow lines on bottom-right corner
- Color per game context, from the green palette
```

---

## 2. Layout & Mobile Optimization

### iPhone-First Design
Every game is designed to fill the entire screen without scrolling.

**Required meta tags:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
```

**Required CSS rules:**
```css
html, body {
  height: 100%; height: 100dvh; overflow: hidden;
  touch-action: none;
  -webkit-user-select: none; user-select: none;
  -webkit-touch-callout: none;
  -webkit-tap-highlight-color: transparent;
}
body {
  padding: env(safe-area-inset-top) env(safe-area-inset-right)
           env(safe-area-inset-bottom) env(safe-area-inset-left);
}
```

**Prevent scrolling:**
```javascript
document.addEventListener('touchmove', e => e.preventDefault(), { passive: false });
```

### Page Structure (Vertical)
```
┌──────────────────────┐
│  TOPBAR              │  ← flex-shrink: 0, padding: 6px 12px
│  Name  PAUSE  Stats  │     Left: game name + score (topbar-left)
│                      │     Center: PAUSE button (between topbar-left and topbar-right)
│                      │     Right: game-specific info (topbar-right)
├──────────────────────┤
│  GAME AREA (canvas)  │  ← flex: 1, align-items: flex-start, padding: 0
│                      │     Canvas sits tight against topbar (no vertical centering)
│                      │     Canvas scaled dynamically (CSS width/height)
│                      │     Logical resolution stays constant
├──────────────────────┤
│  TOUCH HINT          │  ← flex-shrink: 0, 5px text, opacity: 0.3
└──────────────────────┘
```

### Game Area Scaling
Canvas is always drawn at a fixed logical resolution. CSS size is scaled to fill the space:
```javascript
function resize() {
  const wrap = document.getElementById('boardWrap');
  const aH = wrap.clientHeight;
  const aW = wrap.clientWidth;
  // Fit by aspect ratio — game-specific logic
  let h = aH, w = h / ASPECT;
  if (w > aW) { w = aW; h = w * ASPECT; }
  canvas.style.width = Math.floor(w) + 'px';
  canvas.style.height = Math.floor(h) + 'px';
}
window.addEventListener('resize', resize);
setTimeout(resize, 50);
setTimeout(resize, 200);
resize();
```

### NO Buttons
Games are controlled with swipe gestures and taps.

---

## 3. Controls (Touch + Keyboard)

### Touch Controls (Full Game Area)
Controls are bound to the game area wrapper element (`boardWrap`). The entire game area is the touch target.

**Principles:**
- **Swipe left/right:** move/steer (repeating — longer swipe = more steps)
- **Swipe down:** soft drop / quick action
- **Fast swipe down:** hard drop / strong action (detected by speed, not distance alone)
- **Tap (short touch, <250ms, minimal movement):** rotate / primary action
- Thresholds: horizontal ~24px per step, vertical ~30px per step

**Standard code:**
```javascript
const bw = document.getElementById('boardWrap');
let tx, ty, tt, moved, lastMX, totalDY;
const SH = 24, SV = 30, HDSPD = 1.5; // px/ms hard drop speed

bw.addEventListener('touchstart', e => {
  if (!gameRunning || paused || e.target.id === 'pauseBtn') return;
  e.preventDefault();
  const t = e.touches[0];
  tx = t.clientX; ty = t.clientY; tt = Date.now();
  moved = false; lastMX = tx; totalDY = 0;
}, { passive: false });

bw.addEventListener('touchmove', e => {
  if (!gameRunning || paused) return;
  e.preventDefault();
  const t = e.touches[0];
  const dx = t.clientX - lastMX, dy = t.clientY - ty;
  if (Math.abs(dx) > SH) {
    const s = Math.floor(Math.abs(dx) / SH);
    for (let i = 0; i < s; i++) dx > 0 ? moveRight() : moveLeft();
    lastMX += Math.sign(dx) * s * SH;
    moved = true;
  }
  if (dy > SV) {
    const s = Math.floor(dy / SV);
    for (let i = 0; i < s; i++) softDropAction();
    ty += s * SV; totalDY += s * SV;
    moved = true;
  }
}, { passive: false });

bw.addEventListener('touchend', e => {
  if (!gameRunning || paused) return;
  e.preventDefault();
  const el = Date.now() - tt;
  const ct = e.changedTouches[0];
  const fdx = ct.clientX - tx, fdy = ct.clientY - ty + totalDY;
  if (moved && fdy > 50 && el > 0 && (fdy / el) > HDSPD) { hardDropAction(); return; }
  if (!moved && el < 250 && Math.abs(fdx) < 12 && Math.abs(ct.clientY - ty) < 12) tapAction();
}, { passive: false });
```

Game-specific functions (`moveLeft`, `moveRight`, `softDropAction`, `hardDropAction`, `tapAction`) vary per game.

### Keyboard (PC Support)
Every game also has keyboard controls. Standard:
- **Arrow keys:** movement
- **Arrow Up / Space:** primary action
- **P / Escape:** pause and resume

```javascript
document.addEventListener('keydown', e => {
  if (!gameRunning || paused) return;
  switch (e.key) {
    case 'ArrowLeft': e.preventDefault(); moveLeft(); break;
    case 'ArrowRight': e.preventDefault(); moveRight(); break;
    case 'ArrowDown': e.preventDefault(); softDropAction(); break;
    case 'ArrowUp': e.preventDefault(); tapAction(); break;
    case ' ': e.preventDefault(); hardDropAction(); break;
    case 'p': case 'P': e.preventDefault(); togglePause(); break;
  }
});
```

---

## 4. Game Structure (Standard Architecture)

Every game follows the same base structure:

### HTML Skeleton
```
overlay#startScreen   — Game name + instructions + "TAP TO START"
overlay#gameOver      — "GAME OVER" + results + RETRY/EXIT TO MENU buttons
div.game-container    — Topbar + boardWrap + touchHint
div#pauseOverlay      — "PAUSE" text
```

### JavaScript Structure
```
1. Constants (grid size, colors, shapes)
2. Game state variables
3. Audio (beep function via Web Audio API)
4. Game logic (grid, collision, spawn, clear etc.)
5. Draw functions (draw, drawPreview)
6. UI updates (updateUI, showGameOver)
7. Input functions (move, rotate, drop etc.)
8. Keyboard listener
9. Touch listeners (boardWrap)
10. Pause functionality
11. Game loop (requestAnimationFrame)
12. startGame()
13. Overlay listeners (start/retry)
14. resize()
```

### Sound Effects
All sounds are produced via Web Audio API — square wave retro beeps:
```javascript
let audioCtx;
function beep(freq, duration, volume = 0.08) {
  try {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    if (audioCtx.state === 'suspended') audioCtx.resume();
    const o = audioCtx.createOscillator();
    const g = audioCtx.createGain();
    o.type = 'square';
    o.frequency.value = freq;
    g.gain.value = volume;
    o.connect(g); g.connect(audioCtx.destination);
    o.start();
    g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
    o.stop(audioCtx.currentTime + duration);
  } catch(e) {}
}
```
**Note:** `audioCtx.resume()` is required — iOS Safari and many mobile browsers create the AudioContext in `suspended` state. Without resume, no sound plays.
Typical sounds: move 220Hz/50ms, rotate 440Hz/60ms, drop 160Hz/100ms, line clear 660Hz/150ms, game over 100Hz/400ms.

### High Score
Saved to `localStorage` with a game-specific key:
```javascript
try { localStorage.setItem('GAMENAME_hi', hiScore); } catch(e) {}
```

### Game Loop
```javascript
function gameLoop(time) {
  if (!gameRunning) { draw(); return; }
  if (paused) return;
  const dt = time - (lastTime || time);
  lastTime = time;
  // Game logic based on dt
  draw();
  requestAnimationFrame(gameLoop);
}
```

---

## 5. Overlay Screens

### Start Screen
- Game name large (`<h1>`, titleGlow)
- Short instructions (swipe/tap descriptions in game context)
- "PRESS START" with blink animation (or "TAP TO START")
- All text in English
- **Activation:** click, touchend AND `Enter`/`Space` key

### Game Over Screen
- "GAME OVER" in amber
- Final result (score, level, game-specific details)
- Two buttons: RETRY and EXIT TO MENU (same `pause-menu-btn` styles as pause menu)
- RETRY is default selection (`sel` class)
- **Keyboard navigation:** ↑↓ toggles RETRY / EXIT TO MENU, Enter/Space confirms
- **Touch/click:** buttons respond to click + touchend events
- EXIT TO MENU goes directly to menu (`window.location.href = 'index.html'`) — no confirm dialog (game is already over)
- Same applies to win screens (CONTINUE + EXIT TO MENU)

### Pause
- Separate overlay `z-index: 400`
- "PAUSE" with blink animation
- Overlay contains "RESUME" and "EXIT TO MENU" buttons
- `P`/`Escape` pauses and resumes the game (except from EXIT TO MENU button)
- Keyboard navigation: ↑↓ toggles RESUME / EXIT TO MENU, Enter confirms

**Pause button listeners — IMPORTANT:**
pauseBtn needs both `click` and `touchend` — boardWrap calls `e.preventDefault()` on touchend, which prevents the `click` event from firing on mobile:
```javascript
document.getElementById('pauseBtn').addEventListener('click', e => { e.stopPropagation(); togglePause(); });
document.getElementById('pauseBtn').addEventListener('touchend', e => { e.stopPropagation(); e.preventDefault(); togglePause(); });
pauseOv.addEventListener('click', e => { if (!['resumeBtn','menuBtn','confirmYes','confirmNo'].includes(e.target.id) && !e.target.closest('#confirmBox')) togglePause(); });
pauseOv.addEventListener('touchend', e => { if (!['resumeBtn','menuBtn','confirmYes','confirmNo'].includes(e.target.id) && !e.target.closest('#confirmBox')) { e.preventDefault(); togglePause(); } });
['click', 'touchend'].forEach(ev => document.getElementById('resumeBtn').addEventListener(ev, e => { e.stopPropagation(); e.preventDefault(); togglePause(); }));
['click', 'touchend'].forEach(ev => document.getElementById('menuBtn').addEventListener(ev, e => { e.stopPropagation(); e.preventDefault(); showConfirm(); }));
```

### Standard Overlay Listener Code

All overlays support mouse, touch, and keyboard — the game is fully playable without a mouse:

```javascript
// Game over menu
let goSel = 0; // 0=RETRY, 1=EXIT TO MENU
function updateGoSel() {
  document.getElementById('goRetryBtn').classList.toggle('sel', goSel === 0);
  document.getElementById('goMenuBtn').classList.toggle('sel', goSel === 1);
}
function goRetry() { document.getElementById('gameOver').classList.add('hidden'); startGame(); }
function goMenu() { window.location.href = 'index.html'; }

// Click & touch
['click', 'touchend'].forEach(ev => {
  document.getElementById('startScreen').addEventListener(ev, e => {
    e.preventDefault();
    document.getElementById('startScreen').classList.add('hidden');
    startGame();
  });
  document.getElementById('goRetryBtn').addEventListener(ev, e => {
    e.stopPropagation(); e.preventDefault(); goRetry();
  });
  document.getElementById('goMenuBtn').addEventListener(ev, e => {
    e.stopPropagation(); e.preventDefault(); goMenu();
  });
});

// Keyboard — Enter or Space starts from the start screen
document.addEventListener('keydown', e => {
  if (e.key === 'Enter' || e.key === ' ') {
    const start = document.getElementById('startScreen');
    if (!start.classList.contains('hidden')) {
      e.preventDefault(); start.classList.add('hidden'); startGame();
    }
  }
});
```

Game over keyboard navigation (↑↓ + Enter) is added to the main keydown listener after pause logic:
```javascript
  // Game over menu navigation
  if (!document.getElementById('gameOver').classList.contains('hidden')) {
    if (e.key === 'ArrowDown' || e.key === 'ArrowUp') { e.preventDefault(); goSel = goSel === 0 ? 1 : 0; updateGoSel(); }
    else if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); if (goSel === 1) goMenu(); else goRetry(); }
    return;
  }
```

**Note:** If a game has additional overlay screens (e.g. "ALL CLEAR" / "YOU WIN"), add equivalent buttons and navigation the same way.

---

## 6. File Structure

```
games/
├── index.html          ← Main menu
├── tetris.html         ← Tetris
├── arkanoid.html       ← Arkanoid
├── pong.html           ← Pong
└── ...                 ← More games
```

Every game is a standalone HTML file — no shared JS/CSS files. This keeps things simple and each game easy to test and share independently.

---

## 7. Main Menu (index.html)

The menu follows the same CRT style. Structure:

```
┌──────────────────────┐
│     RETRO ARCADE     │  ← titleGlow
├──────────────────────┤
│                      │
│  ► TETRIS      1250  │  ← Game listing, hi-score on right
│  ► ARKANOID     890  │     Selected/hover: --amber highlight
│  ► PONG         ---  │     Tap/click opens game
│                      │     "---" if no hi-score
└──────────────────────┘
│     SWIPE ↑↓ SELECT
      TAP TO PLAY
```

- Game list is scrollable if many games
- Hi-scores are read from `localStorage` (keys: `tetris_hi`, `arkanoid_hi` etc.)
- Navigation: `window.location.href = 'tetris.html'`
- Each game has a way back to menu (e.g. EXIT TO MENU in pause/game over)

---

## 8. Language

- All in-game text **in English** — instructions, hints, overlays, game terms
- Examples: "TAP TO START", "TAP TO RETRY", "GAME OVER", "SWIPE ←→ MOVE", "TAP ROTATE", "FAST↓ DROP", "SCORE", "LEVEL", "LINES", "HI", "NEXT", "PAUSE"
- English is the natural language for a retro arcade context and makes games universally playable

---

## 9. Automated Testing (Playwright)

Every game is tested automatically in a headless browser before release. Tests run with Playwright (Chromium) and simulate real gameplay — keyboard, touch gestures, and direct state manipulation.

### Environment and Setup

```javascript
const { chromium } = require('playwright');
const path = require('path');

const browser = await chromium.launch();
const page = await browser.newPage({
  viewport: { width: 390, height: 844 },  // iPhone 14
  hasTouch: true                           // Required for touch tests
});
await page.goto('file://' + path.resolve('game.html'));
await page.waitForTimeout(500);
```

### Test Categories

Every game's test script covers these categories:

#### 1. UI State and Overlay Screens
- Start screen is visible on load
- Click/tap hides the start screen and starts the game
- Game over screen appears when the game ends
- Retry restarts the game

```javascript
// Example
const startVisible = await page.isVisible('#startScreen');
await page.click('#startScreen');
await page.waitForTimeout(300);
const startHidden = await page.isHidden('#startScreen');
```

#### 2. Initial State
- `gameRunning === true`, `paused === false`
- Score 0, correct number of lives, level 1
- Game-specific initial values (e.g. invader count, brick count)

```javascript
const state = await page.evaluate(() => ({
  gameRunning, paused, scoreVal, livesVal
}));
```

#### 3. Keyboard Controls
- Arrow keys move the player/piece
- Primary action (arrow up/space) works
- Individual key presses register (not just keysDown loop)

```javascript
await page.evaluate(() => { playerX = 150; });
for (let i = 0; i < 10; i++) await page.keyboard.press('ArrowLeft');
const after = await page.evaluate(() => playerX);
// assert after < 150
```

#### 4. Player Bounds
- Player/paddle does not go outside the game area
- Clamp works also with direct state manipulation (not just input functions)

```javascript
await page.evaluate(() => { playerX = -10; });
await page.evaluate(() => new Promise(r => requestAnimationFrame(() => setTimeout(r, 30))));
const clamped = await page.evaluate(() => playerX);
// assert clamped >= 0
```

#### 5. Game Mechanics
Game-specific tests that verify core logic:
- **Arkanoid:** ball bounces off paddle/bricks, bricks are destroyed, level advances
- **Tetris:** piece drops, line clear, collision
- **Invaders:** bullet hits invader, scoring, invaders move, wave clear

```javascript
// Direct hit — place bullet/ball near target
await page.evaluate(() => {
  playerBullet = { x: targetX, y: targetY + 5 };
});
for (let i = 0; i < 30; i++) {
  const s = await page.evaluate(() => ({ scoreVal }));
  if (s.scoreVal > 0) break;
  await page.evaluate(() => new Promise(r => requestAnimationFrame(r)));
}
```

#### 6. Pause
- P key activates pause
- Overlay is visible
- Second P resumes the game

#### 7. Lives and Game Over
- Life loss decrements the counter
- At zero, game over screen appears and `gameRunning === false`

```javascript
// Force death
await page.evaluate(() => {
  livesVal = 1;
  // set state that causes death
});
// Run frames
// Check: gameRunning === false, gameOver overlay visible
```

#### 8. Canvas Resize
- Different viewport sizes produce different canvas CSS sizes
- Test with at least two sizes (e.g. iPhone SE + iPhone 14 Pro Max)

```javascript
await page.setViewportSize({ width: 320, height: 568 });
await page.waitForTimeout(300);
const size1 = await page.evaluate(() => canvas.style.width);
await page.setViewportSize({ width: 428, height: 926 });
await page.waitForTimeout(300);
const size2 = await page.evaluate(() => canvas.style.width);
// assert size1 !== size2
```

#### 9. High Score
- `localStorage` key is saved when points are earned
- Key follows the `gamename_hi` format

#### 10. Touch Simulation
- Tap on game area performs the primary action
- Context: `hasTouch: true` in Playwright settings

```javascript
const box = await page.locator('#boardWrap').boundingBox();
await page.touchscreen.tap(box.x + box.width/2, box.y + box.height/2);
```

#### 11. Long Play Test (Stress Test)
- AI-driven play for 300–500 frames
- Check for NaN values, stuck states, or crashes
- Simple AI follows a target and performs actions

```javascript
for (let i = 0; i < 500; i++) {
  const s = await page.evaluate(() => ({
    gameRunning, playerX, scoreVal, /* game-specific */
  }));
  if (!s.gameRunning) break;

  // NaN check
  if (isNaN(s.playerX)) { anomalies.push('NaN'); break; }

  // Simple AI
  if (targetX < s.playerX) await page.keyboard.press('ArrowLeft');
  else if (targetX > s.playerX) await page.keyboard.press('ArrowRight');
  else await page.keyboard.press('ArrowUp'); // action

  await page.evaluate(() => new Promise(r => requestAnimationFrame(r)));
}
```

### Output Format

The test script prints a clear list of results:

```
=== GAMENAME AUTOMATED TEST ===

[1] Start screen
  ✓ visible
[2] Tap to start
  ✓ hidden after tap
[3] Initial state
  ✓ running
  ✓ 3 lives
  ✗ score 0 FAIL
...

=== RESULTS: 30 passed, 2 failed ===
```

Helper function:
```javascript
let pass = 0, fail = 0;
function check(name, ok) {
  if (ok) { pass++; console.log(`  ✓ ${name}`); }
  else { fail++; console.log(`  ✗ ${name} FAIL`); }
}
```

### Running Tests

```bash
# Install Playwright (once)
npx playwright install chromium

# Run test
node test-gamename.js
```

### Notes

- **Frame-by-frame execution:** Use `await page.evaluate(() => new Promise(r => requestAnimationFrame(r)))` to advance one frame — more precise than `waitForTimeout`
- **State manipulation:** `page.evaluate()` has direct access to game variables — use this to test difficult situations (place ball near target, force death etc.)
- **Timing issues:** There is overhead between Playwright frames (~5–20ms), so for moving target collision tests place the bullet/ball near the target rather than waiting for a natural hit
- **Touch requires `hasTouch: true`:** Otherwise `page.touchscreen.tap()` throws an error

---

## 10. Checklist for a New Game

When creating a new game, verify:

- [ ] Single HTML file, no external dependencies (except Google Fonts)
- [ ] CRT effects (scanlines, vignette) via `body.crt`
- [ ] CSS variables — use the same palette
- [ ] Press Start 2P font
- [ ] iPhone meta tags and safe area support
- [ ] Full screen, no scrolling, `100dvh`
- [ ] Game area fills all space between topbar and hint
- [ ] Touch controls on full game area (no buttons)
- [ ] Keyboard support for PC
- [ ] Start screen with instructions
- [ ] Game over screen with results
- [ ] Overlays respond to Enter/Space key (game fully playable with keyboard)
- [ ] Pause functionality (P key + button)
- [ ] Sound effects (beep function)
- [ ] High score to localStorage
- [ ] `resize()` function that scales the canvas
- [ ] `requestAnimationFrame`-based game loop with delta time
- [ ] All text in English
- [ ] localStorage key in format `gamename_hi`
- [ ] Automated tests run and passing (see section 9)

---

## 11. Reference Implementation

The Tetris clone (`tetris.html`) serves as the reference for all future games. It demonstrates:
- CRT style implementation
- Topbar layout (name + score on left, level + lines + next-preview on right)
- Full game area touch controls without buttons
- Canvas scaling for mobile
- 7-bag randomizer
- Ghost piece effect
- Lock delay mechanic
- Overlay screen structure

Use it as a reference and adapt to the game's requirements.
