# Gelato Comic Animation — Design Spec

## Overview
A simple looping 2D comic-style web animation built as a single `index.html` with inline CSS and Canvas JS. No frameworks or external dependencies.

## Storyboard (loop sequence)

| Phase | Description | Duration (approx) |
|-------|-------------|-------------------|
| 1 | A joyful fat man waddles in from the left, arms outstretched, smiling at a giant gelato swirl on the right. | ~3s |
| 2 | The gelato swirl suddenly pops tiny stick legs. A bold "ZOOOOOM!" text burst appears. The gelato zooms off-screen to the right. | ~1.5s |
| 3 | The man's expression snaps to shock (wide eyes, open mouth). He gives chase, running off-screen right. | ~1.5s |
| 4 | Brief pause, then loop restarts from Phase 1. | ~0.5s |

## Visual Style
- **2D hand-drawn comic aesthetic** — code-drawn shapes with thick black outlines and flat colors
- **Characters:**
  - **Fat man:** Round body (circle), small legs, dot eyes, big smile → shock face (O mouth, wide eyes)
  - **Gelato swirl:** Tall soft-serve cone shape (spiraling triangles/curves) with a rounded top, on a small cone base. Gets tiny stick legs
- **Background:** Simple comic panel feel — maybe a ground line and subtle sky wash
- **Sound effects as text:** "ZOOOOOM!" in bold comic font, with motion lines
- **Colors:** Comic-book flat palette — bright, saturated

## Technical Implementation
- **Canvas:** Full-window canvas, 2D context, `requestAnimationFrame` loop
- **Animation:** Frame-based state machine driving positional tweening and sprite changes
- **Characters drawn procedurally** using Canvas path/arc/fill operations — no external image assets
- **Text rendering:** Canvas `fillText` for "ZOOOOOM!" with comic-style styling
- **Loop:** Animation restarts seamlessly after the chase phase ends

## File Structure
```
index.html          — single file, everything inline
```

## Success Criteria
- Animation plays on open in any modern browser
- Characters are visually distinct and comic-like
- Sequence is readable: approach → gelato sprouts legs → ZOOOOOM! → shock → chase → loop
- No external assets required
