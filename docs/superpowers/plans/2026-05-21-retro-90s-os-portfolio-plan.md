# Retro 90s OS Portfolio — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file `index.html` at the project root that presents a retro 90s OS desktop interface with sidebar navigation, popup windows, and a synthwave background grid.

**Architecture:** Single HTML file with embedded CSS and vanilla JS. Canvas renders the perspective grid background. DOM elements handle the sidebar, taskbar, and draggable window system. Exercises load sub-project content via iframe.

**Tech Stack:** HTML5, CSS3, vanilla JS, Canvas 2D API, Google Fonts (VT323).

**Keywords:** Single-file, no-scroll 100vh/100vw, retro pixel 90s OS, synthwave grid, popup windows, live clocks.

---

### Task 1: Create the HTML Skeleton & CSS

**Files:**
- Create: `D:\Uni Home work\Obsidian\Obsidian\Website\index.html`

- [ ] **Step 1: Write the HTML skeleton and all CSS styles**

Create the complete `index.html` with embedded CSS. This is a large step — the entire file gets built at once.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Retro Portfolio</title>
<link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  font-family: 'VT323', 'Courier New', monospace;
  color: #d4c5a9;
  background: #6b1d2f;
  user-select: none;
}

#app {
  display: flex;
  flex-direction: column;
  width: 100vw;
  height: 100vh;
}

#main {
  display: flex;
  flex: 1;
  position: relative;
  overflow: hidden;
}

#canvas-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
}

/* Sidebar */
#sidebar {
  width: 200px;
  flex-shrink: 0;
  background: #4a2a1e;
  border-right: 3px solid #8b5e3c;
  padding: 16px 8px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  position: relative;
  z-index: 1;
  box-shadow: inset -2px 0 8px rgba(0,0,0,0.3);
}

.sidebar-title {
  text-align: center;
  font-size: 18px;
  color: #d4c5a9;
  padding-bottom: 12px;
  border-bottom: 2px solid #8b5e3c;
  margin-bottom: 8px;
  text-transform: uppercase;
  letter-spacing: 2px;
}

.sidebar-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 12px;
  background: #5c3828;
  border: 2px solid #7a4e3a;
  color: #d4c5a9;
  font-size: 18px;
  cursor: pointer;
  transition: background 0.15s;
  font-family: inherit;
  image-rendering: pixelated;
  box-shadow: 2px 2px 0 #3a1e12, inset 1px 1px 0 #7a5a4a;
}

.sidebar-item:hover {
  background: #7a4e3a;
  border-color: #d4c5a9;
}

.sidebar-item:active {
  box-shadow: inset 2px 2px 0 #3a1e12;
  transform: translate(1px, 1px);
}

.sidebar-item .icon {
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  background: #6b1d2f;
  border: 1px solid #8b5e3c;
  box-shadow: inset -1px -1px 0 #3a1e12;
}

/* Desktop area */
#desktop {
  flex: 1;
  position: relative;
  z-index: 1;
}

/* Taskbar */
#taskbar {
  height: 40px;
  flex-shrink: 0;
  background: #4a2a1e;
  border-top: 3px solid #8b5e3c;
  display: flex;
  align-items: center;
  padding: 0 12px;
  position: relative;
  z-index: 100;
  box-shadow: inset 0 2px 4px rgba(0,0,0,0.3);
}

#marquee-wrap {
  flex: 1;
  overflow: hidden;
  white-space: nowrap;
  font-size: 16px;
  color: #7a9e7e;
}

#marquee-text {
  display: inline-block;
  animation: scrollMarquee 20s linear infinite;
}

@keyframes scrollMarquee {
  from { transform: translateX(100%); }
  to { transform: translateX(-100%); }
}

#clocks {
  display: flex;
  gap: 16px;
  font-size: 16px;
  color: #d4c5a9;
  margin-left: 16px;
  flex-shrink: 0;
}

.clock {
  display: flex;
  align-items: center;
  gap: 4px;
}

.clock .flag {
  font-size: 14px;
}

/* Popup Window */
.window {
  position: absolute;
  width: 600px;
  height: 400px;
  background: #3a1e12;
  border: 3px solid #7a4e3a;
  box-shadow: 4px 4px 0 rgba(0,0,0,0.5), inset 0 0 0 2px #2a1208;
  display: flex;
  flex-direction: column;
  min-width: 300px;
  min-height: 200px;
  image-rendering: pixelated;
}

.window.maximized {
  width: 100% !important;
  height: calc(100vh - 40px) !important;
  top: 0 !important;
  left: 0 !important;
  transform: none !important;
}

.window-titlebar {
  display: flex;
  align-items: center;
  height: 32px;
  background: repeating-linear-gradient(
    45deg,
    #6b1d2f,
    #6b1d2f 4px,
    #8b3a4f 4px,
    #8b3a4f 8px
  );
  border-bottom: 2px solid #4a2a1e;
  flex-shrink: 0;
  padding: 0 4px;
  cursor: default;
}

.window-titlebar .win-btn {
  width: 22px;
  height: 22px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #5c3828;
  border: 2px solid #7a4e3a;
  color: #d4c5a9;
  font-size: 12px;
  cursor: pointer;
  font-family: inherit;
  box-shadow: inset -1px -1px 0 #3a1e12;
  line-height: 1;
  flex-shrink: 0;
}

.window-titlebar .win-btn:hover {
  background: #7a4e3a;
}

.window-titlebar .win-btn:active {
  box-shadow: inset 1px 1px 0 #3a1e12;
}

.window-titlebar .win-title {
  flex: 1;
  text-align: center;
  font-size: 16px;
  color: #d4c5a9;
  letter-spacing: 1px;
}

.window-titlebar .win-spacer {
  width: 22px;
  flex-shrink: 0;
}

.window-body {
  flex: 1;
  padding: 16px;
  overflow-y: auto;
  font-size: 16px;
  color: #d4c5a9;
  background: #2a1208;
}

/* Folder grid */
.folder-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
  gap: 16px;
  padding: 8px;
}

.folder-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 12px 8px;
  background: #3a1e12;
  border: 2px solid #5c3828;
  cursor: pointer;
  transition: border-color 0.15s;
}

.folder-item:hover {
  border-color: #7a9e7e;
}

.folder-item .folder-icon {
  width: 48px;
  height: 36px;
  background: #7a9e7e;
  border: 2px solid #5c3828;
  border-radius: 2px 2px 0 0;
  position: relative;
  box-shadow: inset -2px -2px 0 #4a6e5e;
}

.folder-item .folder-icon::before {
  content: '';
  position: absolute;
  top: -6px;
  left: -2px;
  width: 16px;
  height: 6px;
  background: #7a9e7e;
  border: 2px solid #5c3828;
  border-bottom: none;
  border-radius: 2px 2px 0 0;
}

.folder-item .folder-label {
  font-size: 12px;
  color: #d4c5a9;
  text-align: center;
  word-break: break-word;
}

/* About Me content */
.about-content p {
  margin-bottom: 12px;
  line-height: 1.5;
}

.about-content .highlight {
  color: #7a9e7e;
}

/* Professors list */
.prof-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.prof-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 12px;
  background: #3a1e12;
  border: 1px solid #5c3828;
}

.prof-item .prof-icon {
  width: 32px;
  height: 32px;
  background: #8b5e3c;
  border: 2px solid #5c3828;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
}

.prof-item .prof-name {
  font-size: 16px;
  color: #d4c5a9;
}

/* Back button */
.back-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 4px 12px;
  background: #5c3828;
  border: 2px solid #7a4e3a;
  color: #d4c5a9;
  cursor: pointer;
  font-family: inherit;
  font-size: 14px;
  margin-bottom: 12px;
}

.back-btn:hover {
  background: #7a4e3a;
}

/* Iframe for exercise content */
.exercise-frame {
  width: 100%;
  height: calc(100% - 40px);
  border: 1px solid #5c3828;
  background: #fff;
}
</style>
</head>
<body>
<div id="app">
  <div id="main">
    <canvas id="canvas-bg"></canvas>
    <div id="sidebar">
      <div class="sidebar-title">Menu</div>
      <div class="sidebar-item" data-window="about">
        <span class="icon">&#x1f464;</span>
        <span>About Me</span>
      </div>
      <div class="sidebar-item" data-window="exercises">
        <span class="icon">&#x1f4c1;</span>
        <span>Exercises</span>
      </div>
      <div class="sidebar-item" data-window="professors">
        <span class="icon">&#x1f393;</span>
        <span>Professors</span>
      </div>
    </div>
    <div id="desktop"></div>
  </div>
  <div id="taskbar">
    <div id="marquee-wrap">
      <span id="marquee-text">Welcome to my portfolio &nbsp;&nbsp;&bull;&nbsp;&nbsp; Welcome to my portfolio &nbsp;&nbsp;&bull;&nbsp;&nbsp;</span>
    </div>
    <div id="clocks">
      <div class="clock"><span class="flag">&#x1f1f5;&#x1f1f0;</span> <span id="clock-pk">00:00:00</span></div>
      <div class="clock"><span class="flag">&#x1f1ee;&#x1f1f9;</span> <span id="clock-it">00:00:00</span></div>
    </div>
  </div>
</div>
<script>
// All JavaScript goes here
</script>
</body>
</html>
```

- [ ] **Step 2: Verify the file was created**

Run: `Test-Path "D:\Uni Home work\Obsidian\Obsidian\Website\index.html"`
Expected: `True`

---

### Task 2: Implement Canvas Perspective Grid

**Files:**
- Modify: `D:\Uni Home work\Obsidian\Obsidian\Website\index.html` (add JS inside the `<script>` tag)

- [ ] **Step 1: Add the canvas grid drawing logic**

Replace the empty `<script>` contents with:

```javascript
const canvas = document.getElementById('canvas-bg');
const ctx = canvas.getContext('2d');
const desktop = document.getElementById('desktop');
let zIndexCounter = 100;

function resizeCanvas() {
  canvas.width = desktop.offsetWidth;
  canvas.height = desktop.offsetHeight;
  drawGrid();
}

function drawGrid() {
  const w = canvas.width;
  const h = canvas.height;
  const horizonY = h * 0.55;
  const vanishX = w / 2;
  const vanishY = horizonY;

  // Sky gradient
  const sky = ctx.createLinearGradient(0, 0, 0, horizonY);
  sky.addColorStop(0, '#2a0a12');
  sky.addColorStop(0.5, '#4a1525');
  sky.addColorStop(1, '#6b1d2f');
  ctx.fillStyle = sky;
  ctx.fillRect(0, 0, w, horizonY);

  // Ground
  ctx.fillStyle = '#1a0a08';
  ctx.fillRect(0, horizonY, w, h - horizonY);

  // Horizontal grid lines
  const lineColors = ['#7a9e7e', '#d4c5a9', '#8b5e3c'];
  for (let i = 1; i <= 20; i++) {
    const t = i / 20;
    const y = horizonY + (h - horizonY) * t * t;
    ctx.strokeStyle = lineColors[i % 3];
    ctx.lineWidth = i < 3 ? 2 : 1;
    ctx.globalAlpha = 1 - t * 0.5;
    ctx.beginPath();
    ctx.moveTo(0, y);
    ctx.lineTo(w, y);
    ctx.stroke();
  }

  // Vertical grid lines radiating from vanishing point
  const numRays = 30;
  for (let i = 0; i < numRays; i++) {
    const angle = -Math.PI / 2 - 0.8 + (i / (numRays - 1)) * 1.6;
    const dx = Math.cos(angle);
    const dy = Math.sin(angle);
    if (dy <= 0) continue;
    const tMax = dy > 0 ? (h - vanishY) / dy : 0;
    const endX = vanishX + dx * tMax;
    const endY = vanishY + dy * tMax;
    ctx.strokeStyle = lineColors[i % 3];
    ctx.lineWidth = i % 5 === 0 ? 2 : 1;
    ctx.globalAlpha = 0.6;
    ctx.beginPath();
    ctx.moveTo(vanishX, vanishY);
    ctx.lineTo(endX, endY);
    ctx.stroke();
  }

  // Sun/glow
  const grd = ctx.createRadialGradient(vanishX, vanishY, 0, vanishX, vanishY, 60);
  grd.addColorStop(0, 'rgba(212, 197, 169, 0.4)');
  grd.addColorStop(0.5, 'rgba(212, 197, 169, 0.1)');
  grd.addColorStop(1, 'rgba(212, 197, 169, 0)');
  ctx.fillStyle = grd;
  ctx.beginPath();
  ctx.arc(vanishX, vanishY, 60, 0, Math.PI * 2);
  ctx.fill();
  ctx.globalAlpha = 1;
}

window.addEventListener('resize', resizeCanvas);
resizeCanvas();
```

- [ ] **Step 2: Verify canvas renders**
No automated test — will verify visually by opening the file in a browser.

---

### Task 3: Implement Clocks and Window System

**Files:**
- Modify: `D:\Uni Home work\Obsidian\Obsidian\Website\index.html`

- [ ] **Step 1: Add clock update logic** (append after the canvas code)

```javascript
// Clock update
function updateClocks() {
  const now = new Date();
  const pk = new Date(now.toLocaleString('en-US', { timeZone: 'Asia/Karachi' }));
  const it = new Date(now.toLocaleString('en-US', { timeZone: 'Europe/Rome' }));
  document.getElementById('clock-pk').textContent = pk.toLocaleTimeString('en-US', { hour12: false });
  document.getElementById('clock-it').textContent = it.toLocaleTimeString('en-US', { hour12: false });
}
setInterval(updateClocks, 1000);
updateClocks();
```

- [ ] **Step 2: Implement the popup window creation functions** (append after clock code)

```javascript
// Window system
const WINDOW_CONTENT = {
  about: `
    <div class="about-content">
      <p>Welcome to my portfolio!</p>
      <p>I am a university student on a journey through computer science. This retro OS-themed page showcases the projects and exercises I've built along the way.</p>
      <p>From creative coding experiments to interactive web applications, each project represents a step forward in my learning.</p>
      <p class="highlight">Explore the Exercises section to see my work, or check out the Professors section to learn about the mentors guiding my path.</p>
    </div>
  `,
  exercises: null, // built dynamically
  professors: `
    <div class="prof-list">
      <div class="prof-item">
        <div class="prof-icon">A</div>
        <div class="prof-name">Professor A — Computer Science</div>
      </div>
      <div class="prof-item">
        <div class="prof-icon">B</div>
        <div class="prof-name">Professor B — Mathematics</div>
      </div>
      <div class="prof-item">
        <div class="prof-icon">C</div>
        <div class="prof-name">Professor C — Physics</div>
      </div>
    </div>
  `
};

function createWindow(type, title) {
  const existing = document.querySelector(`.window[data-type="${type}"]`);
  if (existing) {
    bringToFront(existing);
    return;
  }

  const win = document.createElement('div');
  win.className = 'window';
  win.dataset.type = type;
  win.style.zIndex = ++zIndexCounter;
  win.style.top = '50%';
  win.style.left = '50%';
  win.style.transform = 'translate(-50%, -50%)';

  win.innerHTML = `
    <div class="window-titlebar">
      <span class="win-btn win-minimize" title="Minimize">_</span>
      <span class="win-title">${title}</span>
      <span class="win-btn win-maximize" title="Maximize">&#x25a1;</span>
      <span class="win-btn win-close" title="Close">&#x2715;</span>
    </div>
    <div class="window-body"></div>
  `;

  desktop.appendChild(win);

  const body = win.querySelector('.window-body');
  if (type === 'exercises') {
    buildExerciseGrid(body);
  } else {
    body.innerHTML = WINDOW_CONTENT[type] || '';
  }

  // Event handlers
  win.querySelector('.win-close').addEventListener('click', () => win.remove());
  win.querySelector('.win-minimize').addEventListener('click', () => { win.style.display = 'none'; });
  win.querySelector('.win-maximize').addEventListener('click', () => {
    win.classList.toggle('maximized');
    if (win.classList.contains('maximized')) {
      win.style.top = '0';
      win.style.left = '0';
      win.style.transform = 'none';
    } else {
      win.style.top = '50%';
      win.style.left = '50%';
      win.style.transform = 'translate(-50%, -50%)';
    }
  });

  win.addEventListener('mousedown', () => bringToFront(win));
  bringToFront(win);
}

function bringToFront(win) {
  win.style.zIndex = ++zIndexCounter;
}

const EXERCISES = [
  { label: 'Animation', path: 'Animation/index.html' },
  { label: 'Pattern', path: 'Pattern/index.html' },
  { label: 'Gelato', path: 'Gelato/index.html' },
  { label: 'Tic Tac Toe', path: 'Tic Tac toe/index.html' },
  { label: 'Hand Gesture', path: 'Hand Gesture/index.html' },
  { label: 'Basmati', path: 'Basmati/Code Base/index.html' }
];

function buildExerciseGrid(container) {
  container.innerHTML = '';
  const grid = document.createElement('div');
  grid.className = 'folder-grid';
  EXERCISES.forEach(ex => {
    const item = document.createElement('div');
    item.className = 'folder-item';
    item.innerHTML = `
      <div class="folder-icon"></div>
      <div class="folder-label">${ex.label}</div>
    `;
    item.addEventListener('click', () => openExercise(container, ex));
    grid.appendChild(item);
  });
  container.appendChild(grid);
}

function openExercise(container, ex) {
  container.innerHTML = `
    <button class="back-btn" onclick="buildExerciseGrid(this.parentElement)">&#x2190; Back</button>
    <iframe class="exercise-frame" src="${ex.path}" title="${ex.label}"></iframe>
  `;
}
```

- [ ] **Step 3: Connect sidebar clicks to window creation** (append after window functions)

```javascript
// Sidebar handlers
document.querySelectorAll('.sidebar-item').forEach(item => {
  item.addEventListener('click', () => {
    const type = item.dataset.window;
    const titles = { about: 'About Me', exercises: 'Exercises', professors: 'Professors' };
    createWindow(type, titles[type]);
  });
});
```

---

### Task 4: Verify the Final File

- [ ] **Step 1: Verify the file size and structure**

Run: `Select-String -Path "D:\Uni Home work\Obsidian\Obsidian\Website\index.html" -Pattern "<!DOCTYPE html>"`
Expected: Should find the DOCTYPE declaration

- [ ] **Step 2: Check for any syntax issues**

The file should open in a browser without console errors. Key things to verify visually:
- Layout is 100vw x 100vh with no scrollbar
- Canvas perspective grid renders behind everything
- Sidebar shows three clickable items
- Taskbar shows marquee and two clocks
- Clicking sidebar items opens centered popup windows
- Windows can be closed (X), minimized (_), maximized (square)
- Maximized windows fill the desktop area above the taskbar
- Exercises window shows folder icons; clicking one loads an iframe with the project
- Back button in Exercises returns to folder grid
- Clocks update every second with correct timezones
