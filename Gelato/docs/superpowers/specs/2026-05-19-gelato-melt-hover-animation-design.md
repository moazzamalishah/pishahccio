# Gelato Melt Hover Animation — Design Spec

## Overview
A single-file HTML page displaying "GELATO" as individual letters that melt (droop/slouch) one-by-one on hover, then reverse one-by-one when the mouse leaves.

## Visual Style
- Big bold text (Impact font) centered on screen
- Black letters on white background
- Softening effect: slight blur during melt to feel "soft"
- Thick, premium comic-like lettering

## Interaction

| Trigger | Behavior |
|---------|----------|
| Mouse hovers over word | Letters melt left to right (G → E → L → A → T → O), each taking ~0.3s |
| Mouse leaves word | Letters reverse right to left (O → T → A → L → E → G), same timing |

## Per-Letter Melt Transform
- `translateY(+40px)` — drops downward
- `scaleY(0.4)` — compresses vertically (slouch)
- `skewX(±5deg)` — slight tilt for organic feel
- `filter: blur(1px)` — softens appearance

## Technical Approach
- **Text:** 6 `<span>` elements inside a flex container, each containing one letter
- **Animation:** CSS `transition: all 0.3s ease` on each span
- **Trigger:** JavaScript `mouseenter` / `mouseleave` on the container, iterating letters with staggered `setTimeout`
- **File:** Single `index.html`, no dependencies

## File Structure
```
gelato-melt/
  index.html
```

## Success Criteria
- Letters melt one-by-one on hover in sequence
- Letters recover one-by-one on mouse leave in reverse sequence
- Smooth CSS transitions — no abrupt jumps
- Works in any modern browser
