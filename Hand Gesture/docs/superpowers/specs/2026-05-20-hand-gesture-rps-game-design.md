# Hand Gesture Rock-Paper-Scissors Game

> **Date:** 2026-05-20
> **Status:** Approved Design
> **Player:** Pista
> **AI Opponent:** Clanker

## Overview

A browser-based Rock-Paper-Scissors game played via hand gestures detected through a live webcam. Uses TensorFlow.js MediaPipe Hands model to detect 21 hand landmarks and classify gestures as Rock, Paper, or Scissors. The opponent AI picks randomly. Best of 3 rounds.

## Game States

1. **IDLE** — Webcam runs continuously detecting hands. Hovering text prompts the user to show their hand.
2. **COUNTDOWN** — When a hand is detected, countdown begins: "Ready" → "3" → "2" → "1" at 1-second intervals. If hand disappears at any point, cancels back to IDLE.
3. **SNAPSHOT** — At "1" the current gesture is captured and locked. AI opponent picks randomly behind the scenes.
4. **SHOW_RESULT** — Both moves displayed (Pista vs Clanker). Round winner announced. Score updates. After ~3 seconds, either advances to GAME_OVER (if someone has 2 wins) or returns to IDLE.
5. **GAME_OVER** — Champion screen. "Play Again" button resets scores and returns to IDLE.

## Gesture Detection

MediaPipe Hands returns 21 landmarks per hand. Classification uses y-coordinate comparison (tip vs PIP) with 3-frame consistency check to prevent flickering.

| Gesture | Detection Rule |
|---|---|
| **Rock** (fist) | Index, middle, ring, pinky: `tip_y > pip_y` (folded) |
| **Paper** (open palm) | Index, middle, ring, pinky: `tip_y < pip_y` (extended) |
| **Scissors** | Index & middle: `tip_y < pip_y`; Ring & pinky: `tip_y > pip_y` |
| **None** | No hand detected (empty result from `estimateHands`) |

Unrecognized gestures (e.g. only 1 finger up, thumb-only) are treated as invalid and the game waits for a valid gesture.

## Architecture

### Dependencies (CDN)

- `@tensorflow-models/hand-pose-detection` — API
- `@mediapipe/hands` — MediaPipe runtime
- `@tensorflow/tfjs-core` + `@tensorflow/tfjs-backend-webgl` — TF.js backend

### Model Config

- Model: `MediaPipeHands`
- Runtime: `mediapipe` (better FPS with WASM + GPU)
- Model type: `lite` (prioritizes webcam frame rate)

### File Structure

Single file: `index.html` (inline CSS + JS)

No build tools, no `package.json`. Run via any local HTTP server.

## UI Layout

```
┌─────────────────────────────────┐
│      PISTA (You) vs CLANKER     │
│           Score: 0 - 0          │
├──────────┬──────────────────────┤
│          │                      │
│  Camera  │   Countdown /        │
│  Feed    │   Result Display     │
│  (with   │                      │
│  hand    │   "Ready... 3 2 1"   │
│  overlay)│   "Pista: ✊"        │
│          │   "Clanker: ✋"      │
│          │   "You Win!"         │
│          │                      │
├──────────┴──────────────────────┤
│     Round X of 3 (best of 3)    │
│     [Play Again] (end screen)   │
└─────────────────────────────────┘
```

### Details

- **Top bar:** Player name (Pista), AI name (Clanker), current score display
- **Left panel:** Live webcam feed optionally showing hand skeleton overlay
- **Right panel:** Central game display area — shows countdown, then both picks + result
- **Bottom bar:** Round counter, Play Again button (visible only on game over)

## Data Flow

1. Camera stream feeds into `detector.estimateHands()` at each animation frame
2. If hand detected and game not in COUNTDOWN, transition to COUNTDOWN
3. During COUNTDOWN, continue detecting to ensure hand stays visible
4. On SNAPSHOT, classify the last detected gesture
5. AI picks random move
6. Determine round winner: Rock > Scissors > Paper > Rock
7. If either player reaches 2 wins → GAME_OVER, else → IDLE for next round

## Edge Cases

- **Hand disappears during countdown:** Cancel immediately, return to IDLE
- **Unrecognized gesture:** Don't advance — keep showing countdown or show "show a valid hand"
- **Multiple hands detected:** Use the first detected hand
- **Camera permission denied:** Show error message with instructions
- **Very slow model loading:** Show loading spinner until model is ready
- **No hand for prolonged period:** Stay in IDLE, prompt "Show your hand to play"

## Future Considerations (Not Implementing Now)

- Sound effects
- Gesture history/replay
- Difficulty levels for AI (strategy vs random)
- Mobile responsive layout
