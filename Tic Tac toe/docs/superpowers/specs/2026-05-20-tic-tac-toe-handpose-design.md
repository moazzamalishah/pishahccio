# Hand-Pose Tic Tac Toe — Design Spec

## Overview

A single HTML file that runs a Tic Tac Toe (3×3) game in the browser, controlled via hand gestures through the webcam. Uses TensorFlow.js hand-pose-detection with MediaPipe runtime.

## Game Modes

- **1 Player (vs Computer):** Player "Pista" (X) vs Computer "Clanker" (O). Three difficulty levels.
- **2 Player Local:** Player 1 (X) vs Player 2 (O). Same gesture rules; turns alternate after each placement.

## Difficulty Levels (1P Mode)

| Level | Behavior |
|-------|----------|
| Easy | Picks a random empty cell |
| Mid | Wins if possible → blocks opponent win → random |
| Difficult | Full minimax — optimal play, never loses |

## Gesture Controls

Using the 21-keypoint MediaPipe hand model (keypoint names and y-coordinates in pixel space):

- **Index only** (index_finger_tip.y < index_finger_pip.y AND middle_finger_tip.y >= middle_finger_pip.y AND thumb_tip, ring, pinky tips below their PIP joints) → cursor mode — a dot follows the fingertip on screen, hovered cell highlights
- **Index + Middle** (index_finger_tip.y < index_finger_pip.y AND middle_finger_tip.y < middle_finger_pip.y AND ring_tip.y >= ring_finger_pip.y AND pinky_tip.y >= pinky_finger_pip.y) → triggers placement on hovered cell
- **Release guard:** A `canPlace` flag prevents re-triggering. It resets to `false` on placement and only resets to `true` when middle finger goes back down (both conditions: index+middle no longer both extended). This prevents accidentally placing multiple marks from a single gesture hold.

## Layout (visual zones)

1. **Top bar:** Mode toggle (1P/2P), difficulty select (hidden in 2P), scoreboard
2. **Center:** Camera feed (background) with 3×3 grid overlaid on top; fingertip cursor rendered as a dot
3. **Bottom:** Turn indicator, gesture hints, game status messages

## Data Flow

```
requestAnimationFrame loop
  ├─ Detect hands via detector.estimateHands(video)
  ├─ Get index fingertip (x, y) in pixel space
  ├─ Map (x, y) → grid cell (if within board bounds)
  ├─ Highlight hovered cell
  ├─ Check: index+middle extended?
  │    └─ Yes → if cell empty and game active → place mark
  │         └─ Switch turn, check win/draw, run AI if computer turn
  └─ Render board + cursor + UI
```

## AI Implementation

- **Easy:** `Math.floor(Math.random() * emptyCells.length)`
- **Mid:** Check immediate win → check block → random fallback
- **Difficult:** Minimax with alpha-beta pruning, explores all moves, picks best

## State

```
{ board: string[][], currentPlayer: 'X'|'O', turn: number,
  scores: { pista: 0, clanker: 0, p1: 0, p2: 0 },
  mode: '1p'|'2p', difficulty: 'easy'|'mid'|'difficult',
  status: 'playing'|'won'|'draw', winner: null|'X'|'O' }
```

## Dependencies (CDN)

- `@tensorflow/tfjs-core`, `@tensorflow/tfjs-converter`, `@tensorflow/tfjs-backend-webgl`
- `@tensorflow-models/hand-pose-detection`
- `@mediapipe/hands`

## Win Detection

Standard: check all 3 rows, 3 columns, and 2 diagonals for 3 matching non-empty marks. Draw when 9 turns pass with no winner.

## Scoreboard

Persistent within session. After each game, winner's score increments. Reset button available.
