# Hand-Pose Tic Tac Toe Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** A single HTML file that runs Tic Tac Toe controlled by hand gestures via webcam.

**Architecture:** Single self-contained `index.html` using CDN-loaded TensorFlow.js hand-pose-detection + MediaPipe. Game loop uses `requestAnimationFrame` — each frame detects hands, maps fingertip to grid, checks gestures, and re-renders canvas. AI runs synchronously on computer turn.

**Tech Stack:** HTML5 Canvas, CSS3, TensorFlow.js (hand-pose-detection), MediaPipe Hands, vanilla JS

---

### Task 1: HTML Structure & CSS Styling

**Files:**
- Create: `index.html`

- [ ] **Step 1: Write the HTML skeleton and CSS**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Hand Pose Tic Tac Toe</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  font-family: 'Segoe UI', Tahoma, sans-serif;
  background: #1a1a2e;
  color: #eee;
  overflow: hidden;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.top-bar {
  width: 100%;
  padding: 10px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #16213e;
  z-index: 10;
}
.top-bar select, .top-bar button {
  padding: 6px 14px;
  border: none;
  border-radius: 6px;
  background: #0f3460;
  color: #eee;
  cursor: pointer;
  font-size: 14px;
}
.top-bar select:hover, .top-bar button:hover { background: #1a5276; }
.scoreboard {
  display: flex;
  gap: 30px;
  font-size: 18px;
  font-weight: bold;
}
.scoreboard span { color: #e94560; }
.game-area {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  width: 100%;
}
#gameCanvas {
  border: 2px solid #e94560;
  border-radius: 8px;
  z-index: 2;
  max-width: 90vmin;
  max-height: 80vmin;
}
.bottom-bar {
  width: 100%;
  padding: 10px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #16213e;
  z-index: 10;
  font-size: 16px;
}
#gestureHint { color: #aaa; font-style: italic; }
#video { display: none; }
</style>
</head>
<body>
<div class="top-bar">
  <select id="modeSelect">
    <option value="1p">1 Player (vs Computer)</option>
    <option value="2p">2 Players</option>
  </select>
  <select id="difficultySelect">
    <option value="easy">Easy</option>
    <option value="mid">Mid</option>
    <option value="difficult">Difficult</option>
  </select>
  <div class="scoreboard">
    <div id="scoreLeft">Pista (X): <span id="scoreX">0</span></div>
    <div id="scoreRight">Clanker (O): <span id="scoreO">0</span></div>
  </div>
  <button id="resetBtn">Reset Scores</button>
</div>
<div class="game-area">
  <canvas id="gameCanvas" width="500" height="500"></canvas>
</div>
<div class="bottom-bar">
  <div id="turnIndicator">Your turn (X)</div>
  <div id="gestureHint">Point to hover · ✌️ to place</div>
  <div id="camStatus">📷 Loading camera...</div>
</div>
<video id="video" width="640" height="480" autoplay playsinline></video>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-core"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-converter"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs-backend-webgl"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/hand-pose-detection"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands"></script>
<script>
// All JS goes in subsequent tasks
</script>
</body>
</html>
```

- [ ] **Step 2: Verify structure**

Run: Open the file in a browser. Confirm layout renders with placeholder canvas, top bar, bottom bar. CSS looks correct. (No JS logic yet.)

---

### Task 2: Game State & Core Logic

**Files:**
- Modify: `index.html` — add JS inside the existing `<script>` tag

- [ ] **Step 1: Add game state and board management**

Append inside the `<script>` tag, before any other code:

```js
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let scoreXEl = document.getElementById('scoreX');
let scoreOEl = document.getElementById('scoreO');
const turnIndicator = document.getElementById('turnIndicator');
const gestureHint = document.getElementById('gestureHint');
const modeSelect = document.getElementById('modeSelect');
const difficultySelect = document.getElementById('difficultySelect');
const resetBtn = document.getElementById('resetBtn');
const camStatus = document.getElementById('camStatus');

const state = {
  board: Array(9).fill(null),
  currentPlayer: 'X',
  gameActive: true,
  scores: { pista: 0, clanker: 0 },
  scores2p: { p1: 0, p2: 0 },
  mode: '1p',
  difficulty: 'easy',
  turnCount: 0
};

function resetBoard() {
  state.board = Array(9).fill(null);
  state.currentPlayer = 'X';
  state.gameActive = true;
  state.turnCount = 0;
  updateTurnDisplay();
}

function checkWinner(board) {
  const lines = [
    [0,1,2],[3,4,5],[6,7,8],
    [0,3,6],[1,4,7],[2,5,8],
    [0,4,8],[2,4,6]
  ];
  for (const [a,b,c] of lines) {
    if (board[a] && board[a] === board[b] && board[a] === board[c]) return board[a];
  }
  return null;
}

function isDraw(board) { return board.every(c => c !== null); }

function handlePlacement(index) {
  if (!state.gameActive || state.board[index] !== null) return false;
  state.board[index] = state.currentPlayer;
  state.turnCount++;

  const winner = checkWinner(state.board);
  if (winner) {
    state.gameActive = false;
    updateScores(winner);
    turnIndicator.textContent = `${winner === 'X' ? getPlayerName('X') : getPlayerName('O')} wins!`;
    return true;
  }
  if (isDraw(state.board)) {
    state.gameActive = false;
    turnIndicator.textContent = 'Draw!';
    return true;
  }

  state.currentPlayer = state.currentPlayer === 'X' ? 'O' : 'X';
  updateTurnDisplay();
  return true;
}

function getPlayerName(mark) {
  if (state.mode === '1p') return mark === 'X' ? 'Pista' : 'Clanker';
  return mark === 'X' ? 'Player 1' : 'Player 2';
}

function updateTurnDisplay() {
  const name = getPlayerName(state.currentPlayer);
  turnIndicator.textContent = state.gameActive ? `${name}'s turn (${state.currentPlayer})` : turnIndicator.textContent;
}

function updateScores(winner) {
  if (state.mode === '1p') {
    if (winner === 'X') state.scores.pista++;
    else state.scores.clanker++;
  } else {
    if (winner === 'X') state.scores2p.p1++;
    else state.scores2p.p2++;
  }
  updateScoreDisplay();
}

resetBtn.addEventListener('click', () => {
  state.scores = { pista: 0, clanker: 0 };
  state.scores2p = { p1: 0, p2: 0 };
  scoreXEl.textContent = '0';
  scoreOEl.textContent = '0';
  resetBoard();
});

modeSelect.addEventListener('change', (e) => {
  state.mode = e.target.value;
  difficultySelect.style.display = state.mode === '1p' ? 'inline-block' : 'none';
  const left = document.getElementById('scoreLeft');
  const right = document.getElementById('scoreRight');
  if (state.mode === '1p') {
    left.innerHTML = 'Pista (X): <span id="scoreX">0</span>';
    right.innerHTML = 'Clanker (O): <span id="scoreO">0</span>';
    state.scores = { pista: 0, clanker: 0 };
    state.scores2p = { p1: 0, p2: 0 };
  } else {
    left.innerHTML = 'Player 1 (X): <span id="scoreX">0</span>';
    right.innerHTML = 'Player 2 (O): <span id="scoreO">0</span>';
  }
  // Re-bind score elements after innerHTML replacement
  scoreXEl = document.getElementById('scoreX');
  scoreOEl = document.getElementById('scoreO');
  updateScoreDisplay();
  resetBoard();
});

function updateScoreDisplay() {
  if (state.mode === '1p') {
    scoreXEl.textContent = state.scores.pista;
    scoreOEl.textContent = state.scores.clanker;
  } else {
    scoreXEl.textContent = state.scores2p.p1;
    scoreOEl.textContent = state.scores2p.p2;
  }
}

difficultySelect.addEventListener('change', (e) => {
  state.difficulty = e.target.value;
  resetBoard();
});

const playerNames1p = ['Pista', 'Clanker'];
const playerNames2p = ['Player 1', 'Player 2'];
```

- [ ] **Step 2: Add board rendering**

```js
const CELL_SIZE = canvas.width / 3;

function drawBackground() {
  if (video.readyState >= 2) {
    ctx.save();
    ctx.scale(-1, 1);
    ctx.translate(-canvas.width, 0);
    ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
    ctx.restore();
    ctx.fillStyle = 'rgba(26, 26, 46, 0.3)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  } else {
    ctx.fillStyle = '#1a1a2e';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  }
}

function drawBoard() {
  ctx.strokeStyle = '#e94560';
  ctx.lineWidth = 3;
  for (let i = 1; i < 3; i++) {
    ctx.beginPath();
    ctx.moveTo(i * CELL_SIZE, 0);
    ctx.lineTo(i * CELL_SIZE, canvas.height);
    ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(0, i * CELL_SIZE);
    ctx.lineTo(canvas.width, i * CELL_SIZE);
    ctx.stroke();
  }
}

function drawMarks() {
  ctx.font = `${CELL_SIZE * 0.6}px Arial`;
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  for (let i = 0; i < 9; i++) {
    if (!state.board[i]) continue;
    const col = i % 3;
    const row = Math.floor(i / 3);
    const cx = col * CELL_SIZE + CELL_SIZE / 2;
    const cy = row * CELL_SIZE + CELL_SIZE / 2;
    ctx.fillStyle = state.board[i] === 'X' ? '#00d2ff' : '#ff6b6b';
    ctx.fillText(state.board[i] === 'X' ? 'X' : 'O', cx, cy);
  }
}

function drawHover(cellIndex) {
  if (cellIndex === null || cellIndex === undefined) return;
  const col = cellIndex % 3;
  const row = Math.floor(cellIndex / 3);
  ctx.fillStyle = 'rgba(233, 69, 96, 0.15)';
  ctx.fillRect(col * CELL_SIZE, row * CELL_SIZE, CELL_SIZE, CELL_SIZE);
}

function drawCursor(x, y) {
  ctx.beginPath();
  ctx.arc(x, y, 8, 0, Math.PI * 2);
  ctx.fillStyle = 'rgba(255, 255, 255, 0.8)';
  ctx.fill();
  ctx.strokeStyle = '#e94560';
  ctx.lineWidth = 2;
  ctx.stroke();
}

function render(cellIndex, cursorX, cursorY) {
  drawBackground();
  drawBoard();
  drawHover(cellIndex);
  drawMarks();
  if (cursorX !== null) drawCursor(cursorX, cursorY);
}
```

- [ ] **Step 3: Manual smoke test**

Open the file. Click reset scores, toggle mode, toggle difficulty. Confirm the page renders and controls are interactive (no hand tracking yet).

---

### Task 3: AI Implementation

**Files:**
- Modify: `index.html` — add AI functions inside the `<script>` tag

- [ ] **Step 1: Add AI move selection**

```js
function getAvailableMoves(board) {
  return board.reduce((moves, cell, i) => {
    if (cell === null) moves.push(i);
    return moves;
  }, []);
}

function aiMove() {
  if (!state.gameActive || state.currentPlayer !== 'O' || state.mode !== '1p') return;
  const available = getAvailableMoves(state.board);
  if (available.length === 0) return;

  let move;
  if (state.difficulty === 'easy') {
    move = available[Math.floor(Math.random() * available.length)];
  } else if (state.difficulty === 'mid') {
    move = aiMid(available);
  } else {
    move = aiMinimax(state.board, 'O').index;
  }
  handlePlacement(move);
  requestAnimationFrame(gameLoop);
}

function aiMid(available) {
  const opponent = 'X';
  for (const move of available) {
    const boardCopy = [...state.board];
    boardCopy[move] = 'O';
    if (checkWinner(boardCopy) === 'O') return move;
  }
  for (const move of available) {
    const boardCopy = [...state.board];
    boardCopy[move] = opponent;
    if (checkWinner(boardCopy) === opponent) return move;
  }
  return available[Math.floor(Math.random() * available.length)];
}

function aiMinimax(board, player) {
  const available = getAvailableMoves(board);
  const winner = checkWinner(board);
  if (winner === 'X') return { score: -10 };
  if (winner === 'O') return { score: 10 };
  if (available.length === 0) return { score: 0 };

  const moves = [];
  for (const move of available) {
    const boardCopy = [...board];
    boardCopy[move] = player;
    const result = aiMinimax(boardCopy, player === 'O' ? 'X' : 'O');
    moves.push({ index: move, score: result.score });
  }

  let best = moves[0];
  for (const m of moves) {
    if (player === 'O') {
      if (m.score > best.score) best = m;
    } else {
      if (m.score < best.score) best = m;
    }
  }
  return best;
}
```

- [ ] **Step 2: Test AI with console**

Open the browser console and run:
```js
state.board = ['X', null, null, null, 'O', null, null, null, null];
state.currentPlayer = 'O';
state.difficulty = 'mid';
aiMove();
```
Check the board for a placed O. Repeat with `difficult` and confirm minimax blocks X wins.

---

### Task 4: Hand Tracking Integration

**Files:**
- Modify: `index.html` — add hand detection setup and gesture logic inside the `<script>` tag

- [ ] **Step 1: Add camera initialization**

```js
const video = document.getElementById('video');
let detector = null;
let handData = null;
let cursorX = null, cursorY = null;
let hoveredCell = null;
let canPlace = true;

async function initCamera() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
    video.srcObject = stream;
    await video.play();
    camStatus.textContent = '📷 Camera ready';
  } catch (err) {
    camStatus.textContent = '❌ Camera access denied';
    console.error(err);
  }
}

async function initDetector() {
  try {
    const model = handPoseDetection.SupportedModels.MediaPipeHands;
    detector = await handPoseDetection.createDetector(model, {
      runtime: 'mediapipe',
      solutionPath: 'https://cdn.jsdelivr.net/npm/@mediapipe/hands',
      modelType: 'full'
    });
    camStatus.textContent = '📷 Model loaded';
  } catch (err) {
    camStatus.textContent = '❌ Model failed to load';
    console.error(err);
  }
}
```

- [ ] **Step 2: Add gesture detection and hand tracking**

```js
async function detectHands() {
  if (!detector || !video || video.readyState < 2) return;
  try {
    const hands = await detector.estimateHands(video);
    handData = hands.length > 0 ? hands[0] : null;
  } catch {
    handData = null;
  }
}

function processHand() {
  if (!handData) {
    cursorX = cursorY = null;
    hoveredCell = null;
    return;
  }

  const tipIndex = handData.keypoints.find(k => k.name === 'index_finger_tip');
  const pipIndex = handData.keypoints.find(k => k.name === 'index_finger_pip');
  const tipMiddle = handData.keypoints.find(k => k.name === 'middle_finger_tip');
  const pipMiddle = handData.keypoints.find(k => k.name === 'middle_finger_pip');
  const tipRing = handData.keypoints.find(k => k.name === 'ring_finger_tip');
  const pipRing = handData.keypoints.find(k => k.name === 'ring_finger_pip');
  const tipPinky = handData.keypoints.find(k => k.name === 'pinky_finger_tip');
  const pipPinky = handData.keypoints.find(k => k.name === 'pinky_finger_pip');

  if (!tipIndex || !pipIndex) { cursorX = cursorY = null; return; }

  const videoRect = video.getBoundingClientRect();
  const canvasRect = canvas.getBoundingClientRect();

  const scaleX = canvas.width / video.videoWidth;
  const scaleY = canvas.height / video.videoHeight;
  const mirroredX = video.videoWidth - tipIndex.x;
  cursorX = mirroredX * scaleX;
  cursorY = tipIndex.y * scaleY;

  const col = Math.floor(cursorX / CELL_SIZE);
  const row = Math.floor(cursorY / CELL_SIZE);
  hoveredCell = (col >= 0 && col < 3 && row >= 0 && row < 3) ? row * 3 + col : null;

  const indexUp = tipIndex.y < pipIndex.y;
  const middleUp = tipMiddle && pipMiddle ? tipMiddle.y < pipMiddle.y : false;
  const ringDown = tipRing && pipRing ? tipRing.y >= pipRing.y : true;
  const pinkyDown = tipPinky && pipPinky ? tipPinky.y >= pipPinky.y : true;

  const twoFingers = indexUp && middleUp && ringDown && pinkyDown;
  const oneFinger = indexUp && !middleUp;

  if (twoFingers && canPlace && hoveredCell !== null && state.gameActive) {
    const playerMark = state.currentPlayer;
    const placed = handlePlacement(hoveredCell);
    if (placed) {
      canPlace = false;
      if (state.mode === '1p' && state.gameActive && state.currentPlayer === 'O') {
        setTimeout(aiMove, 300);
      }
    }
  }
  if (!middleUp) canPlace = true;

  if (oneFinger) gestureHint.textContent = '👆 Pointing — hover to target';
  else if (twoFingers) gestureHint.textContent = '✌️ Placing!';
  else gestureHint.textContent = 'Show 1 finger to point · ✌️ to place';
}
```

- [ ] **Step 3: Wire the game loop**

```js
async function gameLoop() {
  await detectHands();
  processHand();
  render(hoveredCell, cursorX, cursorY);
  requestAnimationFrame(gameLoop);
}

async function startGame() {
  await initCamera();
  await initDetector();
  resetBoard();
  gameLoop();
}

startGame();
```

- [ ] **Step 4: Quick validation**

Open the file in a browser that has a camera. Allow camera access. Confirm:
- Camera feed initializes (though the video is hidden, status should show "ready")
- Hand model loads (status changes to "Model loaded")
- Canvas renders empty grid
- Moving hand in front of camera shows a cursor dot

---

### Task 5: Integration & Polish

**Files:**
- Modify: `index.html` — final touches and edge case handling

- [ ] **Step 1: Handle game over state and restart**

```js
function handleGameOver() {
  if (state.gameActive) return;
  // Click or tap anywhere on canvas to restart
  canvas.addEventListener('click', () => {
    resetBoard();
    if (state.mode === '1p' && state.currentPlayer === 'O') {
      setTimeout(aiMove, 300);
    }
  });
}

handleGameOver();
```

But actually, let's make this cleaner — add a restart mechanism via a click handler that checks if the game is over:

Replace the above with:

```js
canvas.addEventListener('click', () => {
  if (!state.gameActive) {
    resetBoard();
    render(null, null, null);
    if (state.mode === '1p' && state.currentPlayer === 'O') {
      setTimeout(aiMove, 300);
    }
  }
});
```

- [ ] **Step 2: Clean up camera on page unload**

```js
window.addEventListener('beforeunload', () => {
  const stream = video.srcObject;
  if (stream) {
    stream.getTracks().forEach(t => t.stop());
  }
});
```

- [ ] **Step 3: Add responsive canvas sizing**

```js
function resizeCanvas() {
  const area = document.querySelector('.game-area');
  const size = Math.min(area.clientWidth * 0.9, area.clientHeight * 0.9, 500);
  canvas.style.width = `${size}px`;
  canvas.style.height = `${size}px`;
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();
```

Place this after the canvas element is referenced.

- [ ] **Step 4: Final review pass**

Open the file, test end-to-end:
1. Allow camera + model loads
2. Switch between 1P and 2P mode
3. Change difficulty levels
4. Place marks via ✌️ gesture
5. Confirm AI responds at each difficulty
6. Verify scoreboard updates after wins
7. Click canvas after game over to restart
8. Reset scores via button
