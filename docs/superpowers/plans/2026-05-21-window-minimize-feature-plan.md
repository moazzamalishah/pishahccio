# Window Minimize Feature — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add classic OS desktop minimizing to the retro portfolio — clicking outside a window or switching sections shrinks it into a taskbar tab; clicking the tab restores it.

**Architecture:** Single-file `index.html`. A `winState` object tracks window lifecycle (open/minimized). Minimize uses CSS keyframe animation (scale + translate toward taskbar), then hides the window and creates a taskbar tab. Restore reverses it. Sidebar handler gains 3-case logic (focus/restore/create). Desktop click minimizes active window only.

**Tech Stack:** HTML5, CSS3, vanilla JS (no dependencies beyond existing Google Fonts VT323).

**Files:**
- Modify: `D:\Uni Home work\Obsidian\Obsidian\Website - Copy (2)\index.html`

---

### Task 1: Add Taskbar Tabs Container to HTML

**Files:**
- Modify: `D:\Uni Home work\Obsidian\Obsidian\Website - Copy (2)\index.html`

- [ ] **Step 1: Add `#taskbar-tabs` div inside `#taskbar`, before `#marquee-wrap`**

In the `#taskbar` div, insert a new container before the marquee:

Old HTML (around line 634):
```html
<div id="taskbar">
  <div id="marquee-wrap">
```

New HTML:
```html
<div id="taskbar">
  <div id="taskbar-tabs"></div>
  <div id="marquee-wrap">
```

- [ ] **Step 2: Verify insertion**

Read lines 634-636 of the modified file to confirm `#taskbar-tabs` is between `#taskbar` and `#marquee-wrap`.

---

### Task 2: Add CSS for Taskbar Tabs and Minimize/Restore Animations

**Files:**
- Modify: `D:\Uni Home work\Obsidian\Obsidian\Website - Copy (2)\index.html`

- [ ] **Step 1: Add tab and animation styles to the `<style>` block**

Find the `#crt-toggle.active` block (around line 278-281) and add the new CSS after it:

```css
/* Taskbar tabs for minimized windows */
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

.taskbar-tab:hover {
  background: #7a4e3a;
}

.taskbar-tab:active {
  box-shadow: inset 1px 1px 0 #3a1e12;
}

.taskbar-tab .tab-close {
  margin-left: 4px;
  font-size: 10px;
  cursor: pointer;
  color: #d4c5a9;
}

.taskbar-tab .tab-close:hover {
  color: #ff6b6b;
}

/* Minimize animation */
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

/* Restore animation */
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

- [ ] **Step 2: Verify CSS is valid**

Open the file and confirm the new CSS blocks are inside the `<style>` tag and properly closed before `</style>`.

---

### Task 3: Add Window Manager State and Modify `createWindow()`

**Files:**
- Modify: `D:\Uni Home work\Obsidian\Obsidian\Website - Copy (2)\index.html`

- [ ] **Step 1: Add `winState` object after `let zIndexCounter = 100;`**

Find `let zIndexCounter = 100;` (around line 660) and add the state object right after it:

```js
const winState = {
  openWindows: {},
  activeWindow: null
};
```

- [ ] **Step 2: Modify `createWindow()` to track state**

Replace the existing `createWindow` function (lines 836-890). The new version:
1. Removes the old DOM-based duplicate check (`querySelector(.window[data-type="${type}"])`)
2. Adds a `winState` safety guard
3. Registers the new window in `winState`
4. Stores the close handler reference so it can be called from tab close too

Old `createWindow`:
```js
function createWindow(type, title) {
  const existing = document.querySelector(`.window[data-type="${type}"]`);
  if (existing) {
    bringToFront(existing);
    return;
  }

  const win = document.createElement('div');
  win.className = 'window';
  win.dataset.type = type;
  win.style.zIndex = ++zIndexCounter;
  win.style.top = '50%';
  win.style.left = '50%';
  win.style.transform = 'translate(-50%, -50%)';

  win.innerHTML = `
    <div class="window-titlebar">
      <span class="win-title">${title}</span>
      <span class="win-btn win-maximize" title="Maximize">&#x25a1;</span>
      <span class="win-btn win-close" title="Close">&#x2715;</span>
    </div>
    <div class="window-body"></div>
  `;

  desktop.appendChild(win);

  const body = win.querySelector('.window-body');
  if (type === 'exercises') {
    buildExerciseGrid(body);
  } else {
    body.innerHTML = WINDOW_CONTENT[type] || '';
  }

  win.querySelector('.win-close').addEventListener('click', () => {
    win.classList.add('glitch');
    playZap();
    setTimeout(() => win.remove(), 500);
  });

  win.querySelector('.win-maximize').addEventListener('click', () => {
    win.classList.toggle('maximized');
    if (win.classList.contains('maximized')) {
      win.style.top = '0';
      win.style.left = '0';
      win.style.transform = 'none';
    } else {
      win.style.top = '50%';
      win.style.left = '50%';
      win.style.transform = 'translate(-50%, -50%)';
    }
  });

  win.addEventListener('mousedown', () => bringToFront(win));
  bringToFront(win);
}
```

New `createWindow`:
```js
function createWindow(type, title) {
  if (winState.openWindows[type]) {
    if (winState.openWindows[type].state === 'minimized') {
      restoreWindow(type);
    } else {
      bringToFront(winState.openWindows[type].domElement);
    }
    return;
  }

  const win = document.createElement('div');
  win.className = 'window';
  win.dataset.type = type;
  win.style.zIndex = ++zIndexCounter;
  win.style.top = '50%';
  win.style.left = '50%';
  win.style.transform = 'translate(-50%, -50%)';

  win.innerHTML = `
    <div class="window-titlebar">
      <span class="win-title">${title}</span>
      <span class="win-btn win-maximize" title="Maximize">&#x25a1;</span>
      <span class="win-btn win-close" title="Close">&#x2715;</span>
    </div>
    <div class="window-body"></div>
  `;

  desktop.appendChild(win);

  const body = win.querySelector('.window-body');
  if (type === 'exercises') {
    buildExerciseGrid(body);
  } else {
    body.innerHTML = WINDOW_CONTENT[type] || '';
  }

  win.querySelector('.win-close').addEventListener('click', () => {
    win.classList.add('glitch');
    playZap();
    setTimeout(() => {
      win.remove();
      delete winState.openWindows[type];
      if (winState.activeWindow === type) winState.activeWindow = null;
    }, 500);
  });

  win.querySelector('.win-maximize').addEventListener('click', () => {
    win.classList.toggle('maximized');
    if (win.classList.contains('maximized')) {
      win.style.top = '0';
      win.style.left = '0';
      win.style.transform = 'none';
    } else {
      win.style.top = '50%';
      win.style.left = '50%';
      win.style.transform = 'translate(-50%, -50%)';
    }
  });

  win.addEventListener('mousedown', () => bringToFront(win));

  winState.openWindows[type] = { type, title, domElement: win, state: 'open', tabElement: null };
  winState.activeWindow = type;
  bringToFront(win);
}
```

- [ ] **Step 3: Verify `createWindow` loads content**

Open the file in a browser and confirm clicking a sidebar item still creates a centered window with correct content. Console should have no errors.

---

### Task 4: Add `minimizeWindow()` and `restoreWindow()` Functions

**Files:**
- Modify: `D:\Uni Home work\Obsidian\Obsidian\Website - Copy (2)\index.html`

- [ ] **Step 1: Add `minimizeWindow()` after `bringToFront()`**

Find `function bringToFront(win)` (around line 892) and add these two functions after it:

```js
function minimizeWindow(type) {
  const entry = winState.openWindows[type];
  if (!entry || entry.state !== 'open') return;

  const win = entry.domElement;

  win.classList.add('minimizing');
  if (winState.activeWindow === type) winState.activeWindow = null;

  setTimeout(() => {
    win.style.display = 'none';
    win.classList.remove('minimizing');
    entry.state = 'minimized';

    const tab = document.createElement('div');
    tab.className = 'taskbar-tab';
    tab.innerHTML = `<span>${entry.title}</span><span class="tab-close">&#x2715;</span>`;

    tab.addEventListener('click', (e) => {
      if (e.target.classList.contains('tab-close')) return;
      restoreWindow(type);
    });

    tab.querySelector('.tab-close').addEventListener('click', (e) => {
      e.stopPropagation();
      tab.classList.add('glitch');
      playZap();
      setTimeout(() => {
        win.remove();
        tab.remove();
        delete winState.openWindows[type];
        if (winState.activeWindow === type) winState.activeWindow = null;
      }, 500);
    });

    document.getElementById('taskbar-tabs').appendChild(tab);
    entry.tabElement = tab;
  }, 300);
}
```

- [ ] **Step 2: Add `restoreWindow()` after `minimizeWindow()`**

```js
function restoreWindow(type) {
  const entry = winState.openWindows[type];
  if (!entry || entry.state !== 'minimized') return;

  if (winState.activeWindow && winState.activeWindow !== type) {
    minimizeWindow(winState.activeWindow);
  }

  if (entry.tabElement) {
    entry.tabElement.remove();
    entry.tabElement = null;
  }

  const win = entry.domElement;
  win.style.display = '';
  bringToFront(win);
  win.classList.add('restoring');

  setTimeout(() => {
    win.classList.remove('restoring');
    entry.state = 'open';
    winState.activeWindow = type;
    playChime();
  }, 300);
}
```

- [ ] **Step 3: Quick check**

Read the modified file to confirm both functions are syntactically valid (braces match, no undefined references).

---

### Task 5: Update Sidebar Click Handler and Add Desktop Click Handler

**Files:**
- Modify: `D:\Uni Home work\Obsidian\Obsidian\Website - Copy (2)\index.html`

- [ ] **Step 1: Update the sidebar click handler**

Replace the existing sidebar click handler (around lines 930-940) with the 3-case version:

Old:
```js
document.querySelectorAll('.sidebar-item').forEach(item => {
  item.addEventListener('mouseenter', playBlip);
  item.addEventListener('click', () => {
    const type = item.dataset.window;
    const titles = { about: 'About Me', exercises: 'Exercises', professors: 'Professors' };
    showHourglass(() => {
      createWindow(type, titles[type]);
      playChime();
    });
  });
});
```

New:
```js
document.querySelectorAll('.sidebar-item').forEach(item => {
  item.addEventListener('mouseenter', playBlip);
  item.addEventListener('click', () => {
    const type = item.dataset.window;
    const titles = { about: 'About Me', exercises: 'Exercises', professors: 'Professors' };

    if (winState.openWindows[type]?.state === 'open') {
      bringToFront(winState.openWindows[type].domElement);
      return;
    }

    if (winState.openWindows[type]?.state === 'minimized') {
      restoreWindow(type);
      return;
    }

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

- [ ] **Step 2: Add desktop background click handler**

Add this after the sidebar handler block (before `// CRT toggle`):

```js
// Desktop background click — minimize active window
desktop.addEventListener('click', (e) => {
  if (e.target === desktop || e.target.id === 'bg-title' || e.target.tagName === 'CANVAS') {
    if (winState.activeWindow) {
      minimizeWindow(winState.activeWindow);
    }
  }
});
```

- [ ] **Step 3: Verify in browser**

Open the file and test:
1. Click "About Me" → window opens → click desktop background → window minimizes to taskbar tab
2. Click "Exercises" → About Me minimizes → hourglass plays → Exercises opens
3. Click taskbar "[About Me]" → Exercises auto-minimizes → About Me restores with chime
4. Click taskbar "[About Me] ✕" → tab glitches → window removed
5. Click X on open window → glitch animation plays (existing behavior preserved)

---

### Task 6: Tune Minimize Animation Values (Visual Polish)

**Files:**
- Modify: `D:\Uni Home work\Obsidian\Obsidian\Website - Copy (2)\index.html`

- [ ] **Step 1: Test the minimize animation visually**

Open the file in a browser, open a window, click the desktop background. Observe whether the window animates toward the bottom-left taskbar area before disappearing.

- [ ] **Step 2: Adjust `translate` values if needed**

If the animation trajectory doesn't look right, tweak the `translate(-42vw, 65vh)` values in both `@keyframes minimizeToTaskbar` and `@keyframes restoreFromTaskbar`:
- Increase `-42vw` (more negative = more left) or decrease (less negative = more centered)
- Increase `65vh` (higher = further down) or decrease (lower = less down)
- Target: the window should visually shrink and slide toward the taskbar's left side where tabs appear

Make small adjustments (`±5vw`, `±5vh`), re-test until the motion looks natural.

- [ ] **Step 3: Verify restore animation**

Click a taskbar tab and confirm the window animates back to center from the same trajectory.

---

### Task 7: Final Verification

**Files:**
- Modify: `D:\Uni Home work\Obsidian\Obsidian\Website - Copy (2)\index.html`

- [ ] **Step 1: Check all requirements**

Run through this checklist in the browser:
- [ ] Click sidebar → window opens centered
- [ ] Click desktop background → window minimizes to taskbar tab (no hourglass)
- [ ] Click different sidebar item → previous window minimizes → hourglass plays → new window opens
- [ ] Click same minimized tab → previous window auto-minimizes → restored window plays chime
- [ ] Click X on open window → 0.5s glitch + zap sound → window removed
- [ ] Click ✕ on minimized tab → glitch on tab → zap sound → window + tab removed
- [ ] Maximize button still works on open windows
- [ ] Exercises folder grid and iframe loading still work
- [ ] No console errors
- [ ] Clocks and marquee still function
