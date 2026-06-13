# ✈️ Sky Runner — Flying Game

A Flappy Bird-style side-scrolling flying game built as a single self-contained HTML file. No frameworks, no dependencies, no build step.

---

## How It Was Built

### Architecture

Everything lives in one `index.html` file with inline CSS and JavaScript. The game renders entirely on a `<canvas>` element using the Canvas 2D API, driven by a `requestAnimationFrame` loop that runs continuously — even on the idle/dead screens.

### Game Loop

Unlike the Snake game which uses `setInterval`, Sky Runner uses `requestAnimationFrame` because motion is continuous and frame-smooth animation matters:

```
requestAnimationFrame(loop)
  └── update()   — physics, spawning, collision, scoring
  └── draw()     — sky, stars, clouds, obstacles, plane, particles
```

The loop runs at all times. The `state` variable (`idle | playing | dead`) controls what `update()` actually acts on — stars and clouds animate even on the menu screen.

### Player Physics

The plane has a single vertical velocity `vy` that is modified each frame:

```
vy += thrusting ? -0.35 : +0.25   // thrust up or fall
vy *= 0.92                          // damping (air resistance)
y  += vy
```

The plane tilts visually based on `vy` (capped at ±0.4 radians), giving a natural nose-up/nose-down feel without extra state. Position is hard-clamped to `[20, H-20]` to prevent leaving the canvas.

### Input

Thrust is triggered by `ArrowUp`, `W`, or `Space` on keyboard, and `pointerdown` on touch/mouse. Both the canvas and the overlay listen for pointer events so tapping anywhere — including the start screen — works correctly. A `keys` object tracks held-down keys each frame for continuous thrust.

### Obstacle Generation

Obstacles spawn every 90 frames as a pair of vertical pillars with a randomly positioned gap:

```
gapY  = random position within safe bounds
gap   = 160px fixed gap height
```

Both pillars are drawn as a single rectangle each (top from `y=0` to `gapY`, bottom from `gapY+gap` to `H`). A cyan glow is added at the gap edges using `ctx.shadowBlur`.

### Collision Detection

Collision uses axis-aligned bounding box (AABB) logic, treating the plane as a box centered on its position:

```
inX = planeRight > obstacleLeft  AND  planeLeft < obstacleRight
inY = planeBottom > gapY + gap   OR   planeTop < gapY
```

A hit is registered only when both conditions are true simultaneously.

### Difficulty Scaling

Speed increases linearly with score:

```
speed = 3 + score × 0.02
```

Score ticks up once every 8 frames. Obstacles and stars both scroll at `speed`; clouds scroll at a slower `speed × 0.2` to create a layered parallax depth effect.

### Particle System

Two independent particle emitters run during play:

- **Engine exhaust** — orange particles emitted every 3 frames from the rear of the plane, with slight random spread and a leftward drift.
- **Explosion burst** — 6 red particles emitted in random directions on death (`spawnParticle`), using a separate burst function.

All particles carry a `life` counter that drives both their remaining duration and their alpha transparency (`life / 40`).

### Trail

Every 2 frames a trail point is pushed at the plane's rear. Each trail point decays over 30 frames, rendering as a shrinking orange circle with alpha proportional to remaining life.

### Rendering Layers

The scene is drawn back-to-front each frame:

1. Sky gradient (`createLinearGradient`, dark navy to slate)
2. Stars (blinking via `sin(frame)` alpha)
3. Clouds (semi-transparent white ellipses at `globalAlpha: 0.08`)
4. Obstacles (cyan gradient pillars with glow)
5. Engine trail
6. Plane (body + wings + tail, tilted by `vy`)
7. Particles

### State Machine

```
idle  ──(action)──▶  playing  ──(collision)──▶  dead
 ▲                                                │
 └──────────────────(action after 30 frames)──────┘
```

The 30-frame cooldown on `dead → playing` prevents accidentally restarting the instant the death screen appears.

---

## How to Run

No installation or server required.

1. Download the file:
   ```
   index.html
   ```

2. Open `index.html` in any modern browser (Chrome, Firefox, Edge, Safari).

3. Press `Space` or tap the screen to start.

That's it.

---

## Controls

| Input | Action |
|---|---|
| `Space` / `ArrowUp` / `W` | Thrust upward (hold) |
| Tap / Click | Thrust (mobile) |
| `Space` or tap | Start / Restart |
