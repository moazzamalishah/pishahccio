# Recipe Collection Website — Design Spec

**Date:** 2026-05-18
**Project:** Basmati (recipe collection single-file website)

---

## Overview

A single-file HTML website that displays a collection of recipes in an **organic, imperfect, handcrafted design style**. The site uses earthy colors (olive green #5B6C3E, warm cream backgrounds, burgundy #6B2D3C accents), handwritten fonts for headings, and deliberate visual "imperfections" (sketchy borders, slight rotations, rough underlines) to evoke a personal handwritten cookbook.

---

## Color Palette

| Role | Color | Hex |
|------|-------|-----|
| Page background | Warm cream | `#F5F0E8` |
| Card background | Lighter cream | `#FAF7F2` |
| Primary text | Warm near-black | `#2C2C24` |
| Olive green (primary) | Olive | `#5B6C3E` |
| Olive light (secondary) | Light olive | `#7C8F5A` |
| Burgundy (accent) | Deep burgundy | `#6B2D3C` |
| Muted text / borders | Warm gray | `#8A7F72` |

Usage: green appears on headings, buttons, and key UI. Burgundy is limited to small accents — checkmarks, dots, tag highlights. Never more than ~5% of the visual area.

---

## Typography

- **Headings:** Caveat (Google Font) — a casual handwritten script with uneven baselines
- **Body:** Literata (Google Font) — warm serif with slight texture, reads like a printed book
- **Fallback:** `serif` for both

Usage notes:
- Site title uses Caveat at `~2.5rem`
- Recipe card titles use Caveat at `~1.5rem`
- Body text is Literata at `~1rem`
- `::first-letter` drop caps on recipe descriptions

---

## Layout

### Homepage (grid)
- Centered header with handwritten site title "Basmati" and tagline
- Tag filter bar (pill buttons, olive outline, burgundy active state)
- Responsive CSS grid: `repeat(auto-fill, minmax(280px, 1fr))`
- 3 columns on wide screens, 2 on tablet, 1 on mobile
- Cards have slight random rotations (`transform: rotate(-0.3..0.5deg)`) via CSS classes

### Recipe detail view
- Triggered by clicking a card
- Grid fades out, recipe content slides/transitions in
- "← Back" button (handwritten style) returns to grid
- Layout: header image area → title + tags → ingredients list → numbered steps
- Same responsive max-width (~720px) as existing page

### Recipe cards
- Irregular hand-drawn border (via layered CSS outline/border combos or SVG stroke)
- Card content: decorative illustration/emoji placeholder, title, short description, tag pills
- Hover: slight lift with irregularly offset shadow

---

## Visual "Imperfection" Details

1. **Hand-drawn borders:** Cards use a sketchy border effect (layered dashed/dotted borders, or a repeating SVG stroke path)
2. **Rough underlines:** Heading underlines use a jagged `background-image` gradient rather than a solid line
3. **Slight rotations:** Cards have tiny random-looking rotations, some slightly misaligned within the grid
4. **Noise texture:** A subtle paper-grain noise overlay on the page background
5. **Irregular shadows:** Box shadows are offset unevenly, mimicking a drawn shadow rather than a CSS-perfect one
6. **Typography quirks:** Slightly varied letter-spacing on site title, drop caps, occasional handwritten touches on numbers/labels

---

## Recipes (v1)

Three recipes, stored as a JS array of objects:

1. **Basmati Rice with Scrambled Eggs and Tuna** (existing content)
   - Tags: Rice, Eggs, Tuna, Quick
2. **Pistachio Croissant**
   - Tags: Pastry, Pistachio, Breakfast
3. **Pistachio Gelato**
   - Tags: Dessert, Pistachio, Frozen

Each recipe object: `{ id, title, chef, description, tags, image, ingredients[], steps[] }`

---

## Technical Architecture

**Single file:** `index.html` with inline `<style>` and inline `<script>`.

### CSS
- Google Fonts `@import` for Caveat + Literata
- CSS custom properties for the color palette
- Grid layout for homepage cards
- Transitions for view switching
- Responsive breakpoints at 768px and 480px

### JavaScript (~50-70 lines)
- Array of recipe data objects
- `showRecipe(index)` — hides grid, renders and shows recipe detail
- `showHome()` — returns to grid
- `filterByTag(tag)` — shows/hides cards by tag
- URL hash tracking (`#recipe-<id>`) for bookmarkable links
- No build tools, frameworks, or external dependencies (other than Google Fonts)

### Content structure (HTML)
```
<header>
  <h1>Basmati</h1>
  <p class="tagline">...handwritten cookbook...</p>
</header>

<main id="home">
  <nav class="filter-bar">...</nav>
  <div class="recipe-grid">...</div>
</main>

<main id="recipe-view" style="display:none">
  <button class="back-btn">← Back</button>
  <article>...</article>
</main>

<footer>...</footer>
```

---

## Responsive Behavior

- Mobile (≤480px): 1 column, reduced padding, smaller fonts
- Tablet (481–768px): 2 columns
- Desktop (>768px): 3 columns, full padding
- Recipe detail: max-width 720px centered on all sizes

---

## Acceptance Criteria

1. Single `index.html` file, opens directly in browser — no server needed
2. Homepage shows a grid of recipe cards with filter-by-tag
3. Clicking a card opens the recipe detail view
4. Back button returns to the filtered grid state
5. Olive green + cream + burgundy color scheme throughout
6. Handwritten font (Caveat) for headings
7. Visible "imperfect" design details (sketchy borders, rotation, rough underlines)
8. Responsive at mobile, tablet, desktop widths
9. All three recipes render correctly with ingredients and steps
10. URL hash updates for bookmarkable recipe links
