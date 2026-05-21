# Gelato Squash Text Animation — Design Spec

## Overview
A standalone looping CSS + Web Audio animation of the word "GELATO" dropping, squashing flat with a THUD, pausing, then elastically recovering. Single `index.html`, no dependencies.

## Visual Style
- Massive, extra-black, blocky letters filling the screen
- Canvas-clearing bold presence on a white/light background
- Comic exaggeration: extreme squash (15% height, 140% width), slow elastic recovery with top-heavy wobble

## Animation Sequence

| Time | Event |
|------|-------|
| 0.0s | Text hidden above viewport |
| 0.0–1.0s | Drops straight down to vertical center |
| 1.0s | Instant squash — `scaleY(0.15) scaleX(1.4)`, thud sound plays |
| 1.0–2.5s | Dead pause — text sits flat, silent |
| 2.5–3.5s | Slow elastic recovery to normal with overshoot wobble |
| 3.5–4.5s | Brief hold, then loop back |

## Technical Approach
- **Text rendering:** `<h1>` with bold block font (Impact / Arial Black), styled with thick black color
- **Animation:** CSS `@keyframes` with `cubic-bezier` easing for squash/stretch physics
- **Sound:** Web Audio API — `OscillatorNode` at ~60–80Hz sine wave with fast envelope decay, routed to destination
- **Trigger:** Sound plays via `animationstart` or `animationiteration` event listener at the squash keyframe

## File Structure
```
gelato-squash/
  index.html          — single file, everything inline
```

## Success Criteria
- Word drops, flattens with realistic comic squash
- Thud sound plays exactly at impact
- Elastic recovery feels heavy and sticky
- Loops seamlessly
