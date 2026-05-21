# Hand Gesture Rock-Paper-Scissors Game — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file HTML game where the player uses hand gestures via webcam to play Rock-Paper-Scissors against a random AI opponent (Clanker), best of 3.

**Architecture:** Single `index.html` with inline CSS and JS. TensorFlow.js MediaPipe Hands loaded from CDN for 21-point hand landmark detection. A game state machine manages IDLE → COUNTDOWN → SNAPSHOT → SHOW_RESULT → GAME_OVER transitions. Gesture classification compares fingertip-to-PIP y-coordinates with 3-frame consistency.

**Tech Stack:** Vanilla HTML/CSS/JS, `@tensorflow-models/hand-pose-detection` (CDN), `@mediapipe/hands` (CDN), `@tensorflow/tfjs-core` + `@tensorflow/tfjs-backend-webgl` (CDN)

---

### Task 1: HTML Structure and CSS Layout

**Files:**
- Create: `index.html` (lines 1–~120, HTML skeleton + `<style>` block)

- [ ] **Step 1: Write the HTML skeleton with all UI containers**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Hand Gesture RPS</title>
  <!-- CSS loaded inline below -->
  <style>
    /* styles in next step */
  </style>
</head>
<body>

  <!-- Top bar: scoreboard -->
  <div id="scoreboard">
    <div class="player-name">PISTA</div>
    <div class="vs">VS</div>
    <div class="player-name">CLANKER</div>
    <div id="score">0 - 0</div>
  </div>

  <!-- Main content: left = camera, right = game display -->
  <div id="main-content">
    <div id="camera-panel">
      <video id="webcam" autoplay playsinline></video>
      <canvas id="overlay"></canvas>
    </div>
    <div id="game-panel">
      <div id="countdown"></div>
      <div id="result-area">
        <div id="player-move" class="move-display"></div>
        <div id="result-text"></div>
        <div id="ai-move" class="move-display"></div>
      </div>
      <div id="idle-prompt">Show your hand to play</div>
      <div id="loading-spinner">Loading model...</div>
    </div>
  </div>

  <!-- Bottom bar -->
  <div id="bottom-bar">
    <div id="round-info">Best of 3</div>
    <button id="play-again" style="display:none">Play Again</button>
  </div>

  <!-- JS at bottom of body -->
  <script>
    // Tasks 2–5 go here
  </script>
</body>
</html>
```

- [ ] **Step 2: Write the CSS**

```css
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  font-family: 'Segoe UI', sans-serif;
  background: #1a1a2e;
  color: #eee;
  height: 100vh;
  display: flex;
  flex-direction: column;
  user-select: none;
}

/* Scoreboard */
#scoreboard {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  padding: 16px;
  background: #16213e;
  border-bottom: 2px solid #0f3460;
}
.player-name { font-size: 24px; font-weight: bold; color: #e94560; }
.player-name:last-of-type { color: #f5a623; }
.vs { font-size: 18px; color: #888; }
#score { font-size: 28px; font-weight: bold; color: #fff; margin-left: 20px; }

/* Main content */
#main-content {
  flex: 1;
  display: flex;
  overflow: hidden;
}

/* Camera panel */
#camera-panel {
  flex: 1;
  position: relative;
  background: #000;
  display: flex;
  align-items: center;
  justify-content: center;
}
#webcam {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transform: scaleX(-1);
}
#overlay {
  position: absolute;
  top: 0; left: 0;
  width: 100%;
  height: 100%;
  transform: scaleX(-1);
  pointer-events: none;
}

/* Game panel */
#game-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  background: #1a1a2e;
}
#countdown {
  font-size: 80px;
  font-weight: bold;
  color: #e94560;
  min-height: 100px;
}
#result-area {
  text-align: center;
  display: none;
}
#result-area.visible { display: block; }
.move-display { font-size: 36px; margin: 10px 0; }
#result-text { font-size: 32px; font-weight: bold; margin: 20px 0; color: #f5a623; }
#idle-prompt { font-size: 28px; color: #888; animation: pulse 1.5s ease-in-out infinite; }
@keyframes pulse { 0%, 100% { opacity: 0.4; } 50% { opacity: 1; } }
#loading-spinner { font-size: 20px; color: #888; display: none; }

/* Bottom bar */
#bottom-bar {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  padding: 12px;
  background: #16213e;
  border-top: 2px solid #0f3460;
}
#round-info { font-size: 18px; color: #888; }
#play-again {
  padding: 10px 30px;
  font-size: 18px;
  background: #e94560;
  color: #fff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
#play-again:hover { background: #d63851; }
```

- [ ] **Step 3: Verify layout**

Run: Open `index.html` in browser (via local HTTP server, e.g. `npx serve .` or `python -m http.server`)
Expected: Dark-themed layout with scoreboard, camera panel, game panel, bottom bar. All containers visible.

---

### Task 2: Webcam Setup and TensorFlow.js Initialization

**Files:**
- Modify: `index.html` (add `<script>` tags for CDN deps and webcam + model loading code)

- [ ] **Step 1: Add CDN script tags in `<head>`**

```html
<!-- Before the closing </head> tag, add: -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-core"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-converter"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-backend-webgl"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/hand-pose-detection"></script>
```

- [ ] **Step 2: Add webcam startup and model loading code inside `<script>` at end of body**

Replace the empty `<script></script>` block with:

```javascript
const video = document.getElementById('webcam');
const overlay = document.getElementById('overlay');
const ctx = overlay.getContext('2d');
const countdownEl = document.getElementById('countdown');
const resultArea = document.getElementById('result-area');
const playerMoveEl = document.getElementById('player-move');
const aiMoveEl = document.getElementById('ai-move');
const resultTextEl = document.getElementById('result-text');
const idlePrompt = document.getElementById('idle-prompt');
const loadingSpinner = document.getElementById('loading-spinner');
const scoreEl = document.getElementById('score');
const roundInfoEl = document.getElementById('round-info');
const playAgainBtn = document.getElementById('play-again');

let detector = null;

// Start webcam
async function startCamera() {
  const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'user' } });
  video.srcObject = stream;
  await video.play();
}

// Load TF.js hand pose model
async function loadModel() {
  loadingSpinner.style.display = 'block';
  const model = handPoseDetection.SupportedModels.MediaPipeHands;
  const config = {
    runtime: 'mediapipe',
    solutionPath: 'https://cdn.jsdelivr.net/npm/@mediapipe/hands',
    modelType: 'lite',
  };
  detector = await handPoseDetection.createDetector(model, config);
  loadingSpinner.style.display = 'none';
}

async function init() {
  await startCamera();
  await loadModel();
  // Match overlay size to video
  overlay.width = video.videoWidth;
  overlay.height = video.videoHeight;
  startGameLoop();
}

init();
```

- [ ] **Step 3: Add `startGameLoop` stub**

```javascript
function startGameLoop() {
  idlePrompt.style.display = 'block';
  requestAnimationFrame(gameLoop);
}

async function gameLoop() {
  // detection + game logic in Tasks 3-5
  requestAnimationFrame(gameLoop);
}
```

- [ ] **Step 4: Verify camera + model load**

Open `index.html` via HTTP server, allow camera permission.
Expected: Webcam feed visible in left panel. After a moment "Loading model..." disappears. Console shows no errors.

---

### Task 3: Hand Detection Loop and Gesture Classification

**Files:**
- Modify: `index.html` (flesh out `gameLoop` with detection and classification)

- [ ] **Step 1: Add hand detection and landmark extraction in `gameLoop`**

```javascript
let latestGesture = null;
let gestureFrameCount = 0;
const REQUIRED_FRAMES = 3;

async function detectHands() {
  if (!detector) return null;
  const hands = await detector.estimateHands(video);
  if (hands.length === 0) {
    gestureFrameCount = 0;
    latestGesture = null;
    return null;
  }
  return hands[0].keypoints;
}

function classifyGesture(keypoints) {
  // Build lookup by name
  const kp = {};
  keypoints.forEach(k => { kp[k.name] = k; });

  const indexTip = kp['index_finger_tip'];
  const indexPip = kp['index_finger_pip'];
  const middleTip = kp['middle_finger_tip'];
  const middlePip = kp['middle_finger_pip'];
  const ringTip = kp['ring_finger_tip'];
  const ringPip = kp['ring_finger_pip'];
  const pinkyTip = kp['pinky_finger_tip'];
  const pinkyPip = kp['pinky_finger_pip'];

  const indexUp = indexTip.y < indexPip.y;
  const middleUp = middleTip.y < middlePip.y;
  const ringUp = ringTip.y < ringPip.y;
  const pinkyUp = pinkyTip.y < pinkyPip.y;

  const fingersUp = [indexUp, middleUp, ringUp, pinkyUp];
  const count = fingersUp.filter(Boolean).length;

  if (count === 0) return 'rock';
  if (count === 4) return 'paper';
  if (indexUp && middleUp && !ringUp && !pinkyUp) return 'scissors';
  return null; // unrecognized
}

async function gameLoop() {
  const keypoints = await detectHands();

  if (keypoints) {
    const gesture = classifyGesture(keypoints);
    if (gesture && gesture === latestGesture) {
      gestureFrameCount++;
    } else if (gesture) {
      latestGesture = gesture;
      gestureFrameCount = 1;
    } else {
      gestureFrameCount = 0;
      latestGesture = null;
    }
  }

  // Draw skeleton on overlay
  ctx.clearRect(0, 0, overlay.width, overlay.height);
  if (keypoints) {
    drawSkeleton(keypoints);
    drawKeypoints(keypoints);
  }

  requestAnimationFrame(gameLoop);
}
```

- [ ] **Step 2: Add skeleton drawing helpers**

```javascript
function drawKeypoints(keypoints) {
  keypoints.forEach(kp => {
    ctx.beginPath();
    ctx.arc(kp.x, kp.y, 4, 0, 2 * Math.PI);
    ctx.fillStyle = '#e94560';
    ctx.fill();
  });
}

const CONNECTIONS = [
  ['wrist', 'thumb_cmc'], ['thumb_cmc', 'thumb_mcp'], ['thumb_mcp', 'thumb_ip'], ['thumb_ip', 'thumb_tip'],
  ['wrist', 'index_finger_mcp'], ['index_finger_mcp', 'index_finger_pip'], ['index_finger_pip', 'index_finger_dip'], ['index_finger_dip', 'index_finger_tip'],
  ['wrist', 'middle_finger_mcp'], ['middle_finger_mcp', 'middle_finger_pip'], ['middle_finger_pip', 'middle_finger_dip'], ['middle_finger_dip', 'middle_finger_tip'],
  ['wrist', 'ring_finger_mcp'], ['ring_finger_mcp', 'ring_finger_pip'], ['ring_finger_pip', 'ring_finger_dip'], ['ring_finger_dip', 'ring_finger_tip'],
  ['wrist', 'pinky_finger_mcp'], ['pinky_finger_mcp', 'pinky_finger_pip'], ['pinky_finger_pip', 'pinky_finger_dip'], ['pinky_finger_dip', 'pinky_finger_tip'],
];

function drawSkeleton(keypoints) {
  const map = {};
  keypoints.forEach(k => { map[k.name] = k; });
  ctx.strokeStyle = '#0ff';
  ctx.lineWidth = 2;
  CONNECTIONS.forEach(([a, b]) => {
    if (map[a] && map[b]) {
      ctx.beginPath();
      ctx.moveTo(map[a].x, map[a].y);
      ctx.lineTo(map[b].x, map[b].y);
      ctx.stroke();
    }
  });
}
```

- [ ] **Step 3: Verify hand detection**

Open `index.html`, show hand to camera. Expected: Cyan skeleton lines and red dots drawn over hand on the overlay. Make a fist → "rock" logged (check via console if desired).

---

### Task 4: Game State Machine

**Files:**
- Modify: `index.html` (add state machine logic inside `<script>`)

- [ ] **Step 1: Add state enum and state variables**

```javascript
const STATE = {
  IDLE: 'idle',
  COUNTDOWN: 'countdown',
  SNAPSHOT: 'snapshot',
  SHOW_RESULT: 'show_result',
  GAME_OVER: 'game_over',
};

let state = STATE.IDLE;
let countdownValue = 0;
let countdownTimer = null;
let playerGesture = null;
let aiGesture = null;
let playerScore = 0;
let aiScore = 0;
let roundNumber = 0;
```

- [ ] **Step 2: Add state transition functions**

```javascript
const GESTURE_EMOJI = { rock: '✊', paper: '✋', scissors: '✌️' };

function transitionToIdle() {
  state = STATE.IDLE;
  countdownEl.textContent = '';
  resultArea.classList.remove('visible');
  idlePrompt.style.display = 'block';
  if (countdownTimer) { clearInterval(countdownTimer); countdownTimer = null; }
}

function startCountdown() {
  state = STATE.COUNTDOWN;
  idlePrompt.style.display = 'none';
  resultArea.classList.remove('visible');
  countdownValue = 4; // Ready, 3, 2, 1
  countdownEl.textContent = 'Ready';

  const labels = ['Ready', '3', '2', '1'];
  let idx = 0;
  countdownTimer = setInterval(() => {
    idx++;
    if (idx >= labels.length) {
      clearInterval(countdownTimer);
      countdownTimer = null;
      takeSnapshot();
      return;
    }
    countdownEl.textContent = labels[idx];
  }, 1000);
}

function takeSnapshot() {
  state = STATE.SNAPSHOT;
  // Use the latest consistent gesture
  if (gestureFrameCount >= REQUIRED_FRAMES && latestGesture) {
    playerGesture = latestGesture;
  } else {
    // No valid gesture — go back to idle
    transitionToIdle();
    return;
  }

  // AI picks random move
  const moves = ['rock', 'paper', 'scissors'];
  aiGesture = moves[Math.floor(Math.random() * 3)];

  showResult();
}
```

- [ ] **Step 3: Integrate state machine into `gameLoop`**

Replace the `gameLoop` body with:

```javascript
async function gameLoop() {
  const keypoints = await detectHands();

  ctx.clearRect(0, 0, overlay.width, overlay.height);
  if (keypoints) {
    drawSkeleton(keypoints);
    drawKeypoints(keypoints);
  }

  const handDetected = keypoints !== null && gestureFrameCount >= REQUIRED_FRAMES && latestGesture !== null;

  switch (state) {
    case STATE.IDLE:
      if (handDetected) {
        startCountdown();
      }
      break;

    case STATE.COUNTDOWN:
      if (!handDetected) {
        // Hand lost — cancel countdown
        transitionToIdle();
      }
      break;

    case STATE.SNAPSHOT:
    case STATE.SHOW_RESULT:
    case STATE.GAME_OVER:
      // No detection needed during these states
      break;
  }

  requestAnimationFrame(gameLoop);
}
```

- [ ] **Step 4: Verify state transitions**

Open browser. Show hand → countdown should start ("Ready", "3", "2", "1"). Remove hand during countdown → should cancel back to idle. Show hand through full countdown → should transition to snapshot.

---

### Task 5: Result Display, Scoring, and Game Over

**Files:**
- Modify: `index.html` (add `showResult`, round logic, and game over)

- [ ] **Step 1: Add `showResult` function**

```javascript
function showResult() {
  state = STATE.SHOW_RESULT;
  countdownEl.textContent = '';
  resultArea.classList.add('visible');

  playerMoveEl.textContent = `Pista: ${GESTURE_EMOJI[playerGesture]}`;
  aiMoveEl.textContent = `Clanker: ${GESTURE_EMOJI[aiGesture]}`;

  roundNumber++;

  // Determine round winner
  const beats = { rock: 'scissors', scissors: 'paper', paper: 'rock' };
  let roundResult;
  if (playerGesture === aiGesture) {
    roundResult = 'draw';
    resultTextEl.textContent = 'Draw!';
  } else if (beats[playerGesture] === aiGesture) {
    roundResult = 'player';
    playerScore++;
    resultTextEl.textContent = 'You Win! 🎉';
  } else {
    roundResult = 'ai';
    aiScore++;
    resultTextEl.textContent = 'Clanker Wins!';
  }

  scoreEl.textContent = `${playerScore} - ${aiScore}`;
  roundInfoEl.textContent = `Round ${roundNumber} of 3`;

  // After 3 seconds, check for game over or next round
  setTimeout(() => {
    if (playerScore >= 2 || aiScore >= 2) {
      showGameOver();
    } else {
      transitionToIdle();
    }
  }, 3000);
}
```

- [ ] **Step 2: Add `showGameOver` function**

```javascript
function showGameOver() {
  state = STATE.GAME_OVER;
  resultArea.classList.add('visible');

  if (playerScore > aiScore) {
    resultTextEl.textContent = '🏆 Pista is the Champion! 🏆';
  } else {
    resultTextEl.textContent = '🤖 Clanker is the Champion! 🤖';
  }

  playerMoveEl.textContent = '';
  aiMoveEl.textContent = '';
  playAgainBtn.style.display = 'inline-block';
}
```

- [ ] **Step 3: Add play-again handler**

```javascript
playAgainBtn.addEventListener('click', () => {
  playerScore = 0;
  aiScore = 0;
  roundNumber = 0;
  scoreEl.textContent = '0 - 0';
  playAgainBtn.style.display = 'none';
  resultArea.classList.remove('visible');
  roundInfoEl.textContent = 'Best of 3';
  transitionToIdle();
});
```

- [ ] **Step 4: Verify full game flow**

Open browser. Play through a full best-of-3 game.
Expected:
- Show hand → countdown
- Gesture captured → result shown (both moves + round winner)
- Score updates after each round
- After someone wins 2 rounds → champion screen with Play Again button
- Play Again resets everything

---

### Task 6: Edge Cases and Polish

**Files:**
- Modify: `index.html` (add error handling and polish)

- [ ] **Step 1: Handle camera permission denial**

Wrap `startCamera` in try/catch:

```javascript
async function startCamera() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'user' } });
    video.srcObject = stream;
    await video.play();
  } catch (err) {
    idlePrompt.textContent = 'Camera access denied. Please allow camera permissions.';
    loadingSpinner.style.display = 'none';
    throw err;
  }
}
```

- [ ] **Step 2: Handle no-hand idle prompt text**

Already handled — idle prompt says "Show your hand to play" with a pulse animation.

- [ ] **Step 3: Resize overlay when video metadata loads**

```javascript
video.addEventListener('loadedmetadata', () => {
  overlay.width = video.videoWidth;
  overlay.height = video.videoHeight;
});
```

Add this inside `init()` before calling `startCamera()`.

- [ ] **Step 4: Verify edge cases**

Test each:
- Deny camera → error message shown
- Hide hand during countdown → cancels to idle
- Make a weird gesture (e.g. only 1 finger) → stays in idle/countdown
- Play a full game → scores correct, game over triggers at 2 wins
- Play Again → scores reset, fresh game
