# Retro Hoops Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a single self-contained HTML5 Canvas 16-bit retro basketball game with slingshot shooting, moving hoop, scoring, and a 90-second countdown timer.

**Architecture:** One `index.html` file with all CSS and JS inlined. A single `state` object drives the entire game. A `requestAnimationFrame` loop calls `update()` then `draw()` every frame. Mouse events handle the slingshot drag mechanic.

**Tech Stack:** Vanilla HTML5, Canvas 2D API, vanilla JavaScript (ES6), Google Fonts (`Press Start 2P`) via CSS @import.

**Output file:** `/Users/carlray/VS_Code/RetroHoops/index.html`

---

### Task 1: HTML Shell + Canvas Setup

**Files:**
- Create: `index.html`

**Step 1: Create the HTML shell**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Retro Hoops</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background: #0a0a1a;
      display: flex;
      align-items: center;
      justify-content: center;
      height: 100vh;
      overflow: hidden;
    }
    canvas {
      display: block;
      image-rendering: pixelated;
      cursor: crosshair;
    }
  </style>
</head>
<body>
  <canvas id="game" width="800" height="550"></canvas>
  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');

    // --- CONSTANTS ---
    const W = canvas.width;   // 800
    const H = canvas.height;  // 550
    const BALL_RADIUS = 18;
    const BALL_START_X = W / 2;
    const BALL_START_Y = H - 80;
    const MAX_DRAG = 120;
    const GRAVITY = 0.4;
    const BOUNCE = 0.5;
    const GAME_DURATION = 90; // seconds

    // Hoop dimensions
    const RIM_HALF = 28;      // half-width of the hoop opening
    const RIM_Y_DEFAULT = 180;
    const NET_HEIGHT = 40;

    // --- STATE ---
    const state = {
      ball: { x: BALL_START_X, y: BALL_START_Y, vx: 0, vy: 0, dragging: false, launched: false },
      hoop: { x: W / 2, y: RIM_Y_DEFAULT, targetX: W / 2, targetY: RIM_Y_DEFAULT },
      score: 0,
      bestScore: parseInt(localStorage.getItem('retroHoops_best') || '0'),
      timeLeft: GAME_DURATION,
      phase: 'idle', // 'idle' | 'dragging' | 'flying' | 'resetting' | 'gameover'
      dragStart: { x: 0, y: 0 },
      dragCurrent: { x: 0, y: 0 },
      frameCount: 0,
      popTexts: [],       // { text, x, y, alpha, dy }
      lastShotRimHit: false, // tracks if ball touched rim this shot
      basketCount: 0,
    };

    console.log('Game initialized');
  </script>
</body>
</html>
```

**Step 2: Open in browser and verify**

```bash
open /Users/carlray/VS_Code/RetroHoops/index.html
```

Expected: Dark navy page with a black canvas rectangle centered. No errors in browser console.

**Step 3: Commit**

```bash
cd /Users/carlray/VS_Code/RetroHoops
git add index.html
git commit -m "feat: add HTML shell with canvas and game state"
```

---

### Task 2: Draw Court + Background

**Files:**
- Modify: `index.html` (add `drawCourt()` and a basic game loop)

**Step 1: Add `drawCourt()` and stub game loop after the state declaration**

Add these functions inside the `<script>` block:

```javascript
// ---- DRAWING ----

function drawBackground() {
  // Sky gradient
  const grad = ctx.createLinearGradient(0, 0, 0, H * 0.65);
  grad.addColorStop(0, '#0a0a2e');
  grad.addColorStop(1, '#1a1a4e');
  ctx.fillStyle = grad;
  ctx.fillRect(0, 0, W, H * 0.65);

  // Crowd silhouettes (simple pixel rows)
  ctx.fillStyle = '#12124a';
  for (let row = 0; row < 4; row++) {
    const y = 60 + row * 28;
    for (let x = 0; x < W; x += 18 + (row % 3) * 6) {
      const h = 18 + (row % 2) * 8;
      ctx.fillRect(x, y, 14, h);
    }
  }
}

function drawCourt() {
  const floorY = H * 0.65;

  // Floor base
  ctx.fillStyle = '#8B5E1A';
  ctx.fillRect(0, floorY, W, H - floorY);

  // Hardwood planks
  ctx.fillStyle = '#7A5016';
  const plankH = 14;
  for (let y = floorY; y < H; y += plankH * 2) {
    ctx.fillRect(0, y + plankH, W, plankH);
  }

  // Plank seam lines
  ctx.strokeStyle = '#6B4512';
  ctx.lineWidth = 1;
  for (let x = 60; x < W; x += 60) {
    ctx.beginPath();
    ctx.moveTo(x, floorY);
    ctx.lineTo(x, H);
    ctx.stroke();
  }

  // Court line
  ctx.strokeStyle = '#ffffff44';
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(0, floorY);
  ctx.lineTo(W, floorY);
  ctx.stroke();
}

function drawScanlines() {
  ctx.fillStyle = 'rgba(0,0,0,0.08)';
  for (let y = 0; y < H; y += 3) {
    ctx.fillRect(0, y, W, 1);
  }
}

function draw() {
  ctx.clearRect(0, 0, W, H);
  drawBackground();
  drawCourt();
  drawScanlines();
}

function gameLoop() {
  draw();
  requestAnimationFrame(gameLoop);
}

gameLoop();
```

**Step 2: Verify in browser**

Reload the page. Expected: Dark navy background with crowd silhouette rows, brown hardwood floor at bottom 35%, CRT scanline effect.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: draw court background with crowd and hardwood floor"
```

---

### Task 3: Draw Hoop + Ball

**Files:**
- Modify: `index.html`

**Step 1: Add `drawHoop()` and `drawBall()` functions**

Insert before the `draw()` function:

```javascript
function drawHoop(x, y) {
  // Backboard (right side)
  ctx.fillStyle = '#eeeeee';
  ctx.fillRect(x + RIM_HALF + 6, y - 36, 10, 56);
  // Backboard inner square
  ctx.strokeStyle = '#e05000';
  ctx.lineWidth = 2;
  ctx.strokeRect(x + RIM_HALF + 8, y - 24, 6, 20);

  // Support arm from backboard to rim
  ctx.strokeStyle = '#aaaaaa';
  ctx.lineWidth = 3;
  ctx.beginPath();
  ctx.moveTo(x + RIM_HALF + 6, y + 4);
  ctx.lineTo(x + RIM_HALF, y + 4);
  ctx.stroke();

  // Rim (two pixel-art circle ends)
  ctx.fillStyle = '#e05000';
  ctx.fillRect(x - RIM_HALF - 5, y, 10, 6);      // left rim end
  ctx.fillRect(x + RIM_HALF - 5, y, 10, 6);      // right rim end

  // Rim bar connecting
  ctx.fillStyle = '#e05000';
  ctx.fillRect(x - RIM_HALF, y + 2, RIM_HALF * 2, 3);

  // Net (pixel lines hanging down)
  ctx.strokeStyle = '#ffffff88';
  ctx.lineWidth = 1;
  const netSegs = 7;
  for (let i = 0; i <= netSegs; i++) {
    const nx = x - RIM_HALF + (RIM_HALF * 2 / netSegs) * i;
    const bottomX = x - RIM_HALF * 0.4 + (RIM_HALF * 0.8 / netSegs) * i;
    ctx.beginPath();
    ctx.moveTo(nx, y + 5);
    ctx.lineTo(bottomX, y + NET_HEIGHT);
    ctx.stroke();
  }
  // Net horizontal lines
  for (let row = 1; row < 4; row++) {
    const t = row / 4;
    const lx = x - RIM_HALF + RIM_HALF * 0.2 * t;
    const rx = x + RIM_HALF - RIM_HALF * 0.2 * t;
    const ny = y + 5 + NET_HEIGHT * t;
    ctx.beginPath();
    ctx.moveTo(lx, ny);
    ctx.lineTo(rx, ny);
    ctx.stroke();
  }
}

function drawBall(x, y) {
  // Main orange circle
  ctx.fillStyle = '#e85d04';
  ctx.beginPath();
  ctx.arc(x, y, BALL_RADIUS, 0, Math.PI * 2);
  ctx.fill();

  // Highlight
  ctx.fillStyle = '#f4a261';
  ctx.beginPath();
  ctx.arc(x - 5, y - 5, 6, 0, Math.PI * 2);
  ctx.fill();

  // Black seam lines (pixel art style)
  ctx.strokeStyle = '#111';
  ctx.lineWidth = 2;
  // Vertical seam
  ctx.beginPath();
  ctx.moveTo(x, y - BALL_RADIUS);
  ctx.bezierCurveTo(x + 10, y - 5, x + 10, y + 5, x, y + BALL_RADIUS);
  ctx.stroke();
  ctx.beginPath();
  ctx.moveTo(x, y - BALL_RADIUS);
  ctx.bezierCurveTo(x - 10, y - 5, x - 10, y + 5, x, y + BALL_RADIUS);
  ctx.stroke();
  // Horizontal seam
  ctx.beginPath();
  ctx.moveTo(x - BALL_RADIUS, y);
  ctx.bezierCurveTo(x - 5, y - 10, x + 5, y - 10, x + BALL_RADIUS, y);
  ctx.stroke();
}
```

**Step 2: Call them in `draw()`**

Update the `draw()` function to:

```javascript
function draw() {
  ctx.clearRect(0, 0, W, H);
  drawBackground();
  drawCourt();
  drawHoop(state.hoop.x, state.hoop.y);
  drawBall(state.ball.x, state.ball.y);
  drawScanlines();
}
```

**Step 3: Verify in browser**

Reload. Expected: Orange basketball at center-bottom, orange rim with white net and backboard at center-upper area.

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: draw pixel-art hoop and basketball"
```

---

### Task 4: Mouse Drag + Trajectory Preview

**Files:**
- Modify: `index.html`

**Step 1: Add mouse event listeners after `gameLoop()` call**

```javascript
function getCanvasPos(e) {
  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width / rect.width;
  const scaleY = canvas.height / rect.height;
  return {
    x: (e.clientX - rect.left) * scaleX,
    y: (e.clientY - rect.top) * scaleY,
  };
}

function isOnBall(pos) {
  const dx = pos.x - state.ball.x;
  const dy = pos.y - state.ball.y;
  return Math.sqrt(dx * dx + dy * dy) < BALL_RADIUS + 12; // generous hit area
}

canvas.addEventListener('mousedown', (e) => {
  if (state.phase !== 'idle') return;
  const pos = getCanvasPos(e);
  if (isOnBall(pos)) {
    state.phase = 'dragging';
    state.ball.dragging = true;
    state.dragStart = { x: state.ball.x, y: state.ball.y };
    state.dragCurrent = { x: pos.x, y: pos.y };
  }
});

canvas.addEventListener('mousemove', (e) => {
  if (state.phase !== 'dragging') return;
  const pos = getCanvasPos(e);
  // Clamp drag to MAX_DRAG
  const dx = pos.x - state.dragStart.x;
  const dy = pos.y - state.dragStart.y;
  const dist = Math.sqrt(dx * dx + dy * dy);
  if (dist > MAX_DRAG) {
    state.dragCurrent.x = state.dragStart.x + (dx / dist) * MAX_DRAG;
    state.dragCurrent.y = state.dragStart.y + (dy / dist) * MAX_DRAG;
  } else {
    state.dragCurrent = { x: pos.x, y: pos.y };
  }
});

canvas.addEventListener('mouseup', () => {
  if (state.phase !== 'dragging') return;
  const dx = state.dragStart.x - state.dragCurrent.x;
  const dy = state.dragStart.y - state.dragCurrent.y;
  const speed = Math.sqrt(dx * dx + dy * dy);
  if (speed < 8) {
    // Too short a drag — cancel
    state.phase = 'idle';
    state.ball.dragging = false;
    return;
  }
  // Launch: velocity is opposite to drag direction
  const power = speed / MAX_DRAG;
  state.ball.vx = dx * 0.18 * power;
  state.ball.vy = dy * 0.18 * power;
  state.ball.launched = true;
  state.ball.dragging = false;
  state.lastShotRimHit = false;
  state.phase = 'flying';
});
```

**Step 2: Add trajectory preview dots**

Add this function before `draw()`:

```javascript
function drawTrajectory() {
  if (state.phase !== 'dragging') return;
  const dx = state.dragStart.x - state.dragCurrent.x;
  const dy = state.dragStart.y - state.dragCurrent.y;
  const speed = Math.sqrt(dx * dx + dy * dy);
  if (speed < 8) return;

  const power = speed / MAX_DRAG;
  let sx = state.ball.x;
  let sy = state.ball.y;
  let svx = dx * 0.18 * power;
  let svy = dy * 0.18 * power;

  ctx.fillStyle = '#ffffffaa';
  for (let i = 0; i < 18; i++) {
    svx *= 1;
    svy += GRAVITY;
    sx += svx;
    sy += svy;
    if (sy > H) break;
    const alpha = 1 - i / 18;
    ctx.globalAlpha = alpha * 0.7;
    const r = 4 - i * 0.15;
    ctx.beginPath();
    ctx.arc(sx, sy, Math.max(1, r), 0, Math.PI * 2);
    ctx.fill();
  }
  ctx.globalAlpha = 1;
}

function drawDragLine() {
  if (state.phase !== 'dragging') return;
  // Draw rubber band from ball to drag point
  ctx.strokeStyle = '#ffffff55';
  ctx.lineWidth = 2;
  ctx.setLineDash([4, 4]);
  ctx.beginPath();
  ctx.moveTo(state.ball.x, state.ball.y);
  ctx.lineTo(state.dragCurrent.x, state.dragCurrent.y);
  ctx.stroke();
  ctx.setLineDash([]);
}
```

**Step 3: Add drag visual to draw() — call before ball, after hoop**

```javascript
function draw() {
  ctx.clearRect(0, 0, W, H);
  drawBackground();
  drawCourt();
  drawHoop(state.hoop.x, state.hoop.y);
  drawTrajectory();
  drawDragLine();
  drawBall(state.ball.x, state.ball.y);
  drawScanlines();
}
```

**Step 4: Verify in browser**

Reload. Click and drag the ball — a dashed rubber band and dotted trajectory arc should appear. Release fires the ball in the opposite direction.

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: slingshot drag mechanic with trajectory preview"
```

---

### Task 5: Physics Update Loop + Ball Reset

**Files:**
- Modify: `index.html`

**Step 1: Add `update()` function before `gameLoop()`**

```javascript
function resetBall() {
  state.ball.x = BALL_START_X;
  state.ball.y = BALL_START_Y;
  state.ball.vx = 0;
  state.ball.vy = 0;
  state.ball.launched = false;
  state.ball.dragging = false;
  state.phase = 'idle';
}

function update() {
  if (state.phase === 'flying') {
    state.ball.vy += GRAVITY;
    state.ball.x += state.ball.vx;
    state.ball.y += state.ball.vy;

    // Floor bounce / out of bounds
    if (state.ball.y > H + 60 || state.ball.x < -60 || state.ball.x > W + 60) {
      // Miss — reset after short delay
      state.phase = 'resetting';
      setTimeout(() => resetBall(), 800);
    }

    // Floor collision
    if (state.ball.y + BALL_RADIUS > H - 10) {
      state.ball.y = H - 10 - BALL_RADIUS;
      state.ball.vy *= -BOUNCE;
      state.ball.vx *= 0.85;
      if (Math.abs(state.ball.vy) < 1) {
        state.phase = 'resetting';
        setTimeout(() => resetBall(), 500);
      }
    }
  }

  // Hoop lerp toward target
  state.hoop.x += (state.hoop.targetX - state.hoop.x) * 0.05;
  state.hoop.y += (state.hoop.targetY - state.hoop.y) * 0.05;
}
```

**Step 2: Call `update()` in the game loop**

```javascript
function gameLoop() {
  update();
  draw();
  requestAnimationFrame(gameLoop);
}
```

**Step 3: Verify in browser**

Shoot the ball — it should arc, hit the floor and bounce, then reset. Missed shots (off screen) reset after 0.8s.

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: physics update loop with gravity and ball reset"
```

---

### Task 6: Collision Detection + Scoring

**Files:**
- Modify: `index.html`

**Step 1: Add collision detection and scoring logic inside `update()`, before the floor check**

Add this inside the `if (state.phase === 'flying')` block, before the floor collision:

```javascript
// Rim collision
const hoopX = state.hoop.x;
const hoopY = state.hoop.y;

const leftRimX = hoopX - RIM_HALF;
const rightRimX = hoopX + RIM_HALF;
const rimY = hoopY + 3;

// Left rim
const dlx = state.ball.x - leftRimX;
const dly = state.ball.y - rimY;
if (Math.sqrt(dlx * dlx + dly * dly) < BALL_RADIUS + 4) {
  state.ball.vx = Math.abs(state.ball.vx) * BOUNCE + 1;
  state.ball.vy *= -BOUNCE;
  state.lastShotRimHit = true;
}

// Right rim
const drx = state.ball.x - rightRimX;
const dry = state.ball.y - rimY;
if (Math.sqrt(drx * drx + dry * dry) < BALL_RADIUS + 4) {
  state.ball.vx = -Math.abs(state.ball.vx) * BOUNCE - 1;
  state.ball.vy *= -BOUNCE;
  state.lastShotRimHit = true;
}

// Backboard collision
const bbX = hoopX + RIM_HALF + 6;
if (state.ball.x + BALL_RADIUS > bbX && state.ball.x - BALL_RADIUS < bbX + 10
    && state.ball.y > hoopY - 36 && state.ball.y < hoopY + 20) {
  state.ball.vx *= -BOUNCE;
  state.lastShotRimHit = true;
}

// Score detection: ball passes through net zone
const netLeft = hoopX - RIM_HALF + 6;
const netRight = hoopX + RIM_HALF - 6;
const netTop = hoopY + 5;
const netBottom = hoopY + NET_HEIGHT;
if (
  state.ball.x > netLeft && state.ball.x < netRight &&
  state.ball.y > netTop && state.ball.y < netBottom &&
  state.ball.vy > 0 // moving downward
) {
  const isSwish = !state.lastShotRimHit;
  const points = isSwish ? 3 : 2;
  state.score += points;
  state.basketCount++;

  if (state.score > state.bestScore) {
    state.bestScore = state.score;
    localStorage.setItem('retroHoops_best', state.bestScore);
  }

  // Pop text
  state.popTexts.push({
    text: isSwish ? 'SWISH! +3' : 'NICE! +2',
    x: hoopX,
    y: hoopY - 20,
    alpha: 1,
    dy: -1.5,
  });

  // Move hoop
  moveHoop();

  state.phase = 'resetting';
  setTimeout(() => resetBall(), 700);
}
```

**Step 2: Add `moveHoop()` function**

```javascript
function moveHoop() {
  const margin = 120;
  state.hoop.targetX = margin + Math.random() * (W - margin * 2);

  // Every 5 baskets, also change Y
  if (state.basketCount % 5 === 0) {
    state.hoop.targetY = 130 + Math.random() * 120;
  }
}
```

**Step 3: Add pop text update + draw**

Add to `update()` at the end (outside `if flying`):

```javascript
// Update pop texts
state.popTexts = state.popTexts.filter(p => p.alpha > 0.02);
state.popTexts.forEach(p => {
  p.y += p.dy;
  p.alpha -= 0.018;
});
```

Add `drawPopTexts()` function:

```javascript
function drawPopTexts() {
  ctx.font = '14px "Press Start 2P", monospace';
  ctx.textAlign = 'center';
  state.popTexts.forEach(p => {
    ctx.globalAlpha = p.alpha;
    ctx.fillStyle = p.text.startsWith('SWISH') ? '#ffd700' : '#ffffff';
    ctx.fillText(p.text, p.x, p.y);
  });
  ctx.globalAlpha = 1;
  ctx.textAlign = 'left';
}
```

Add `drawPopTexts()` call in `draw()` before `drawScanlines()`.

**Step 4: Verify in browser**

Shoot the ball through the hoop. Score should appear as a floating text. Hoop should move after each basket. Swish (clean shot, no rim) should show "SWISH! +3".

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: collision detection, scoring, pop text, hoop movement"
```

---

### Task 7: HUD — Score, Timer, Best Score

**Files:**
- Modify: `index.html`

**Step 1: Add timer tick logic to `update()`**

At the top of `update()`, before the `if flying` block:

```javascript
// Timer
if (state.phase !== 'gameover') {
  state.frameCount++;
  if (state.frameCount >= 60) {
    state.frameCount = 0;
    if (state.timeLeft > 0) {
      state.timeLeft--;
    } else {
      state.phase = 'gameover';
      if (state.score > state.bestScore) {
        state.bestScore = state.score;
        localStorage.setItem('retroHoops_best', state.bestScore);
      }
    }
  }
}
```

**Step 2: Add `drawHUD()` function**

```javascript
function drawHUD() {
  const font = '"Press Start 2P", monospace';
  ctx.textAlign = 'left';

  // Score (top-left)
  ctx.font = `13px ${font}`;
  ctx.fillStyle = '#ffffff';
  ctx.fillText('SCORE', 20, 30);
  ctx.font = `20px ${font}`;
  ctx.fillStyle = '#ffd700';
  ctx.fillText(String(state.score).padStart(3, '0'), 20, 56);

  // Timer (top-center)
  const mins = Math.floor(state.timeLeft / 60);
  const secs = String(state.timeLeft % 60).padStart(2, '0');
  const timeStr = `${mins}:${secs}`;
  ctx.font = `20px ${font}`;
  ctx.textAlign = 'center';
  ctx.fillStyle = state.timeLeft <= 10 ? '#ff4444' : '#ffffff';
  ctx.fillText(timeStr, W / 2, 50);
  ctx.font = `10px ${font}`;
  ctx.fillStyle = '#aaaaaa';
  ctx.fillText('TIME', W / 2, 28);

  // Best (top-right)
  ctx.textAlign = 'right';
  ctx.font = `11px ${font}`;
  ctx.fillStyle = '#ffffff';
  ctx.fillText('BEST', W - 20, 30);
  ctx.font = `18px ${font}`;
  ctx.fillStyle = '#00ff88';
  ctx.fillText(String(state.bestScore).padStart(3, '0'), W - 20, 56);

  ctx.textAlign = 'left';
}
```

**Step 3: Add `drawHUD()` to `draw()` before `drawScanlines()`**

**Step 4: Verify in browser**

Reload. Score, timer (counting down), and best score should all appear at the top. Timer turns red in the last 10 seconds.

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: HUD with score, countdown timer, best score"
```

---

### Task 8: Game Over Screen + Play Again

**Files:**
- Modify: `index.html`

**Step 1: Add `drawGameOver()` function**

```javascript
function drawGameOver() {
  if (state.phase !== 'gameover') return;

  // Dark overlay
  ctx.fillStyle = 'rgba(0,0,0,0.75)';
  ctx.fillRect(0, 0, W, H);

  const font = '"Press Start 2P", monospace';
  ctx.textAlign = 'center';

  // Title
  ctx.font = `28px ${font}`;
  ctx.fillStyle = '#ffd700';
  ctx.fillText('GAME OVER', W / 2, H / 2 - 90);

  // Final score
  ctx.font = `14px ${font}`;
  ctx.fillStyle = '#ffffff';
  ctx.fillText('SCORE', W / 2, H / 2 - 40);
  ctx.font = `36px ${font}`;
  ctx.fillStyle = '#ffd700';
  ctx.fillText(state.score, W / 2, H / 2);

  // Best score
  ctx.font = `11px ${font}`;
  ctx.fillStyle = '#00ff88';
  ctx.fillText(`BEST: ${state.bestScore}`, W / 2, H / 2 + 50);

  // Play again button
  const btnX = W / 2 - 130;
  const btnY = H / 2 + 80;
  const btnW = 260;
  const btnH = 44;
  ctx.fillStyle = '#e85d04';
  ctx.fillRect(btnX, btnY, btnW, btnH);
  ctx.fillStyle = '#ffffff';
  ctx.font = `13px ${font}`;
  ctx.fillText('PLAY AGAIN', W / 2, btnY + 28);

  // Store button bounds for click detection
  state.playAgainBtn = { x: btnX, y: btnY, w: btnW, h: btnH };

  ctx.textAlign = 'left';
}
```

**Step 2: Add `drawGameOver()` call to `draw()` at the very end (after scanlines)**

**Step 3: Add click handler for Play Again**

Add after the existing mouse event listeners:

```javascript
canvas.addEventListener('click', (e) => {
  if (state.phase !== 'gameover') return;
  const pos = getCanvasPos(e);
  const btn = state.playAgainBtn;
  if (!btn) return;
  if (pos.x >= btn.x && pos.x <= btn.x + btn.w &&
      pos.y >= btn.y && pos.y <= btn.y + btn.h) {
    // Reset game
    state.score = 0;
    state.timeLeft = GAME_DURATION;
    state.frameCount = 0;
    state.basketCount = 0;
    state.popTexts = [];
    state.hoop.x = W / 2;
    state.hoop.y = RIM_Y_DEFAULT;
    state.hoop.targetX = W / 2;
    state.hoop.targetY = RIM_Y_DEFAULT;
    resetBall();
  }
});
```

**Step 4: Verify in browser**

Let the timer run out (or set `GAME_DURATION = 5` temporarily). Game over overlay should appear with score and Play Again button. Clicking Play Again restarts cleanly.

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: game over screen with play again button"
```

---

### Task 9: Polish Pass — Pixel Art Details + Feel

**Files:**
- Modify: `index.html`

**Step 1: Add "MISS" pop text when ball goes out of bounds**

In `update()`, where we currently do `state.phase = 'resetting'` after the ball goes out of bounds (before `setTimeout`), add:

```javascript
state.popTexts.push({
  text: 'MISS',
  x: state.ball.x,
  y: state.ball.y,
  alpha: 1,
  dy: -1,
});
```

**Step 2: Add pixel "star" burst on score**

Add this function:

```javascript
function spawnStars(x, y) {
  for (let i = 0; i < 8; i++) {
    const angle = (i / 8) * Math.PI * 2;
    state.popTexts.push({
      text: '★',
      x: x + Math.cos(angle) * 20,
      y: y + Math.sin(angle) * 20,
      alpha: 1,
      dy: -1.2 - Math.random(),
    });
  }
}
```

Call `spawnStars(hoopX, hoopY)` right after adding the score pop text in the net collision block.

**Step 3: Add ball shadow**

In `drawBall()`, draw a shadow ellipse before the ball:

```javascript
// Draw shadow under ball (only when ball is in lower half of screen)
if (state.ball.y > H * 0.5) {
  const shadowY = H - 14;
  const shadowScale = 1 - (shadowY - state.ball.y) / (H * 0.5);
  ctx.fillStyle = 'rgba(0,0,0,0.3)';
  ctx.beginPath();
  ctx.ellipse(state.ball.x, shadowY, BALL_RADIUS * Math.max(0.2, shadowScale), 5, 0, 0, Math.PI * 2);
  ctx.fill();
}
```

**Step 4: Add a subtle "DRAG TO SHOOT" prompt when idle**

In `draw()`, after `drawBall()`:

```javascript
if (state.phase === 'idle' && state.timeLeft > 0) {
  const font = '"Press Start 2P", monospace';
  ctx.font = '8px ' + font;
  ctx.fillStyle = 'rgba(255,255,255,0.5)';
  ctx.textAlign = 'center';
  ctx.fillText('DRAG TO SHOOT', W / 2, BALL_START_Y + 38);
  ctx.textAlign = 'left';
}
```

**Step 5: Verify in browser**

Play a few rounds — verify miss text appears, stars burst on baskets, shadow tracks ball, idle prompt shows.

**Step 6: Commit**

```bash
git add index.html
git commit -m "feat: polish - miss text, star burst, ball shadow, idle prompt"
```

---

### Task 10: Final Review + Open

**Step 1: Remove any debug `console.log` lines**

Search the file for `console.log` and delete them.

**Step 2: Final play-through checklist**

- [ ] Ball spawns at center-bottom
- [ ] Drag shows rubber band + trajectory dots
- [ ] Release fires ball with correct direction and power
- [ ] Ball arcs with gravity
- [ ] Rim collision bounces ball realistically
- [ ] Swish scores 3, rim-in scores 2
- [ ] Hoop moves after each basket
- [ ] Every 5 baskets hoop changes height
- [ ] Timer counts down from 90
- [ ] Timer turns red at 10s
- [ ] Game over screen appears at 0
- [ ] Best score persists across refreshes
- [ ] Play Again resets everything
- [ ] No JS errors in browser console

**Step 3: Final commit**

```bash
git add index.html
git commit -m "feat: complete retro hoops game - single file HTML5 canvas arcade"
```

**Step 4: Open the finished game**

```bash
open /Users/carlray/VS_Code/RetroHoops/index.html
```
