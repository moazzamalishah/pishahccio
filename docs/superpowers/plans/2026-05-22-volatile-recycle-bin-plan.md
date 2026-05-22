# Volatile Recycle Bin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the current Recycle Bin "reject" behavior (spit+roast+floating icons) with a delayed "capture, charge, throw back" mechanic on sidebar items and taskbar tabs.

**Architecture:** A `binCharge` state machine (`idle | charging | ejecting`) intercepts ghost-drag hits on the bin. The ghost element stays in DOM, absorbs into the bin, then after 2s of CSS color-cycling animation, flies back to the original slot position and is removed.

**Tech Stack:** Vanilla JS, CSS `@keyframes`, all in single `index.html`.

---

### Task 1: Remove old reject-bin code

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Delete `rejectGhostToBin` function**

Remove the entire `rejectGhostToBin` function (lines ~2339-2369 in the current file). It creates floating icons, shows roasts, and spits items away — all unwanted.

- [ ] **Step 2: Delete `BIN_ROASTS` array**

Remove `const BIN_ROASTS = [...]` (lines ~2229-2233).

- [ ] **Step 3: Delete `makeFloatingIcon` function**

Remove the entire `makeFloatingIcon` function (lines ~2327-2337).

- [ ] **Step 4: Remove floating icon creation from `endGhostDrag`**

In `endGhostDrag` (~line 2412-2423), replace the `if (ghostDragState.active)` branch that calls `makeFloatingIcon` with simply removing the ghost:

```javascript
function endGhostDrag() {
  if (!ghostDragState) return;
  ghostDragState.ghost.remove();
  ghostDragState = null;
  document.removeEventListener('mousemove', onGhostMove);
  document.removeEventListener('mouseup', endGhostDrag);
}
```

- [ ] **Step 5: Remove angry face/text bubble triggers from ghost path**

The functions `checkGhostReject`, `onGhostMove` hit branch, and the bin's angry face logic will be rewritten in Task 3 — but for now, confirm the old `rejectGhostToBin` call in `onGhostMove` is dead code once we remove it.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "refactor: remove reject-bin, roasts, and floating icon code"
```

---

### Task 2: Add CSS bin-charge animation

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add the `.bin-charging` state class and `@keyframes`**

Insert this CSS block after the existing bin styles (~line 1175, after `.desktop-bin.angry` rules):

```css
@keyframes bin-charge {
  0%, 100% { background: #7a4e3a; border-color: #5c3828; }
  10% { background: #ff0000; border-color: #ff6600; box-shadow: 0 0 12px #ff0000; }
  20% { background: #00ff00; border-color: #00aa00; box-shadow: 0 0 12px #00ff00; }
  30% { background: #ff00ff; border-color: #ffff00; box-shadow: 0 0 12px #ff00ff; }
  40% { background: #00ffff; border-color: #0088ff; box-shadow: 0 0 12px #00ffff; }
  50% { background: #ff0000; border-color: #ff6600; box-shadow: 0 0 12px #ff0000; }
  60% { background: #ffaa00; border-color: #ffff00; box-shadow: 0 0 12px #ffaa00; }
  70% { background: #00ff00; border-color: #00aa00; box-shadow: 0 0 12px #00ff00; }
  80% { background: #ff00ff; border-color: #ffff00; box-shadow: 0 0 12px #ff00ff; }
  90% { background: #00ffff; border-color: #0088ff; box-shadow: 0 0 12px #00ffff; }
}

.bin-charging .bin-body {
  animation: bin-charge 0.5s steps(1) infinite;
}

.bin-charging .bin-lid {
  animation: bin-charge 0.5s steps(1) infinite;
}

.bin-charging .bin-label {
  color: #ff6600;
  animation: hgBlink 0.15s step-end infinite;
}
```

This cycles through 10 high-saturation colors in 0.5s (steps(1) = sharp no-interpolation switches), repeating for 4 cycles over 2s. The label blinks rapidly.

- [ ] **Step 2: Verify no CSS conflicts**

The `.bin-charging` class goes on `#desktop-bin` (same level as `.angry`). The `.bin-charging .bin-body` etc. selectors only apply when the charging class is active. No overlap with `.angry` since they won't be active together.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add bin-charge CSS animation for 2s color cycling"
```

---

### Task 3: Add binCharge state machine

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add `binCharge` state object**

Insert this after the `let binRejecting = false;` declaration (~line 2234-2235 in current file) — but replace those lines. The new state replaces both `binDragState` (which was for windows) and `binRejecting`:

```javascript
let binCharge = {
  state: 'idle',       // 'idle' | 'charging' | 'ejecting'
  ghost: null,
  origRect: null,
  timer: null
};
```

- [ ] **Step 2: Add `binCharge.start()` method**

```javascript
function startBinCharge(ghost, origRect) {
  if (binCharge.state !== 'idle') {
    ghost.remove();
    return;
  }
  binCharge.state = 'charging';
  binCharge.ghost = ghost;
  binCharge.origRect = origRect;

  // Absorb ghost into bin
  const binRect = document.getElementById('desktop-bin').getBoundingClientRect();
  const binCx = binRect.left + binRect.width / 2;
  const binCy = binRect.top + binRect.height / 2;
  const gRect = ghost.getBoundingClientRect();
  ghost.style.transition = 'left 0.15s ease-in, top 0.15s ease-in, width 0.15s ease-in, height 0.15s ease-in, opacity 0.15s ease-in';
  ghost.style.left = (binCx - gRect.width / 2) + 'px';
  ghost.style.top = (binCy - gRect.height / 2) + 'px';
  ghost.style.width = '20px';
  ghost.style.height = '20px';
  ghost.style.opacity = '0.3';
  ghost.style.pointerEvents = 'none';

  // Start bin charging animation
  document.getElementById('desktop-bin').classList.add('bin-charging');

  // Set 2s timer
  binCharge.timer = setTimeout(() => ejectBinCharge(), 2000);
}
```

- [ ] **Step 3: Add `ejectBinCharge()` method**

```javascript
function ejectBinCharge() {
  if (binCharge.state !== 'charging') return;
  binCharge.state = 'ejecting';

  // Reset bin visual
  document.getElementById('desktop-bin').classList.remove('bin-charging');

  // Play throw-back sound
  playBoing();

  // Position ghost at bin center for return flight
  const ghost = binCharge.ghost;
  const binRect = document.getElementById('desktop-bin').getBoundingClientRect();
  const binCx = binRect.left + binRect.width / 2;
  const binCy = binRect.top + binRect.height / 2;
  const gRect = ghost.getBoundingClientRect();
  ghost.style.transition = 'none';
  ghost.style.left = (binCx - gRect.width / 2) + 'px';
  ghost.style.top = (binCy - gRect.height / 2) + 'px';
  ghost.style.width = '20px';
  ghost.style.height = '20px';
  ghost.style.opacity = '1';

  // Force layout
  void ghost.offsetWidth;

  // Animate to original position
  ghost.style.transition = 'left 0.3s cubic-bezier(0.25, 0.1, 0.25, 1), top 0.3s cubic-bezier(0.25, 0.1, 0.25, 1), width 0.2s ease, height 0.2s ease';
  ghost.style.left = binCharge.origRect.left + 'px';
  ghost.style.top = binCharge.origRect.top + 'px';
  ghost.style.width = binCharge.origRect.width + 'px';
  ghost.style.height = binCharge.origRect.height + 'px';

  // Cleanup after animation
  ghost.addEventListener('transitionend', function cleanup() {
    ghost.removeEventListener('transitionend', cleanup);
    ghost.remove();
    resetBinCharge();
  });

  // Fallback cleanup
  setTimeout(() => {
    if (binCharge.state === 'ejecting') {
      ghost.remove();
      resetBinCharge();
    }
  }, 500);
}
```

- [ ] **Step 4: Add `resetBinCharge()` method**

```javascript
function resetBinCharge() {
  document.getElementById('desktop-bin').classList.remove('bin-charging');
  if (binCharge.timer) {
    clearTimeout(binCharge.timer);
    binCharge.timer = null;
  }
  binCharge.state = 'idle';
  binCharge.ghost = null;
  binCharge.origRect = null;
}
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add binCharge state machine with charge/eject/reset"
```

---

### Task 4: Wire ghost drag pipeline to binCharge

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Capture original rect in `startGhostDrag`**

In `startGhostDrag` (~line 2381), add `config.origRect = r;` after the rect capture:

```javascript
function startGhostDrag(e, config) {
  if (binRejecting) return;
  const t = e.currentTarget;
  const r = t.getBoundingClientRect();
  const ghost = createDragGhost(r, config.html);
  config.origRect = r;  // <-- ADD THIS
  ghostDragState = { ghost, config, startX: e.clientX, startY: e.clientY, oX: r.left, oY: r.top, active: false };
  document.addEventListener('mousemove', onGhostMove);
  document.addEventListener('mouseup', endGhostDrag);
}
```

- [ ] **Step 2: Remove old `binRejecting` guard from `startGhostDrag`**

Replace `if (binRejecting) return;` with `if (binCharge.state !== 'idle') return;`:

```javascript
function startGhostDrag(e, config) {
  if (binCharge.state !== 'idle') return;
  // ... rest unchanged
}
```

- [ ] **Step 3: Rewrite `onGhostMove` hit branch**

In `onGhostMove` (~line 2402-2409), replace the reject call with bin charge:

```javascript
  if (checkGhostReject(ghostDragState.ghost)) {
    const state = ghostDragState;
    state.ghost.style.transition = 'none';
    document.removeEventListener('mousemove', onGhostMove);
    document.removeEventListener('mouseup', endGhostDrag);
    ghostDragState = null;
    startBinCharge(state.ghost, state.config.origRect);
  }
```

- [ ] **Step 4: Remove old `binRejecting` guard from `onGhostMove`**

In `onGhostMove`, change `if (!ghostDragState || binRejecting) return;` to `if (!ghostDragState) return;`:

```javascript
function onGhostMove(e) {
  if (!ghostDragState) return;
  // ...
}
```

- [ ] **Step 5: Remove old `binRejecting` variable declaration**

Delete or comment out `let binRejecting = false;` (the `binCharge` state object replaces it).

- [ ] **Step 6: Rename `checkGhostReject` to `checkBinHit`**

Rename the function and update its call site in `onGhostMove`:

```javascript
function checkBinHit(ghost) {
  // ... same body as checkGhostReject
}
```

Update call in `onGhostMove`:
```javascript
  if (checkBinHit(ghostDragState.ghost)) {
```

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: wire ghost drag pipeline to binCharge state machine"
```

---

### Task 5: Verify all behavior

**Files:**
- Modify: `index.html` (no changes — just manual testing)

- [ ] **Step 1: Open index.html in browser and test sidebar item drag**

1. Drag "About Me" sidebar item toward Recycle Bin
2. Ghost should follow cursor
3. When ghost enters bin zone → ghost shrinks and fades into bin
4. Bin starts color-cycling rapidly for 2s
5. After 2s → bin snaps normal → boing sound → ghost flies from bin back to sidebar slot
6. Ghost removed on arrival
7. Original sidebar item still functional (click opens window)

- [ ] **Step 2: Test taskbar tab drag**

1. Open an exercise window → it minimizes → taskbar tab appears
2. Drag the taskbar tab toward bin
3. Same flow: absorb → charge → throw back → tab still functional

- [ ] **Step 3: Test window drag to bin (unchanged behavior)**

1. Open a window
2. Drag window to bin → window should still get spit away (old reject behavior intact)
3. Verify angry face + text bubble still shows for window drops

- [ ] **Step 4: Test desktop drop (no-op)**

1. Drag sidebar item, drop on desktop (not near bin)
2. Ghost should simply disappear, no floating icon created

- [ ] **Step 5: Test double-drag during charging**

1. Drag sidebar item into bin → charging starts
2. While bin is charging, try dragging another item
3. Second drag should be ignored (guard at top of `startGhostDrag`)

- [ ] **Step 6: Commit final verification**

```bash
git add index.html
git commit -m "feat: implement volatile recycle bin throw-back mechanic"
```
