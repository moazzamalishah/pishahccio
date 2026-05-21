# Sound-Reactive SVG Animation Design

## Overview

An HTML page that embeds the existing `Animation.svg` and uses live microphone input to drive fluid, wind-like animations on specific SVG elements. Built as a single self-contained HTML file.

## Audio Pipeline

1. `navigator.mediaDevices.getUserMedia` requests microphone access
2. `AudioContext` + `AnalyserNode` reads time-domain audio data in a continuous loop
3. RMS amplitude computed from the time-domain data
4. Exponential moving average smooths the raw value for fluid transitions (~300ms decay)
5. Normalized 0–1 amplitude value fed to the animation engine each frame

## Animation Engine

A `requestAnimationFrame` loop reads the smoothed amplitude and applies CSS transforms to SVG `<g>` groups:

| Element | Behavior | Transform |
|---------|----------|-----------|
| `#Air` | `opacity: 0` by default. On sound, fades to max 0.8. Individual paths oscillate in rotation (±3–8°) and y-translation at random frequencies. | `rotate(a) translateY(dy)` per child path |
| `#Towel` | Rotates counter-clockwise and shifts up-left with sine oscillation. Intensity proportional to sound level. | `rotate(-θ) translate(-x, -y)` with sine modulation |
| `#Napkin` | Same as Towel but with offset phase and smaller rotation range (±1–4°) for variety. | `rotate(-θ) translate(-x, -y)` with offset sine |
| `#Hair` | Gentle slow sine-wave rotation, lower frequency than Towel/Napkin. | `rotate(±α)` with slow sine modulation |
| `#Hand`, `#Body`, `#Wire`, `#Bucket` | No animation (static). | — |

When amplitude drops to zero, EMA smoothing naturally eases all elements back to their resting position/opacity over ~300ms.

## Responsive Layout

- SVG inlined in HTML with `viewBox="0 0 1366 768"`
- CSS: `width: 100%; height: 100vh` with `preserveAspectRatio`
- Touch-friendly (no interaction needed, always-listening)

## File Structure

Single file: `index.html` containing inline SVG, CSS, and JavaScript.

Unchanged: `Animation.svg` (original source kept as-is).

## Requirements

- Browser with Web Audio API and `getUserMedia` support (Chrome, Firefox, Safari, Edge)
- Microphone permission required
- Works on desktop and mobile browsers
