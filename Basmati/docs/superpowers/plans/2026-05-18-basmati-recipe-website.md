# Recipe Collection Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file HTML recipe collection site with organic/imperfect design, olive green + burgundy palette, and handwritten typography.

**Architecture:** Single `Code Base/index.html` with inline CSS and JS. Recipe data stored as a JS array of objects. View switching between a card grid (homepage) and recipe detail view via vanilla JS. URL hash for bookmarkable links.

**Tech Stack:** HTML5, CSS3 (Grid, Custom Properties, Transitions, Transforms), Vanilla JS, Google Fonts (Caveat + Literata)

---

### Task 1: HTML skeleton + CSS foundations

**Files:**
- Modify: `Code Base/index.html`

- [ ] **Step 1: Replace index.html with skeleton, CSS reset, custom properties, and Google Fonts**

Write the complete new HTML structure. This step establishes the foundation — the visual design, font imports, CSS custom properties for the palette, and the two-section layout (`#home` grid and `#recipe-view` detail).

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Basmati — Handwritten Recipes</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Caveat:wght@400;600;700&family=Literata:opsz,wght@7..72,300;7..72,400;7..72,500;7..72,600&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #F5F0E8;
            --card-bg: #FAF7F2;
            --text: #2C2C24;
            --olive: #5B6C3E;
            --olive-light: #7C8F5A;
            --burgundy: #6B2D3C;
            --muted: #8A7F72;
            --font-heading: 'Caveat', cursive, serif;
            --font-body: 'Literata', Georgia, serif;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: var(--font-body);
            background-color: var(--bg);
            color: var(--text);
            line-height: 1.7;
            padding: 2rem 1rem;
            min-height: 100vh;
            position: relative;
        }

        /* Paper grain noise */
        body::before {
            content: '';
            position: fixed;
            top: 0; left: 0; right: 0; bottom: 0;
            opacity: 0.03;
            background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.5'/%3E%3C/svg%3E");
            background-repeat: repeat;
            pointer-events: none;
            z-index: 0;
        }

        .container {
            position: relative;
            z-index: 1;
            max-width: 1200px;
            margin: 0 auto;
        }

        header {
            text-align: center;
            margin-bottom: 2.5rem;
            padding-bottom: 1rem;
        }

        h1 {
            font-family: var(--font-heading);
            font-size: 3rem;
            font-weight: 700;
            color: var(--olive);
            letter-spacing: 1px;
            margin-bottom: 0.2rem;
        }

        /* Rough underline for headings */
        .title-underline {
            display: inline-block;
            padding-bottom: 4px;
            background-image: url("data:image/svg+xml,%3Csvg width='200' height='8' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M2 5 Q 20 0, 40 5 T 80 5 T 120 5 T 160 5 T 190 5' stroke='%236B2D3C' fill='none' stroke-width='2.5' opacity='0.4'/%3E%3C/svg%3E");
            background-repeat: repeat-x;
            background-position: bottom;
            background-size: 80px 8px;
        }

        .tagline {
            font-family: var(--font-heading);
            font-size: 1.2rem;
            color: var(--muted);
            margin-top: 0.3rem;
        }

        .view {
            display: none;
        }

        .view.active {
            display: block;
        }

        footer {
            text-align: center;
            margin-top: 3rem;
            padding-top: 1rem;
            font-family: var(--font-heading);
            font-size: 1rem;
            color: var(--muted);
        }

        @media (max-width: 480px) {
            body { padding: 1rem 0.75rem; }
            h1 { font-size: 2.2rem; }
            .tagline { font-size: 1rem; }
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1><span class="title-underline">Basmati</span></h1>
            <p class="tagline">handwritten recipes from an imperfect kitchen</p>
        </header>

        <main id="home" class="view active"></main>
        <main id="recipe-view" class="view"></main>

        <footer>
            <p>&copy; 2026 Moazzam</p>
        </footer>
    </div>
    <script>
        // Recipes + JS logic will go here
    </script>
</body>
</html>
```

- [ ] **Step 2: Verify the file exists and opens**

Run: `Test-Path -LiteralPath "Code Base/index.html"`
Expected: True

Open in browser to confirm the cream background, olive green title with rough underline, and Caveat/Literata fonts load.

---

### Task 2: Recipe data + JS view controller

**Files:**
- Modify: `Code Base/index.html`

- [ ] **Step 1: Add the recipe data array and view-switching JS**

Insert this inside the `<script>` tag:

```javascript
const recipes = [
    {
        id: 'basmati-rice',
        title: 'Basmati Rice with Scrambled Eggs and Tuna',
        chef: 'Moazzam',
        description: 'Fluffy basmati rice, soft scrambled eggs, and flaked tuna — a humble, satisfying one-pan meal.',
        tags: ['Rice', 'Eggs', 'Tuna', 'Quick'],
        emoji: '🍚',
        ingredients: [
            '1 cup Basmati rice',
            '1.5 cups water (for cooking)',
            'Cold water (for soaking and rinsing)',
            '2 fresh eggs',
            '1 can of tuna (packed in oil or water)',
            '1 tbsp cooking oil',
            'Salt (to taste, optional)'
        ],
        steps: [
            ['Soak the Rice', [
                'Retrieve the sealed package of Basmati rice and open it.',
                'Measure 1 cup of dry rice using a measuring cup.',
                'Place the rice in a mixing bowl and fill with cold water until the water level is at least 2 cm above the rice.',
                'Let the rice soak for exactly 60 minutes.',
                'After soaking, drain the water using a fine-mesh strainer.',
                'Return the drained rice to the bowl.'
            ]],
            ['Rinse the Rice', [
                'Cover the rice with fresh cold water.',
                'Gently agitate the rice for 10 seconds to rinse off excess starch.',
                'Drain through a fine-mesh strainer.',
                'Repeat the rinsing process a second time.',
                'Ensure the draining water runs clear, then let the strainer sit for 2 minutes to drain excess water.'
            ]],
            ['Cook the Rice', [
                'Transfer the cleaned rice to a cooking pot.',
                'Add 1.5 cups of fresh water.',
                'Place the pot on a stovetop burner over high heat and bring to a rolling boil.',
                'Cover with a tight-fitting lid and immediately reduce the heat to the lowest setting.',
                'Cook for exactly 12 minutes without lifting the lid.',
                'Turn off the heat and let the pot steam for another 5 minutes with the lid on.',
                'Remove the lid and fluff the rice with a fork.'
            ]],
            ['Prepare the Eggs', [
                'Crack 2 eggs into a small bowl.',
                'Whisk until the yolks and whites are fully blended into a uniform yellow liquid.'
            ]],
            ['Prepare the Tuna', [
                'Open the can of tuna and drain all excess oil or water.',
                'Flake the tuna into small pieces using a fork.'
            ]],
            ['Combine and Serve', [
                'Heat 1 tablespoon of cooking oil in a frying pan over medium heat for 1 minute.',
                'Pour the whisked eggs into the pan and scramble until soft curds form.',
                'Add the cooked Basmati rice to the pan with the scrambled eggs.',
                'Add the flaked tuna to the pan.',
                'Gently fold and stir all ingredients together over medium-low heat for 2 minutes.',
                'Turn off the heat and transfer the finished mixture to a serving plate.'
            ]]
        ]
    }
];

let currentFilter = null;

function showHome() {
    document.getElementById('recipe-view').classList.remove('active');
    document.getElementById('home').classList.add('active');
    document.getElementById('recipe-view').innerHTML = '';
    window.location.hash = '';
}

function showRecipe(id) {
    const recipe = recipes.find(r => r.id === id);
    if (!recipe) return;
    window.location.hash = 'recipe-' + id;
    document.getElementById('home').classList.remove('active');
    const container = document.getElementById('recipe-view');
    container.innerHTML = '';
    container.appendChild(buildRecipeDetail(recipe));
    container.classList.add('active');
    window.scrollTo({ top: 0, behavior: 'smooth' });
}

// On load, check for hash
window.addEventListener('DOMContentLoaded', () => {
    const match = window.location.hash.match(/^#recipe-(.+)$/);
    if (match) {
        const recipe = recipes.find(r => r.id === match[1]);
        if (recipe) showRecipe(recipe.id);
    }
});
```

- [ ] **Step 2: Verify no syntax errors**

Open `Code Base/index.html` in a browser and check the console (F12) — no errors expected.

---

### Task 3: Build the home grid with recipe cards

**Files:**
- Modify: `Code Base/index.html`

- [ ] **Step 1: Add the grid CSS**

Insert into the `<style>` block (after the header/footer styles, before `@media`):

```css
.filter-bar {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    justify-content: center;
    margin-bottom: 2rem;
}

.filter-btn {
    font-family: var(--font-heading);
    font-size: 1.1rem;
    padding: 0.3rem 1rem;
    border: 2px solid var(--olive);
    border-radius: 50px;
    background: transparent;
    color: var(--olive);
    cursor: pointer;
    transition: all 0.2s;
}

.filter-btn:hover {
    background: var(--olive);
    color: var(--card-bg);
}

.filter-btn.active {
    background: var(--burgundy);
    border-color: var(--burgundy);
    color: var(--card-bg);
}

.recipe-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 1.5rem;
}

.recipe-card {
    background: var(--card-bg);
    padding: 1.5rem;
    border: 2.5px dashed var(--olive-light);
    border-radius: 12px;
    cursor: pointer;
    transition: all 0.25s ease;
    position: relative;
}

.recipe-card:nth-child(odd) {
    transform: rotate(-0.5deg);
}

.recipe-card:nth-child(even) {
    transform: rotate(0.7deg);
}

.recipe-card:hover {
    transform: rotate(0deg) scale(1.02);
    box-shadow: 4px 6px 12px rgba(0,0,0,0.08), -2px 3px 6px rgba(0,0,0,0.04);
    border-style: solid;
    border-color: var(--olive);
}

.recipe-card.hidden {
    display: none;
}

.recipe-card .emoji {
    font-size: 2.5rem;
    display: block;
    margin-bottom: 0.5rem;
}

.recipe-card h2 {
    font-family: var(--font-heading);
    font-size: 1.6rem;
    font-weight: 600;
    color: var(--olive);
    border: none;
    padding: 0;
    margin-bottom: 0.3rem;
}

.recipe-card .description {
    font-size: 0.9rem;
    color: var(--muted);
    margin-bottom: 0.8rem;
    line-height: 1.5;
}

.recipe-card .tags {
    display: flex;
    flex-wrap: wrap;
    gap: 0.4rem;
}

.recipe-card .tag {
    font-family: var(--font-heading);
    font-size: 0.9rem;
    padding: 0.1rem 0.6rem;
    border: 1.5px solid var(--olive-light);
    border-radius: 50px;
    color: var(--olive);
}

@media (max-width: 480px) {
    .recipe-grid {
        grid-template-columns: 1fr;
        gap: 1rem;
    }

    .recipe-card {
        padding: 1rem;
    }

    .recipe-card:nth-child(odd),
    .recipe-card:nth-child(even) {
        transform: none;
    }
}
```

- [ ] **Step 2: Add the renderHome function in the `<script>` tag**

Insert after the `showRecipe` function:

```javascript
function renderHome() {
    const home = document.getElementById('home');
    home.innerHTML = '';

    // Build tag filter
    const allTags = [...new Set(recipes.flatMap(r => r.tags))].sort();
    const filterBar = document.createElement('nav');
    filterBar.className = 'filter-bar';

    const allBtn = document.createElement('button');
    allBtn.className = 'filter-btn active';
    allBtn.textContent = 'All';
    allBtn.dataset.tag = '';
    allBtn.addEventListener('click', () => filterByTag(''));
    filterBar.appendChild(allBtn);

    allTags.forEach(tag => {
        const btn = document.createElement('button');
        btn.className = 'filter-btn';
        btn.textContent = tag;
        btn.dataset.tag = tag;
        btn.addEventListener('click', () => filterByTag(tag));
        filterBar.appendChild(btn);
    });

    home.appendChild(filterBar);

    // Build grid
    const grid = document.createElement('div');
    grid.className = 'recipe-grid';

    recipes.forEach(recipe => {
        const card = document.createElement('div');
        card.className = 'recipe-card';
        card.dataset.id = recipe.id;

        const emoji = document.createElement('span');
        emoji.className = 'emoji';
        emoji.textContent = recipe.emoji;
        card.appendChild(emoji);

        const title = document.createElement('h2');
        title.textContent = recipe.title;
        card.appendChild(title);

        const desc = document.createElement('p');
        desc.className = 'description';
        desc.textContent = recipe.description;
        card.appendChild(desc);

        const tags = document.createElement('div');
        tags.className = 'tags';
        recipe.tags.forEach(tag => {
            const span = document.createElement('span');
            span.className = 'tag';
            span.textContent = tag;
            tags.appendChild(span);
        });
        card.appendChild(tags);

        card.addEventListener('click', () => showRecipe(recipe.id));
        grid.appendChild(card);
    });

    home.appendChild(grid);
}

function filterByTag(tag) {
    currentFilter = tag;
    document.querySelectorAll('.filter-btn').forEach(btn => {
        btn.classList.toggle('active', btn.dataset.tag === tag);
    });
    document.querySelectorAll('.recipe-card').forEach(card => {
        const recipe = recipes.find(r => r.id === card.dataset.id);
        if (!recipe) return;
        if (!tag) {
            card.classList.remove('hidden');
        } else {
            card.classList.toggle('hidden', !recipe.tags.includes(tag));
        }
    });
}
```

- [ ] **Step 3: Wire up renderHome in the DOMContentLoaded handler**

Update the event listener:

```javascript
window.addEventListener('DOMContentLoaded', () => {
    renderHome();
    const match = window.location.hash.match(/^#recipe-(.+)$/);
    if (match) {
        const recipe = recipes.find(r => r.id === match[1]);
        if (recipe) showRecipe(recipe.id);
    }
});
```

- [ ] **Step 4: Verify in browser**

Open `Code Base/index.html`. Expected: the home grid shows one recipe card (Basmati Rice) with emoji, title, description, and tags. The filter bar shows "All" + all tags. Card has dashed olive border and slight rotation.

---

### Task 4: Build the recipe detail view

**Files:**
- Modify: `Code Base/index.html`

- [ ] **Step 1: Add recipe detail CSS**

Insert into `<style>` after the recipe-grid styles:

```css
.back-btn {
    font-family: var(--font-heading);
    font-size: 1.3rem;
    color: var(--olive);
    background: none;
    border: none;
    cursor: pointer;
    padding: 0.3rem 0.6rem;
    margin-bottom: 1.5rem;
    display: inline-block;
    transition: transform 0.2s;
}

.back-btn:hover {
    transform: translateX(-4px);
}

.recipe-detail {
    max-width: 720px;
    margin: 0 auto;
    animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
    from { opacity: 0; transform: translateY(12px); }
    to { opacity: 1; transform: translateY(0); }
}

.recipe-detail .detail-header {
    text-align: center;
    margin-bottom: 2rem;
}

.recipe-detail .detail-header .emoji {
    font-size: 3.5rem;
    display: block;
    margin-bottom: 0.5rem;
}

.recipe-detail .detail-header h2 {
    font-family: var(--font-heading);
    font-size: 2.2rem;
    font-weight: 600;
    color: var(--olive);
    border: none;
    padding: 0;
    text-align: center;
    margin-bottom: 0.3rem;
}

.recipe-detail .detail-header .chef {
    font-family: var(--font-heading);
    font-size: 1.1rem;
    color: var(--muted);
    margin-bottom: 0.5rem;
}

.recipe-detail .detail-header .tags {
    display: flex;
    flex-wrap: wrap;
    gap: 0.4rem;
    justify-content: center;
    margin-bottom: 1rem;
}

.recipe-detail .detail-header .tag {
    font-family: var(--font-heading);
    font-size: 0.95rem;
    padding: 0.15rem 0.7rem;
    border: 1.5px solid var(--olive-light);
    border-radius: 50px;
    color: var(--olive);
}

.ingredients-box {
    background: var(--card-bg);
    border: 2px dashed var(--olive-light);
    border-radius: 12px;
    padding: 1.5rem 2rem;
    margin-bottom: 2rem;
}

.ingredients-box h3 {
    font-family: var(--font-heading);
    font-size: 1.5rem;
    color: var(--burgundy);
    margin-bottom: 0.8rem;
    border-bottom: 2px dashed var(--olive-light);
    padding-bottom: 0.4rem;
}

.ingredients-box ul {
    list-style: none;
    padding: 0;
}

.ingredients-box li {
    padding: 0.3rem 0;
    padding-left: 1.8rem;
    position: relative;
    font-size: 0.95rem;
}

.ingredients-box li::before {
    content: '✓';
    position: absolute;
    left: 0;
    color: var(--burgundy);
    font-family: var(--font-heading);
    font-size: 1.1rem;
}

.step-group {
    background: var(--card-bg);
    border: 2px dashed var(--olive-light);
    border-radius: 12px;
    padding: 1.2rem 1.8rem;
    margin-bottom: 1.2rem;
}

.step-group h3 {
    font-family: var(--font-heading);
    font-size: 1.3rem;
    color: var(--olive);
    margin-bottom: 0.5rem;
}

.step-group ol {
    padding-left: 1.5rem;
}

.step-group li {
    margin-bottom: 0.4rem;
    font-size: 0.95rem;
    line-height: 1.6;
}

@media (max-width: 480px) {
    .recipe-detail .detail-header h2 { font-size: 1.6rem; }
    .ingredients-box { padding: 1rem 1.2rem; }
    .step-group { padding: 1rem 1.2rem; }
}
```

- [ ] **Step 2: Add the `buildRecipeDetail` function in `<script>`**

Insert before `showHome`:

```javascript
function buildRecipeDetail(recipe) {
    const div = document.createElement('div');
    div.className = 'recipe-detail';

    // Back button
    const backBtn = document.createElement('button');
    backBtn.className = 'back-btn';
    backBtn.innerHTML = '← Back';
    backBtn.addEventListener('click', showHome);
    div.appendChild(backBtn);

    // Header
    const header = document.createElement('div');
    header.className = 'detail-header';

    const emoji = document.createElement('span');
    emoji.className = 'emoji';
    emoji.textContent = recipe.emoji;
    header.appendChild(emoji);

    const title = document.createElement('h2');
    title.textContent = recipe.title;
    header.appendChild(title);

    const chef = document.createElement('p');
    chef.className = 'chef';
    chef.textContent = 'Chef: ' + recipe.chef;
    header.appendChild(chef);

    const tags = document.createElement('div');
    tags.className = 'tags';
    recipe.tags.forEach(t => {
        const span = document.createElement('span');
        span.className = 'tag';
        span.textContent = t;
        tags.appendChild(span);
    });
    header.appendChild(tags);

    div.appendChild(header);

    // Ingredients
    const ingBox = document.createElement('div');
    ingBox.className = 'ingredients-box';
    const ingTitle = document.createElement('h3');
    ingTitle.textContent = 'Ingredients';
    ingBox.appendChild(ingTitle);
    const ingList = document.createElement('ul');
    recipe.ingredients.forEach(ing => {
        const li = document.createElement('li');
        li.textContent = ing;
        ingList.appendChild(li);
    });
    ingBox.appendChild(ingList);
    div.appendChild(ingBox);

    // Steps
    recipe.steps.forEach(([stepTitle, stepItems]) => {
        const group = document.createElement('div');
        group.className = 'step-group';
        const h3 = document.createElement('h3');
        h3.textContent = stepTitle;
        group.appendChild(h3);
        const ol = document.createElement('ol');
        stepItems.forEach(item => {
            const li = document.createElement('li');
            li.textContent = item;
            ol.appendChild(li);
        });
        group.appendChild(ol);
        div.appendChild(group);
    });

    return div;
}
```

- [ ] **Step 3: Verify in browser**

Open `Code Base/index.html`, click the recipe card. Expected: smooth fade-in of recipe detail with "← Back" button, emoji, title, chef, tags, ingredients list with burgundy checkmarks, and step groups. Click "← Back" to return to grid.

---

### Task 5: Add tag filtering

**Files:**
- Modify: `Code Base/index.html`

Note: the filter UI and `filterByTag` function were already built in Task 3. This task verifies and tweaks it.

- [ ] **Step 1: Verify filtering works**

Open `Code Base/index.html`. Click each filter button. Expected: only recipes with matching tags show. "All" shows everything. Active button has burgundy background. Hidden cards use `display: none` and don't break grid layout.

---

### Task 6: Add Pistachio Croissant and Pistachio Gelato recipes

**Files:**
- Modify: `Code Base/index.html`

- [ ] **Step 1: Add two new recipes to the `recipes` array**

Insert these objects after the Basmati Rice object (before the closing `];`):

```javascript
    {
        id: 'pistachio-croissant',
        title: 'Pistachio Croissant',
        chef: 'Moazzam',
        description: 'Buttery, flaky croissant filled with rich pistachio cream — a bakery treat made at home.',
        tags: ['Pastry', 'Pistachio', 'Breakfast'],
        emoji: '🥐',
        ingredients: [
            '1 sheet puff pastry (thawed)',
            '1/2 cup shelled pistachios',
            '2 tbsp powdered sugar',
            '2 tbsp unsalted butter (softened)',
            '1 egg (for egg wash)',
            '1 tbsp milk',
            '1/4 tsp vanilla extract',
            'Chopped pistachios for topping'
        ],
        steps: [
            ['Make the Pistachio Cream', [
                'Grind the shelled pistachios in a food processor until fine.',
                'Add powdered sugar, softened butter, and vanilla extract.',
                'Blend until a smooth paste forms. Set aside.'
            ]],
            ['Prepare the Pastry', [
                'Roll out the puff pastry sheet on a lightly floured surface.',
                'Cut the pastry into 4 triangles.',
                'Spread 1 tablespoon of pistachio cream at the wide end of each triangle.'
            ]],
            ['Shape the Croissants', [
                'Roll each triangle from the wide end toward the tip.',
                'Curve the ends inward to form a crescent shape.',
                'Place on a parchment-lined baking sheet.'
            ]],
            ['Bake', [
                'Preheat oven to 200°C (400°F).',
                'Beat the egg with milk to make an egg wash.',
                'Brush each croissant with egg wash.',
                'Sprinkle chopped pistachios on top.',
                'Bake for 15-18 minutes until golden brown.',
                'Let cool on a wire rack for 5 minutes before serving.'
            ]]
        ]
    },
    {
        id: 'pistachio-gelato',
        title: 'Pistachio Gelato',
        chef: 'Moazzam',
        description: 'Silky, creamy Italian-style gelato with real pistachio flavour — no ice cream maker required.',
        tags: ['Dessert', 'Pistachio', 'Frozen'],
        emoji: '🍦',
        ingredients: [
            '2 cups whole milk',
            '1 cup heavy cream',
            '3/4 cup sugar',
            '1/2 cup shelled pistachios (unsalted)',
            '4 egg yolks',
            '1/2 tsp almond extract',
            'Pinch of salt'
        ],
        steps: [
            ['Infuse the Pistachios', [
                'Grind pistachios in a food processor until very fine.',
                'In a saucepan, combine milk, cream, and ground pistachios.',
                'Heat over medium heat until warm, stirring occasionally.',
                'Remove from heat, cover, and let steep for 30 minutes.'
            ]],
            ['Make the Custard Base', [
                'In a bowl, whisk egg yolks with sugar until pale and thick.',
                'Strain the pistachio milk mixture through a fine-mesh sieve into a clean pot, pressing on the solids.',
                'Rewarm the strained milk over medium heat until steaming.',
                'Slowly pour the warm milk into the egg mixture, whisking constantly.',
                'Return everything to the pot and cook over low heat, stirring, until it thickens enough to coat the back of a spoon.'
            ]],
            ['Chill and Freeze', [
                'Stir in almond extract and a pinch of salt.',
                'Pour through a sieve into a bowl and let cool to room temperature.',
                'Refrigerate for at least 4 hours (or overnight).',
                'Pour into a shallow dish and freeze for 2 hours.',
                'Stir vigorously with a fork to break up ice crystals.',
                'Repeat freezing and stirring every 30 minutes for 2-3 hours until creamy.',
                'Cover and freeze until firm (at least 4 more hours).'
            ]]
        ]
    }
```

- [ ] **Step 2: Verify in browser**

Open `Code Base/index.html`. Expected: three recipe cards on the grid. Filter by "Pistachio" shows two recipes. Filter by "Rice" shows one. Click each recipe to verify full detail view renders correctly with ingredients and steps.

---

### Task 7: Final polish and responsive verification

**Files:**
- Modify: `Code Base/index.html`

- [ ] **Step 1: Add a `font-display: swap` optimization**

In the Google Fonts link, already handled by the `&display=swap` parameter. Confirm visually that text is visible during font load.

- [ ] **Step 2: Verify responsive layout**

Resize browser to mobile width (~375px). Expected:
- Header scales down
- Grid becomes single column
- Cards lose rotation on mobile
- Filter bar wraps to multiple rows
- Recipe detail fits within viewport with no horizontal scroll

- [ ] **Step 3: Final open and visual check**

Open `Code Base/index.html` and confirm:
- Olive green + cream + burgundy palette is consistent
- Caveat headings render with handwritten feel
- Cards have dashed borders and slight rotation
- Paper grain texture is visible (very subtle)
- Recipe detail has burgundy checkmark bullets
- Back button navigates correctly
- URL hash updates and works on page reload
