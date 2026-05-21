# Gelato Comic Animation — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** A single-file HTML/Canvas 2D comic animation of a fat man chasing a runaway gelato swirl, looping forever.

**Architecture:** Single `index.html` with inline CSS and JS. All characters drawn procedurally via Canvas 2D API. A frame-based state machine drives the scene sequence. No assets, no frameworks.

**Tech Stack:** HTML5, CSS, Canvas 2D API (`requestAnimationFrame`)

---

### Task 1: Scaffold the HTML + Canvas Shell

**Files:**
- Create: `index.html`

- [ ] **Step 1: Write the HTML scaffold with full-window canvas**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gelato Chase!</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { background: #fff; overflow: hidden; }
  canvas {
    display: block;
    width: 100vw;
    height: 100vh;
  }
</style>
</head>
<body>
<canvas id="c"></canvas>
<script>
</script>
</body>
</html>
```

- [ ] **Step 2: Add canvas setup and resize handler**

```js
const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');

function resize() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}
window.addEventListener('resize', resize);
resize();
```

- [ ] **Step 3: Add main loop skeleton with requestAnimationFrame**

```js
let frame = 0;

function loop() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  // draw calls go here
  frame++;
  requestAnimationFrame(loop);
}

loop();
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "chore: scaffold canvas shell"
```

---

### Task 2: Implement the Gelato Swirl Drawing

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add a function to draw the gelato swirl**

Insert inside `<script>` before `loop()`:

```js
function drawGelato(ctx, x, y, scale) {
  const s = scale || 1;
  // Cone
  ctx.fillStyle = '#D2691E';
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 3;
  ctx.beginPath();
  ctx.moveTo(x - 20*s, y + 40*s);
  ctx.lineTo(x + 20*s, y + 40*s);
  ctx.lineTo(x, y + 10*s);
  ctx.closePath();
  ctx.fill();
  ctx.stroke();
  // Swirl body — three overlaid ovals
  const swirlColors = ['#FFB6C1', '#FFC0CB', '#FF69B4'];
  for (let i = 0; i < 3; i++) {
    ctx.fillStyle = swirlColors[i];
    ctx.beginPath();
    ctx.ellipse(x, y - 10*s + i*12*s, 25*s - i*3*s, 15*s - i*2*s, 0, 0, Math.PI * 2);
    ctx.fill();
    ctx.stroke();
  }
  // Cherry on top
  ctx.fillStyle = '#FF0000';
  ctx.beginPath();
  ctx.arc(x, y - 18*s, 6*s, 0, Math.PI * 2);
  ctx.fill();
  ctx.stroke();
}
```

- [ ] **Step 2: Add legs drawing function for when gelato runs**

```js
function drawGelatoLegs(ctx, x, y, scale, phase) {
  const s = scale || 1;
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 3;
  // Two stick legs with alternating walk cycle
  const legOffset = Math.sin(phase) * 4;
  ctx.beginPath();
  ctx.moveTo(x - 8*s, y + 40*s);
  ctx.lineTo(x - 12*s - legOffset, y + 55*s);
  ctx.moveTo(x + 8*s, y + 40*s);
  ctx.lineTo(x + 12*s + legOffset, y + 55*s);
  ctx.stroke();
  // Tiny feet
  ctx.fillStyle = '#000';
  ctx.beginPath();
  ctx.arc(x - 12*s - legOffset, y + 55*s, 3*s, 0, Math.PI * 2);
  ctx.fill();
  ctx.beginPath();
  ctx.arc(x + 12*s + legOffset, y + 55*s, 3*s, 0, Math.PI * 2);
  ctx.fill();
}
```

- [ ] **Step 3: Test by drawing gelato on canvas**

Add to `loop()`:
```js
drawGelato(ctx, canvas.width * 0.7, canvas.height * 0.4, 1.5);
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add gelato swirl and legs drawing"
```

---

### Task 3: Implement the Man Character Drawing

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add man drawing function with expression support**

```js
function drawMan(ctx, x, y, scale, expression, walkPhase) {
  const s = scale || 1;
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 3;

  // Body — big round belly
  ctx.fillStyle = '#4A90D9';
  ctx.beginPath();
  ctx.ellipse(x, y - 10*s, 30*s, 35*s, 0, 0, Math.PI * 2);
  ctx.fill();
  ctx.stroke();

  // Head
  ctx.fillStyle = '#FFDAB9';
  ctx.beginPath();
  ctx.arc(x, y - 48*s, 18*s, 0, Math.PI * 2);
  ctx.fill();
  ctx.stroke();

  // Legs (walking)
  const legSwing = Math.sin(walkPhase) * 8;
  ctx.beginPath();
  ctx.moveTo(x - 10*s, y + 26*s);
  ctx.lineTo(x - 14*s - legSwing, y + 48*s);
  ctx.moveTo(x + 10*s, y + 26*s);
  ctx.lineTo(x + 14*s + legSwing, y + 48*s);
  ctx.stroke();

  // Arms
  ctx.beginPath();
  ctx.moveTo(x - 28*s, y - 5*s);
  ctx.lineTo(x - 40*s, y - 15*s);
  ctx.moveTo(x + 28*s, y - 5*s);
  ctx.lineTo(x + 40*s, y - 15*s);
  ctx.stroke();

  // Face
  if (expression === 'happy') {
    // Eyes — closed happy arches
    ctx.beginPath();
    ctx.arc(x - 7*s, y - 52*s, 3*s, Math.PI, 0);
    ctx.arc(x + 7*s, y - 52*s, 3*s, Math.PI, 0);
    ctx.stroke();
    // Smile
    ctx.beginPath();
    ctx.arc(x, y - 42*s, 6*s, 0.1, Math.PI - 0.1);
    ctx.stroke();
  } else if (expression === 'shock') {
    // Eyes — wide open
    ctx.fillStyle = '#fff';
    ctx.beginPath();
    ctx.arc(x - 7*s, y - 52*s, 5*s, 0, Math.PI * 2);
    ctx.fill();
    ctx.stroke();
    ctx.beginPath();
    ctx.arc(x + 7*s, y - 52*s, 5*s, 0, Math.PI * 2);
    ctx.fill();
    ctx.stroke();
    // Pupils tiny
    ctx.fillStyle = '#000';
    ctx.beginPath();
    ctx.arc(x - 7*s, y - 52*s, 2*s, 0, Math.PI * 2);
    ctx.fill();
    ctx.beginPath();
    ctx.arc(x + 7*s, y - 52*s, 2*s, 0, Math.PI * 2);
    ctx.fill();
    // Mouth — open O
    ctx.fillStyle = '#000';
    ctx.beginPath();
    ctx.ellipse(x, y - 40*s, 5*s, 7*s, 0, 0, Math.PI * 2);
    ctx.fill();
  }
}
```

- [ ] **Step 2: Test by drawing man on canvas (alongside gelato)**

Add to `loop()`:
```js
drawMan(ctx, canvas.width * 0.2, canvas.height * 0.5, 1, 'happy', frame * 0.05);
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add man character drawing with expressions"
```

---

### Task 4: Implement Sound Effect Drawing (ZOOOOOM!)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add ZOOOOOM! text burst function**

```js
function drawZoomText(ctx, x, y, scale) {
  ctx.save();
  ctx.translate(x, y);
  ctx.scale(scale, scale);
  // Motion lines
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 3;
  for (let i = 0; i < 6; i++) {
    const angle = -0.3 + i * 0.12;
    ctx.beginPath();
    ctx.moveTo(-40, -10 + i * 4);
    ctx.lineTo(-40 - 20 * Math.cos(angle), -10 + i * 4 - 20 * Math.sin(angle));
    ctx.stroke();
  }
  // Zoom text
  ctx.fillStyle = '#FF4500';
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 4;
  ctx.font = 'bold 48px Impact, sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.strokeText('ZOOOOOM!', 0, 0);
  ctx.fillText('ZOOOOOM!', 0, 0);
  ctx.restore();
}
```

- [ ] **Step 2: Also add a puff/dust particle function for the punchline**

```js
function drawPuff(ctx, x, y, size, alpha) {
  ctx.globalAlpha = alpha;
  ctx.fillStyle = '#FFB6C1';
  ctx.beginPath();
  ctx.arc(x, y, size, 0, Math.PI * 2);
  ctx.fill();
  ctx.globalAlpha = 1;
}
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add ZOOOOOM text burst and puff particle"
```

---

### Task 5: Implement the Animation State Machine

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace the test draw calls in `loop()` with a state machine**

Replace the `loop()` function and add state constants before it:

```js
const STATE_APPROACH = 0;
const STATE_RUNNING = 1;  // gelato zooms, man chases
const STATE_PAUSE = 2;    // brief pause before restart

let state = STATE_APPROACH;
let stateTimer = 0;
const PHASE1_DURATION = 120;  // frames
const PHASE2_DURATION = 80;
const PHASE3_DURATION = 20;

// Positions
let manX, manY, gelatoX, gelatoY, zoomAlpha;
```

- [ ] **Step 2: Implement the state machine logic**

```js
function loop() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  const cx = canvas.width / 2;
  const cy = canvas.height / 2;

  // Draw ground
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(0, cy + 60);
  ctx.lineTo(canvas.width, cy + 60);
  ctx.stroke();

  if (state === STATE_APPROACH) {
    // Man waddles in from left
    manX = lerp(-100, cx - 150, easeOut(stateTimer / PHASE1_DURATION));
    manY = cy - 10;
    gelatoX = cx + 180;
    gelatoY = cy - 20;

    drawMan(ctx, manX, manY, 1, 'happy', stateTimer * 0.08);
    drawGelato(ctx, gelatoX, gelatoY, 1.5);

    stateTimer++;
    if (stateTimer >= PHASE1_DURATION) {
      state = STATE_RUNNING;
      stateTimer = 0;
    }
  } else if (state === STATE_RUNNING) {
    const t = stateTimer / PHASE2_DURATION;
    // Gelato stays a bit then zooms right
    const gelatoStay = Math.min(t * 4, 1);
    const gelatoRun = Math.max(0, (t - 0.25) / 0.75);
    gelatoX = lerp(cx + 180, canvas.width + 100, easeIn(gelatoRun));
    gelatoY = cy - 20;

    // Man reaction
    const shockStart = 0.15;
    const chaseStart = 0.35;
    let manExpr = 'happy';
    if (t > shockStart) manExpr = 'shock';
    manX = lerp(cx - 150, canvas.width + 50, easeIn(Math.max(0, (t - chaseStart) / 0.65)));

    if (t < shockStart) {
      drawGelato(ctx, gelatoX, gelatoY, 1.5);
    } else {
      // Gelato runs!
      drawGelato(ctx, gelatoX, gelatoY, 1.3);
      drawGelatoLegs(ctx, gelatoX, gelatoY, 1.3, stateTimer * 0.3);
    }

    drawMan(ctx, manX, manY, 1, manExpr, stateTimer * 0.12);

    // ZOOOOOM! text appears during gelato run
    if (t > 0.25 && t < 0.6) {
      const zoomScale = 1 + Math.sin(stateTimer * 0.2) * 0.1;
      zoomAlpha = Math.min(1, (t - 0.25) * 8);
      ctx.globalAlpha = zoomAlpha;
      drawZoomText(ctx, gelatoX - 40, gelatoY - 50, zoomScale);
      ctx.globalAlpha = 1;
    }

    stateTimer++;
    if (stateTimer >= PHASE2_DURATION) {
      state = STATE_PAUSE;
      stateTimer = 0;
    }
  } else if (state === STATE_PAUSE) {
    // Fade out — draw final frame remnants + puff
    drawPuff(ctx, cx + 180, cy - 20, 12 * (1 - stateTimer / PHASE3_DURATION), 1 - stateTimer / PHASE3_DURATION);

    stateTimer++;
    if (stateTimer >= PHASE3_DURATION) {
      state = STATE_APPROACH;
      stateTimer = 0;
    }
  }

  frame++;
  requestAnimationFrame(loop);
}

function lerp(a, b, t) { return a + (b - a) * t; }
function easeOut(t) { return 1 - Math.pow(1 - t, 3); }
function easeIn(t) { return t * t; }
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: implement full animation state machine"
```

---

### Task 6: Polish — Timing, Particles, and Visual Tweaks

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add lingering "man wobbles" after chase ends in pause state**

Replace the pause state in loop:
```js
} else if (state === STATE_PAUSE) {
  // Man visible on left, wobbling
  const wobble = Math.sin(stateTimer * 0.3) * 3;
  drawMan(ctx, canvas.width * 0.15 + wobble, cy - 10, 1, 'shock', 0);
  // Puff particles where gelato was
  drawPuff(ctx, cx + 180, cy - 20, 12 * (1 - stateTimer / PHASE3_DURATION), 1 - stateTimer / PHASE3_DURATION);

  stateTimer++;
  if (stateTimer >= PHASE3_DURATION) {
    state = STATE_APPROACH;
    stateTimer = 0;
  }
}
```

- [ ] **Step 2: Add a comic-style panel border**

Add to top of `loop()`, after `ctx.clearRect`:
```js
// Comic panel border
ctx.strokeStyle = '#000';
ctx.lineWidth = 6;
ctx.strokeRect(20, 20, canvas.width - 40, canvas.height - 40);
```

- [ ] **Step 3: Tune timings — adjust frame durations if animation feels off**

(Test in browser and adjust `PHASE1_DURATION`, `PHASE2_DURATION`, `PHASE3_DURATION` as needed — 180, 100, 30 may be smoother.)

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add wobble, panel border, timing polish"
```
