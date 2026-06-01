# Template Walkthrough

This is a guided tour of `assets/article-template.html`. Read alongside the file - this doc explains *why* each block exists and *what* you should fill in. The template captures structural invariants that are easy to break (sticky offsets, DPR scaling, scroll tracking); start from it, don't rebuild from scratch.

## How to Use the Template

1. Decide on the output path (e.g. `my-explainer/index.html`).
2. Copy the template to that path: `cp <skill-path>/assets/article-template.html my-explainer/index.html`.
3. Replace every `{{PLACEHOLDER}}` with real content. Search for `{{` to find them all.
4. Add sections, figures, and prose by duplicating the patterns shown.

## Block-by-Block

### `<head>` block

- **`<title>`**: `{{TITLE}} - {{SUBTITLE_SHORT}}` shows in the browser tab and search results. Subtitle short = a 5-8 word version of the subtitle.
- **Google Fonts `<link>`**: Loads Crimson Pro and JetBrains Mono. Don't remove. Don't add other fonts.
- **`:root` custom properties**: All theming lives here. Pick accent colors from `references/color-palettes.md` and replace the four `{{ACCENT*}}` placeholders.

### CSS classes you'll use

The template defines every class you need. You don't need to invent new ones for typical content. Quick map:

| Class | Used for |
|-------|----------|
| `.lead` | First paragraph of a section, slightly larger and gray |
| `.callout` | "Key insight" box with accent left-border |
| `.epigraph` | Pull-quote with attribution |
| `.figure` + children | Interactive figure container |
| `.code-block` | Multi-line code with syntax-color spans |
| `.comparison-grid` + `.comparison-cell` | Side-by-side cells (e.g., "before vs. after") |
| `.concept-cards` + `.concept-card` | 3-up grid for introducing concepts |

If you need something that isn't here, prefer adding a small inline class to the article's `<style>` over restructuring.

### Sidebar TOC

```html
<aside class="sidebar">
  <nav id="toc">
    <a href="#section-one">Section title</a>
    <a href="#fig-one" class="sub">Figure title</a>
  </nav>
</aside>
```

One `<a>` per `<h2 id="...">`. Use `class="sub"` for `<h3>` children. The IDs in `href="#..."` must match the actual section IDs - the scroll-tracking script reads them. Two sub-entries per major section is plenty; if you have 5+, the section is too dense and should be split.

### Article header

```html
<header class="article-header">
  <div class="overline">{{OVERLINE}}</div>
  <h1>{{TITLE}}</h1>
  <p class="article-subtitle">{{SUBTITLE}}</p>
  <div class="article-meta-header">
    <span>{{DATE}}</span>
    <span>{{READ_TIME}} min read</span>
    <span>{{FIGURE_COUNT}} interactive figures</span>
  </div>
</header>
```

- **Overline**: Short attribution above the title - "Stanford NLP / DSPy Framework", "Google DeepMind / Genie", etc. Two-part if you can: organization + project. All caps, accent-color, small.
- **Title**: 3-12 words. Punchy, claim-shaped. Not a question unless the question is the point.
- **Subtitle**: One italic sentence. The "tl;dr" of the article. Promises what the reader will get.
- **Meta**: Date as "Month YYYY" (e.g., "April 2026"). Read time in whole minutes. Figure count helps the reader budget.

### Sections

```html
<section id="section-one">
  <h2>Section title</h2>
  <p class="lead">Opening line, larger and grayer.</p>
  <p>Body paragraph.</p>
  ...
  <div class="figure" id="fig-one"> ... </div>
</section>
```

Each `<section>` gets a stable `id` - the sidebar TOC uses these. Use a descriptive slug (`#training`, `#evaluation`, not `#section-3`).

The `.lead` class is for the *first paragraph* of the article, and optionally the first paragraph of each major section. It's a sizing hint - a slightly larger, slightly grayer paragraph that signals "you're starting something new". Don't put it on every paragraph.

### Figure block

```html
<div class="figure" id="fig-one">
  <div class="figure-label">Figure 1 - Title</div>
  <div class="figure-canvas-wrap">
    <canvas id="canvas-one" width="720" height="384"></canvas>
  </div>
  <div class="figure-controls">
    <button class="ctrl-btn primary" id="one-step">Step</button>
    <button class="ctrl-btn" id="one-play">Play</button>
    <button class="ctrl-btn" id="one-reset">Reset</button>
    <div class="ctrl-separator"></div>
    <span class="ctrl-label" id="one-info">Step: 0</span>
  </div>
  <div class="figure-caption">
    Caption with <strong>bold key term</strong> the reader should track.
  </div>
</div>
```

Invariants:
- `width="720" height="384"` is the standard aspect (close to 16:9). Use this unless you have a strong reason. Square (`512x512`) and tall (`520x600`) are sometimes right - just don't go wider than 720 (it'll exceed the content column).
- Canvas ID matches the figure ID's slug: `fig-one` → `canvas-one`. Buttons use the same prefix: `one-step`, `one-play`, etc.
- Figures are numbered globally across the article: Figure 1, Figure 2, Figure 3...
- The `figure-label` is a typographic detail (small uppercase). Don't put the full descriptive title here - that goes in the caption.

### Footer

```html
<footer class="article-footer">
  <p>
    Attribution / source links / acknowledgements.
  </p>
</footer>
```

Optional. Use it for source links and acknowledgements when relevant; delete the block entirely if the article has nothing to attribute.

### Script block

The script block has three parts:

1. **`initCanvas()` utility** - shared by every figure. Don't modify.
2. **One IIFE per figure** - see `references/figure-archetypes.md` for patterns. Each IIFE is independent; they don't share state.
3. **Sidebar scroll tracking IIFE** - reads `#toc a` entries and highlights the section in view. Don't modify unless you need a different scroll offset (the `+ 80` in `scrollY + 80` is the trigger zone above the section top).

Order matters: `initCanvas` must come before any IIFE that uses it. The scroll-tracking IIFE goes last.

## Adding a Section

1. Add `<a href="#new-section">New Section</a>` to `#toc` in the right position.
2. Add the `<section id="new-section">` block in the corresponding spot in `<main>`.
3. If the section has sub-anchors (e.g., a figure ID), add them as `<a class="sub">` immediately after the parent in the TOC.

## Adding a Figure

1. Add the figure block (HTML above) in a `<section>`.
2. Add a TOC entry with `class="sub"` linking to the figure's `id`.
3. Add an IIFE in the script block. Pick the closest archetype from `references/figure-archetypes.md`.
4. Update the figure count in `.article-meta-header`.

## Common Edits

- **Different overline format**: Plain text works. A slash separator (`Group / Project`) is the conventional pattern.
- **Need a wider canvas**: You can't, content column is capped at 720px. Use the `figure-canvas-wrap` overflow-scroll pattern (already in CSS) and `min-width: 800px` on the canvas to let mobile users scroll horizontally.
- **Need a hero figure above the article header**: Don't. The article-header animations rely on it being the first thing.

## Don't

- Don't add external CSS or JS files for the article. Single-file is the convention.
- Don't move `initCanvas` out of the script block. Each article is self-contained.
- Don't use SVG figures. The constraint is Canvas + custom rendering. SVGs work for static diagrams elsewhere, not for the interactive figures here.
- Don't change the page background, the typography stack, or the responsive breakpoints. They're shared invariants across the template.
