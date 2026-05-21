# Retro 90s OS Portfolio — Design Spec

## Overview

A single-page, non-scrollable (100vh x 100vw, overflow hidden) portfolio website styled as a retro pixelated 90s OS desktop, built in a single `index.html` file using HTML, CSS, and vanilla JS.

---

## 1. Color Palette

| Role | Color |
|------|-------|
| Primary (background / brand) | Deep burgundy/wine `#6b1d2f` |
| Earthy accent 1 | Muted sage green `#7a9e7e` |
| Earthy accent 2 | Warm beige `#d4c5a9` |
| Earthy accent 3 | Clay brown `#8b5e3c` |

---

## 2. Typography

- **Font**: Google Fonts "VT323" (monospace, pixelated aesthetic), fallback to "Courier New".
- Heading sizes: ~24px for sidebar items, ~16px for taskbar, ~14px for window title bars.

---

## 3. Layout Structure

```
┌──────────────────────────────────────────────────┐
│ ┌─────────┐  ┌───────────────────────────────┐   │
│ │ Sidebar │  │  Desktop (popup windows area)  │   │
│ │ 200px   │  │  flex: 1                       │   │
│ │         │  │                                │   │
│ │ About   │  │  (Canvas perspective grid       │   │
│ │ Me      │  │   renders behind everything)    │   │
│ │         │  │                                │   │
│ │ Exer-   │  │                                │   │
│ │ cises   │  │                                │   │
│ │         │  │                                │   │
│ │ Profes- │  │                                │   │
│ │ sors    │  │                                │   │
│ └─────────┘  └───────────────────────────────┘   │
│ ┌────────────────────────────────────────────────┐│
│ │  🏁 Welcome to my portfolio (marquee)      ││
│ │                              🇵🇰 14:30:22   ││
│ │                              🇮🇹 11:30:22   ││
│ └────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────┘
```

- Body: `100vw x 100vh`, `overflow: hidden`, flex column.
- Main row: sidebar (`flex: 0 0 200px`) + desktop area (`flex: 1`, `position: relative`).
- Taskbar: fixed bottom, ~40px height, dark background.

---

## 4. Background (Canvas)

- A full-viewport `<canvas>` element behind the desktop area.
- Draws a synthwave/vaporwave-style perspective grid:
  - Horizon line ~60% from top.
  - Grid lines radiate from a vanishing point toward the bottom edge.
  - Lines in muted sage green, warm beige, and clay brown over the burgundy background.
- Redrawn on window resize.

---

## 5. Left Sidebar

- Vertical menu, fixed width 200px, dark earthy background (clay brown).
- Three items rendered as clickable blocks:
  - **About Me**
  - **Exercises**
  - **Professors**
- Each has a pixel-art-style text block (styled with borders and box-shadow to look like pixel icons).
- Clicking any item opens the corresponding popup window.
- Active/selected state highlighting.

---

## 6. Bottom Taskbar

- Fixed to bottom, full width, ~40px height.
- Dark burgundy background, pixel border on top.
- **Left section**: `<marquee>` (or CSS `@keyframes` animation) displaying "Welcome to my portfolio" scrolling continuously right-to-left.
- **Right section**: Two digital clocks side-by-side:
  - `🇵🇰 HH:MM:SS` — Pakistan time (UTC+5)
  - `🇮🇹 HH:MM:SS` — Italy time (UTC+1 winter / UTC+2 summer)
  - Updated every second via `setInterval`.

---

## 7. Popup Window System

### 7.1 Window Creation
- Clicking a sidebar item creates a new window div in the desktop area.
- Default size: `width: 600px, height: 400px`.
- Positioned centered via `top: 50%; left: 50%; transform: translate(-50%, -50%)`.
- `position: absolute`, with a `z-index` counter that increments on focus.

### 7.2 Title Bar
- Height ~32px.
- Diagonal-striped texture via `repeating-linear-gradient(45deg, ...)`.
- Left side: minimize/iconize box (small square button).
- Center: window title text ("About Me", "Exercises", "Professors").
- Right side: maximize button (square icon), close button (X).
- Title bar buttons are styled as pixel squares.

### 7.3 Pixel Border
- 3px solid border in earthy tones.
- `box-shadow` for additional depth.

### 7.4 Window Controls
- **Close (X)**: Removes the window div from the DOM.
- **Maximize**: Toggles between standard centered size and fullscreen (`width: 100%, height: calc(100vh - 40px)`). Button icon changes (square → two overlapping squares).
- **Minimize**: Hides the window (set `display: none`). Window can be re-opened from sidebar.

### 7.5 Z-Index / Bring to Front
- Global `zIndexCounter` variable.
- On mousedown or click on any window, increment counter and set that window's `z-index`.
- The most recently clicked window is always on top.

---

## 8. Window Content

### 8.1 About Me
- A short introduction section about the user's university journey.
- Placeholder text, editable by the user.

### 8.2 Exercises
- A grid of pixelated folder icons, one per project:
  - Animation, Pattern, Gelato, Tic Tac Toe, Hand Gesture, Basmati.
- Each folder is a styled div with a folder-like appearance (CSS).
- Clicking a folder replaces the window content with that project's content:
  - An `<iframe>` loading the project's `index.html`, or inline HTML render.
  - A "back" button at the top returns to the folder grid view.

### 8.3 Professors
- A styled list or clean layout of professors with names and details.

---

## 9. Implementation

### 9.1 Target File
- `D:\Uni Home work\Obsidian\Obsidian\Website\index.html` (single file).

### 9.2 Dependencies
- Google Fonts: "VT323" (loaded from CDN).
- No other external libraries. All vanilla JS.

### 9.3 Browser Compatibility
- Modern browsers (Chrome, Firefox, Edge, Safari).
- No legacy IE support needed.

---

## 10. Out of Scope (YAGNI)

- No draggable windows (unless requested).
- No file system simulation (folder clicks load content directly).
- No right-click context menus.
- No sound effects.
- No localStorage persistence for window state.
