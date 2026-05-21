# Window Minimize Feature — Design Spec

## Overview

Add a classic OS desktop minimizing feature to the retro 90s OS portfolio. When clicking outside an open window or switching sections, the window minimizes (doesn't close) — it shrinks into a small taskbar tab. Clicking the tab restores the window to center with the rising chime sound.

All changes are within the single `index.html` file.

---

## 1. Window Manager State

A new `winState` object manages all window lifecycle:

```js
const winState = {
  openWindows: {},     // type -> { type, title, domElement, state, tabElement }
  activeWindow: null   // type of the currently open (non-minimized) window, or null
};
```

- `openWindows` maps each window type (`"about"`, `"exercises"`, `"professors"`) to its tracked state.
- `state` is `"open"` (visible, centered) or `"minimized"` (hidden, tab in taskbar).
- `createWindow()` stores the reference; re-clicking a sidebar item for the same type either bringsToFront or restores.
- Clicking a different sidebar item or the desktop background triggers minimize via `minimizeWindow()`.

---

## 2. Taskbar Tabs Area

A new `#taskbar-tabs` container is added to the taskbar, left side, before the marquee:

```
┌────────────────────────────────────────────────────────┐
│ [About Me ✕]  │  🏁 marquee...     🇵🇰 14:30  🇮🇹 11:30  │
└────────────────────────────────────────────────────────┘
```

```css
#taskbar-tabs {
  display: flex;
  align-items: center;
  gap: 4px;
  flex-shrink: 0;
  margin-right: 8px;
}

.taskbar-tab {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 2px 8px;
  background: #5c3828;
  border: 2px solid #7a4e3a;
  color: #d4c5a9;
  font-size: 12px;
  cursor: pointer;
  font-family: inherit;
  box-shadow: inset -1px -1px 0 #3a1e12;
  flex-shrink: 0;
  line-height: 1;
}

.taskbar-tab .tab-close {
  margin-left: 4px;
  font-size: 10px;
  cursor: pointer;
  color: #d4c5a9;
}
```

- Each minimized window gets one `.taskbar-tab` containing the section name and an ✕ button.
- The ✕ button closes the window with the glitch animation (applied to the tab itself).

---

## 3. Minimize Animation & Logic

### Animation (CSS)

```css
@keyframes minimizeToTaskbar {
  0% {
    transform: translate(-50%, -50%) scale(1);
    opacity: 1;
  }
  100% {
    transform: translate(-42vw, 65vh) scale(0.15);
    opacity: 0;
  }
}

.window.minimizing {
  animation: minimizeToTaskbar 0.3s ease-in forwards;
}
```

The window starts centered (`translate(-50%, -50%)`) and moves toward the bottom-left taskbar area while shrinking and fading.

> **Note:** The `translate(-42vw, 65vh)` values are approximations that will be tuned during implementation to visually land near the taskbar tabs area. The exact values depend on viewport size and the taskbar's position.

### `minimizeWindow(type)` function

1. Look up entry in `winState.openWindows[type]`.
2. If state is already `"minimized"`, do nothing.
3. Remove any existing focus/active state.
4. Apply `.minimizing` CSS class to the window DOM element.
5. After 300ms (on animationend or setTimeout):
   - Set `win.style.display = 'none'`.
   - Remove `.minimizing` class.
   - Create a `.taskbar-tab` element and append to `#taskbar-tabs`.
   - Wire the tab's click to restore, and its ✕ to close-with-glitch.
   - Update state to `"minimized"`, set `tabElement`.
   - Set `winState.activeWindow = null` if this was the active window.

---

## 4. Restore Animation & Logic

### Animation (CSS)

```css
@keyframes restoreFromTaskbar {
  0% {
    transform: translate(-42vw, 65vh) scale(0.15);
    opacity: 0;
  }
  100% {
    transform: translate(-50%, -50%) scale(1);
    opacity: 1;
  }
}

.window.restoring {
  animation: restoreFromTaskbar 0.3s ease-out forwards;
}
```

### `restoreWindow(type)` function

1. If `winState.activeWindow` is set to a different type → call `minimizeWindow(activeWindow)` first.
2. Look up entry; if state isn't `"minimized"`, do nothing.
3. Remove the taskbar tab element from DOM.
4. Set `win.style.display = ''` and apply `.restoring` class.
5. After 300ms:
   - Remove `.restoring` class.
   - Update state to `"open"`.
   - Set `winState.activeWindow = type`.
   - Call `bringToFront(win)`.
   - `playChime()`.

---

## 5. Click Handlers

### Desktop Background Click

```js
desktop.addEventListener('click', (e) => {
  if (e.target === desktop || e.target.id === 'bg-title' || e.target.tagName === 'CANVAS') {
    if (winState.activeWindow) {
      minimizeWindow(winState.activeWindow);
    }
  }
});
```

No hourglass, no new window — just minimize if anything is open.

### Sidebar Click (updated)

```js
document.querySelectorAll('.sidebar-item').forEach(item => {
  item.addEventListener('click', () => {
    const type = item.dataset.window;
    const titles = { about: 'About Me', exercises: 'Exercises', professors: 'Professors' };

    // Case 1: Already open in center → just bring to front
    if (winState.openWindows[type]?.state === 'open') {
      bringToFront(winState.openWindows[type].domElement);
      return;
    }

    // Case 2: Minimized → restore it
    if (winState.openWindows[type]?.state === 'minimized') {
      restoreWindow(type);
      return;
    }

    // Case 3: New window → minimize current if any, then hourglass, then open
    if (winState.activeWindow) {
      minimizeWindow(winState.activeWindow);
    }

    showHourglass(() => {
      createWindow(type, titles[type]);
      playChime();
    });
  });
});
```

- The 0.5s hourglass processing currently runs for every sidebar click; it now only runs when creating a **new** window (not when restoring a minimized one or bringing to front).
- Fix: the `playBlip` on mouseenter is preserved as-is.

---

## 6. Close (X) Behavior on Minimized Tabs

When the ✕ on a taskbar tab is clicked:

1. Prevent the tab's own click-to-restore from firing.
2. Temporarily show the hidden window element (`display: ''`) at the last known position (or at center with proper transform).
3. Apply the `.glitch` animation to the **tab element** instead (shake, distort, flicker on the tab itself).
4. Call `playZap()`.
5. After 500ms:
   - Remove the window element from DOM.
   - Remove the tab element from DOM.
   - Delete the entry from `winState.openWindows`.
   - If this was `activeWindow`, set to `null`.

Since the tab is small, the existing glitch animation keyframes apply but the visual is constrained to the tab's bounding box — the effect appears as a pixel-shake with flicker on the button-sized element.

**Close from center (existing behavior)** — unchanged. The X on an open window's title bar triggers the `.glitch` animation on the window div + `playZap()`, then removes after 500ms.

---

## 7. `createWindow()` Changes

The existing `createWindow()` is modified to:
- Store the new window in `winState.openWindows[type]`.
- Set initial state to `"open"`.
- Set `winState.activeWindow = type`.
- The existing DOM-based duplicate check (`querySelector(.window[data-type="${type}"])`) is removed. The sidebar handler now guarantees `createWindow` is only called for truly new windows via `winState` checks.
- A safety guard at the top of the function remains: if `winState.openWindows[type]` already exists, just bring it to front and return (belt-and-suspenders).

---

## 8. Files & Dependencies

- **File**: `D:\Uni Home work\Obsidian\Obsidian\Website - Copy (2)\index.html` (single file, no new files).
- **Dependencies**: None beyond existing (Google Fonts VT323, no external libs).

---

## 9. Out of Scope

- No changes to the maximized window state.
- No changes to the exercises iframe flow.
- No persistence of window state across page reloads.
- No dragging windows.
