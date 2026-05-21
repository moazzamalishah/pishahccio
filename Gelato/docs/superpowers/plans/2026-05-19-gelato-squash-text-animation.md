# Gelato Squash Text Animation — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** A standalone looping CSS + Web Audio animation of "GELATO" dropping, squashing flat with a THUD, then recovering.

**Architecture:** Single `index.html`. Text as an `<h1>` with CSS keyframe animation for squash/stretch. Web Audio API generates the thud sound.

**Tech Stack:** HTML5, CSS3 `@keyframes`, Web Audio API

---

### Task 1: Scaffold HTML + Text Styling

**Files:**
- Create: `gelato-squash/index.html`

- [ ] **Step 1: Write the HTML scaffold with full-screen centered text**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>GELATO Squash</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #fff;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
  }
  h1 {
    font-family: Impact, 'Arial Black', sans-serif;
    font-size: min(20vw, 200px);
    color: #000;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    user-select: none;
  }
</style>
</head>
<body>
<h1>GELATO</h1>
</body>
</html>
```

- [ ] **Step 2: Commit**

```bash
git add gelato-squash/index.html
git commit -m "chore: scaffold gelato-squash text page"
```

---

### Task 2: Implement CSS Squash Animation

**Files:**
- Modify: `gelato-squash/index.html`

- [ ] **Step 1: Replace the h1 rule and add keyframes**

Replace the `<style>` block content:

```css
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #fff;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
  }
  h1 {
    font-family: Impact, 'Arial Black', sans-serif;
    font-size: min(20vw, 200px);
    color: #000;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    user-select: none;
    animation: squash 4.5s ease-in-out infinite;
  }
  @keyframes squash {
    0%   { transform: translateY(-80vh) scale(1); }
    22%  { transform: translateY(0) scale(1); }
    23%  { transform: translateY(0) scaleX(1.4) scaleY(0.15); }
    55%  { transform: translateY(0) scaleX(1.4) scaleY(0.15); }
    65%  { transform: translateY(0) scale(1); }
    70%  { transform: translateY(0) scale(1.05, 0.95); }
    75%  { transform: translateY(0) scale(1); }
    100% { transform: translateY(0) scale(1); }
  }
```

Timing per spec (4.5s loop):
- 0-22% (0-1.0s): text drops from above
- 22-23% (1.0s): instant squash
- 23-55% (1.0-2.5s): holds squashed for 1.5s
- 55-65% (2.5-3.5s): recovers to normal
- 65-70% (3.5s): overshoot wobble
- 70-100% (3.5-4.5s): rest before loop

- [ ] **Step 2: Commit**

```bash
git add gelato-squash/index.html
git commit -m "feat: add squash keyframe animation with spec timing"
```

---

### Task 3: Implement Web Audio Thud Sound

**Files:**
- Modify: `gelato-squash/index.html`

- [ ] **Step 1: Add Web Audio thud function and trigger**

Insert before `</body>`:

```html
<script>
function playThud() {
  const actx = new (window.AudioContext || window.webkitAudioContext)();
  const osc = actx.createOscillator();
  const gain = actx.createGain();
  osc.type = 'sine';
  osc.frequency.setValueAtTime(70, actx.currentTime);
  osc.frequency.exponentialRampToValueAtTime(30, actx.currentTime + 0.15);
  gain.gain.setValueAtTime(0.8, actx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001, actx.currentTime + 0.2);
  osc.connect(gain);
  gain.connect(actx.destination);
  osc.start(actx.currentTime);
  osc.stop(actx.currentTime + 0.2);
}

const h1 = document.querySelector('h1');
function scheduleThud() {
  setTimeout(playThud, 1000);
}
h1.addEventListener('animationiteration', scheduleThud);
scheduleThud();
</script>
```

The `animationiteration` event fires when the 4.5s loop restarts. Thud plays 1s later, matching the squash at the 22% mark.

- [ ] **Step 2: Commit**

```bash
git add gelato-squash/index.html
git commit -m "feat: add Web Audio thud at squash impact"
```

---

### Task 4: Polish Timing and Feel

**Files:**
- Modify: `gelato-squash/index.html`

- [ ] **Step 1: Replace linear easing with custom cubic-bezier for elastic feel**

Replace the h1 animation value:
```css
    animation: squash 4.5s cubic-bezier(0.25, 0.4, 0.4, 1) infinite;
```

- [ ] **Step 2: Commit**

```bash
git add gelato-squash/index.html
git commit -m "polish: adjust easing for elastic feel"
```
