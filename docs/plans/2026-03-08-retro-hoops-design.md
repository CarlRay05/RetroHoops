# Retro Hoops — Design Doc

**Date:** 2026-03-08
**Project:** `/Users/carlray/VS_Code/RetroHoops/index.html`
**Format:** Single self-contained HTML file (no build tooling, no dependencies)

---

## Overview

A 16-bit retro arcade basketball game playable in any browser. The player drags and releases the ball (slingshot mechanic) to shoot into a hoop that moves after each successful basket. A 90-second countdown timer creates urgency. Score and best score are tracked.

---

## Visual Design

- **Canvas:** 800×550px HTML5 Canvas
- **16-bit aesthetic:** Pixel-art ball, chunky hardwood floor planks, dithered crowd silhouettes, CRT scanline overlay
- **Color palette:** Dark navy/purple background, orange rim, white net, orange basketball with black seams
- **Font:** `Press Start 2P` (Google Fonts @import, monospace fallback)
- **HUD:** Score (top-left), countdown timer (top-center), best score (top-right) — all overlaid on canvas

---

## Gameplay Mechanics

### Shooting (slingshot)
- Ball rests at center-bottom of canvas
- Mousedown on ball → drag in any direction
- Dotted trajectory arc (5–6 ghost dots via physics projection) previews the shot
- Release fires the ball with velocity inverse to drag vector
- Max drag capped at 120px

### Physics
- Gravity: `vy += 0.4` per frame at ~60fps
- Rim collision: two collision points (left/right rim edge), bounce factor `0.5`
- Backboard collision on right side
- Swish detection: ball passes through net zone without touching rim
- Ball resets to center-bottom 1s after each shot (score or miss)

### Hoop Movement
- Starts centered horizontally
- Slides to a new horizontal position (lerp) after each basket
- Every 5 baskets: also shifts vertically (simulating depth change)

### Scoring
| Result | Points |
|--------|--------|
| Swish (clean through net) | 3 |
| Off rim and in | 2 |
| Miss | 0 |

- Pop-text floats and fades: "SWISH!", "+3", "+2", "MISS"
- Best score persisted in `localStorage`

### Timer
- 90 seconds, counts down each second
- Game-over screen on 0: shows final score, best score, "PLAY AGAIN" button

---

## Code Architecture

### State Object
```javascript
const state = {
  ball: { x, y, vx, vy, dragging, launched },
  hoop: { x, y, targetX, targetY },
  score: 0,
  bestScore: 0,       // from localStorage
  timeLeft: 90,
  phase: 'idle' | 'dragging' | 'flying' | 'scored' | 'gameover',
  dragStart: { x, y },
  streak: 0,
};
```

### Game Loop
`requestAnimationFrame` drives:
1. `update()` — physics, collision detection, timer tick (every 60 frames)
2. `draw()` — court → hoop → ball → HUD → scanlines

### Key Functions
- `drawCourt()` — pixel planks, crowd silhouettes
- `drawHoop(x, y)` — rim, backboard, net
- `drawBall(x, y)` — pixel-art basketball with seams
- `drawTrajectory(dragX, dragY)` — ghost dots via physics projection
- `checkCollision()` — rim, backboard, net zone (score detection)
- `moveHoop()` — lerp toward target after each basket
- `showPopText(text, x, y)` — floating score feedback

### Approach
- HTML5 Canvas, vanilla JS, zero external runtime dependencies
- `Press Start 2P` font loaded via Google Fonts CSS @import (degrades gracefully to monospace)
