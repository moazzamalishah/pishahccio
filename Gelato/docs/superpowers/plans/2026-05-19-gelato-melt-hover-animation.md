# Gelato Melt Hover Animation — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** A single-file HTML page where "GELATO" letters melt one-by-one on hover, then reverse when the mouse leaves.

**Architecture:** Single `index.html`. Each letter in its own `<span>`. CSS transitions for smooth animation. JavaScript staggers the melt/reverse with `setTimeout`.

**Tech Stack:** HTML5, CSS3 transitions, vanilla JS

---

### Task 1: Scaffold HTML with Letter Spans

**Files:**
- Create: `gelato-melt/index.html`

- [ ] **Step 1: Write the HTML scaffold with individual letter spans**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>GELATO Melt</title>
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
  #word {
    display: flex;
    gap: 0.05em;
    cursor: pointer;
  }
  .letter {
    font-family: Impact, 'Arial Black', sans-serif;
    font-size: min(20vw, 200px);
    color: #000;
    text-transform: uppercase;
    user-select: none;
    display: inline-block;
    transition: all 0.3s ease;
  }
  .letter.melt {
    transform: translateY(40px) scaleY(0.4) skewX(5deg);
    filter: blur(1px);
  }
</style>
</head>
<body>
<div id="word">
  <span class="letter">G</span>
  <span class="letter">E</span>
  <span class="letter">L</span>
  <span class="letter">A</span>
  <span class="letter">T</span>
  <span class="letter">O</span>
</div>
</body>
</html>
```

- [ ] **Step 2: Commit**

```bash
git add gelato-melt/index.html
git commit -m "chore: scaffold gelato-melt with letter spans"
```

---

### Task 2: Add JavaScript Hover Logic

**Files:**
- Modify: `gelato-melt/index.html`

- [ ] **Step 1: Add JS for staggered melt and reverse**

Insert before `</body>`:

```html
<script>
const letters = document.querySelectorAll('.letter');
const word = document.getElementById('word');
const STAGGER = 80;
const LETTERS_COUNT = letters.length;

function melt() {
  letters.forEach((l, i) => {
    setTimeout(() => l.classList.add('melt'), i * STAGGER);
  });
}

function unmelt() {
  for (let i = 0; i < LETTERS_COUNT; i++) {
    setTimeout(() => letters[LETTERS_COUNT - 1 - i].classList.remove('melt'), i * STAGGER);
  }
}

word.addEventListener('mouseenter', melt);
word.addEventListener('mouseleave', unmelt);
</script>
```

`melt()` adds the `.melt` class to letters left-to-right with 80ms stagger. `unmelt()` removes it right-to-left (reverse order) with the same stagger. CSS `transition: all 0.3s ease` handles the smooth animation.

- [ ] **Step 2: Commit**

```bash
git add gelato-melt/index.html
git commit -m "feat: add staggered melt on hover, reverse on leave"
```

---

### Task 3: Polish — Variety in Tilt and Timing

**Files:**
- Modify: `gelato-melt/index.html`

- [ ] **Step 1: Add alternating skew directions for organic look**

Replace `.letter.melt` rule:
```css
  .letter.melt {
    transform: translateY(40px) scaleY(0.4);
    filter: blur(1px);
  }
```

Add alternating skew via JS — modify the melt function:
```js
function melt() {
  letters.forEach((l, i) => {
    const tilt = i % 2 === 0 ? 'skewX(5deg)' : 'skewX(-5deg)';
    setTimeout(() => {
      l.style.transform = `translateY(40px) scaleY(0.4) ${tilt}`;
      l.classList.add('melt');
    }, i * STAGGER);
  });
}

function unmelt() {
  for (let i = 0; i < LETTERS_COUNT; i++) {
    setTimeout(() => {
      letters[LETTERS_COUNT - 1 - i].style.transform = '';
      letters[LETTERS_COUNT - 1 - i].classList.remove('melt');
    }, i * STAGGER);
  }
}
```

This gives alternating tilt: G skews right, E left, L right, A left, T right, O left.

- [ ] **Step 2: Commit**

```bash
git add gelato-melt/index.html
git commit -m "polish: add alternating tilt for organic melt feel"
```
