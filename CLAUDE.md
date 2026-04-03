# RetroHoops

Single-file HTML5 Canvas basketball game with a slingshot drag-to-shoot mechanic, moving hoop, 90-second countdown, and best score via localStorage.

## Stack
- **Single file:** `index.html` — all HTML, CSS, and JS inline
- **No build step.** No frameworks. Vanilla JS + Canvas 2D API.
- **Served via:** `npx serve -p 4000`
- **Font:** Press Start 2P (Google Fonts) for arcade HUD

## Key Constants (`index.html`)
```
BALL_RADIUS = 18       BALL_START_Y = H - 110 (440)
MAX_DRAG = 150         GRAVITY = 0.4
BOUNCE = 0.5           RIM_HALF = 28
GAME_DURATION = 90     NET_HEIGHT = 40
```

## Game Loop
`requestAnimationFrame` → `update()` → `draw()`

## Scoring
- Swish (no rim): **+3 pts**
- Off rim: **+2 pts**
- Score requires ball enter from above the rim (`prevBallY <= netTop`, `vy > 2`)

## Physics Notes
- Launch velocity is **linear**: `dist * 0.155` (not quadratic)
- Backboard only deflects balls moving **downward** (`vy >= 0`)
- Hoop moves via lerp: `hoop.x += (targetX - hoop.x) * 0.05`
