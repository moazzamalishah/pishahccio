# Basmati Rice Recipe Website — Design Spec

**Date:** 2026-05-18
**Status:** Approved

---

## Overview

A single-page HTML website displaying the Basmati Rice with Scrambled Eggs and Tuna recipe. Static, zero-dependency, designed to be opened directly in a browser.

## File Structure

```
Code Base/
  index.html      # Single-file website (HTML + CSS inline)
```

Everything lives in one file. No external stylesheets, scripts, or dependencies.

## Layout

One-column, centered layout, responsive for both desktop and mobile.

1. **Header** — Recipe title ("Basmati Rice with Scrambled Eggs and Tuna") + Chef name ("Moazzam")
2. **Ingredients** — Unordered list of ingredients with quantities
3. **Preparation** — Numbered steps grouped into 6 sections: Soak, Rinse, Cook, Eggs, Tuna, Combine
4. **Footer** — Minimal, optional attribution

## Content

Sourced from `Untitled.md` in the vault root. The recipe content is:

- 1 cup Basmati rice
- 1.5 cups water
- Cold water (soaking and rinsing)
- 2 eggs
- 1 can tuna
- 1 tbsp cooking oil
- Salt (optional)

Preparation steps follow the established 6-section structure from the markdown file.

## Styling

- **Palette:** Warm, earthy tones (terracotta/orange accents, warm neutrals)
- **Background:** White or off-white (warm cream)
- **Typography:** Clean readable font (system font stack or serif)
- **Mobile:** Responsive — reflows naturally on small screens
- **Print:** Readable when printed

## Constraints

- No external dependencies
- No build step
- Works by opening index.html in a browser
- Placed in Code Base/ directory

## Success Criteria

- File opens in any modern browser with no setup
- Recipe is fully readable, with ingredients and steps clearly separated
- Styling is visually warm and pleasant
