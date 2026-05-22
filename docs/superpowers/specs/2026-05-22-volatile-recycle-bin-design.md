# Volatile Recycle Bin — "Throw Back" Easter Egg Design

## Overview

Replace the current Recycle Bin "reject" behavior (spit + roasts + floating icons) with a delayed "capture, charge, throw back" mechanic. Affects ghost-drag on sidebar items and minimized taskbar tabs only. Window drag-to-bin behavior remains unchanged.

## Architecture

A new `binCharge` state machine (`idle | charging | ejecting`) intercepts ghost-drag hits on the bin. The existing `startGhostDrag` / `onGhostMove` / `endGhostDrag` pipeline stays untouched — only the hit-routing changes from `rejectGhostToBin` to `binCharge.start()`.

```
ghost enters bin zone (40px radius)
  → document mouse listeners removed (existing logic)
  → ghost element stays in DOM, transitions to bin center (absorb)
  → binCharge.start(ghost, origRect)
  → bin class .bin-charging applied (2s CSS animation)
  → after 2s timeout: binCharge.eject()
  → bin class removed, playBoing(), ghost animates from bin → origRect
  → on arrival: ghost removed from DOM
```

## Changes to Existing Code

### Removed
- `rejectGhostToBin()` — entire function
- `BIN_ROASTS` array
- `makeFloatingIcon()` function (no more floating icons)
- Floating icon creation in `endGhostDrag()`
- Bin angry face / text bubble triggers from ghost path
- `rejectGhostToBin` call in `onGhostMove()` hit branch

### Modified
- `startGhostDrag()`: capture `e.currentTarget.getBoundingClientRect()` as `config.origRect`
- `onGhostMove()` hit branch: call `binCharge.start()` instead of `rejectGhostToBin()`
- `endGhostDrag()`: remove ghost on drop (no floating icon creation)
- `checkGhostReject()`: rename to `checkBinHit()` (same logic, new name)

### Added
- `binCharge` state object with methods: `start()`, `eject()`, `reset()`
- CSS `@keyframes bin-charge` animation
- Bin `.bin-charging` state class

---

## Coordinate Capture

On `mousedown` inside `startGhostDrag()`, store `e.currentTarget.getBoundingClientRect()` as `config.origRect`. This is the return target for the throw-back. The original element (sidebar item or tab) never moves — only the ghost animates.

---

## The 2-Second Charge

`binCharge.start()`:
1. Sets state to `charging`
2. Removes mouse event listeners (caller already does this)
3. Applies `.bin-charging` class to `#desktop-bin`
4. Sets a `setTimeout` for 2000ms → `binCharge.eject()`

`@keyframes bin-charge` cycles the bin icon's colors rapidly:

```
0%, 100% → normal colors (#8b5e3c / #7a4e3a)
10% → bright red (#ff0000) with inverted borders
20% → neon green (#00ff00) 
30% → magenta (#ff00ff) with yellow border
40% → cyan (#00ffff) 
50% → back to red, cycling through
```

`animation: bin-charge 0.5s steps(1) infinite` for a sharp, glitchy 16-bit cycling feel that repeats 4 times over 2s.

No text bubbles, no facial animations.

---

## The Throw-Back

`binCharge.eject()`:
1. Removes `.bin-charging` class → bin snaps to normal colors
2. Plays `playBoing()`
3. Positions the ghost element at the bin's center coordinates
4. Sets ghost `transition: left 0.3s cubic-bezier(0.25, 0.1, 0.25, 1), top 0.3s cubic-bezier(...)`
5. Sets ghost `left` / `top` to `config.origRect.left` / `config.origRect.top`
6. On `transitionend` or 300ms timeout: removes ghost from DOM

The ghost shrinks/fades during the absorb phase (0.15s transition into bin) and returns at full opacity/size for the throw-back.

---

## Edge Cases

- **Double-hit**: If `binCharge.start()` is called while already `charging`, ignore (second ghost cannot queue).
- **Rapid drag-drop outside bin**: Ghost removed, no floating icon, nothing happens.
- **Bin charging + window drag proximity**: Bin stays in charge mode; window reject behavior still independent.
- **Multiple minimizes during charge**: No interaction — bin charge runs independently.

---

## Testing

- Drag sidebar item to bin → ghost absorbs → bin glitches 2s → ghost flies back → original item functional
- Drag taskbar tab to bin → same flow
- Drop ghost on desktop (not bin) → ghost removed, nothing created
- Open a window and drag to bin → original spit/reject behavior still works
- Try dragging second item while bin is charging → second ghost ignored

---

## Files Changed

- `index.html` (single file): add CSS animation, modify ghost-bin hit path, remove old reject code, add `binCharge` state machine
