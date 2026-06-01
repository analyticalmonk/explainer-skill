# Color Palettes

Each article picks one **accent color** that signals the topic, plus a small set of secondary colors for distinguishing concepts within the article. Pick early, before drafting figures - the accent shows up in figure rendering, callouts, links, and the article overline.

## Accent Color Sets

Each set is `--accent` / `--accent-light` / `--accent-border` / `--accent-dark`. Pick the set whose feel matches the topic.

| Theme | accent | accent-light | accent-border | accent-dark | Used in |
|-------|--------|--------------|---------------|-------------|---------|
| **Blue** (technical, neutral, "frameworks") | `#2563EB` | `#EFF6FF` | `#BFDBFE` | `#1D4ED8` | dspy |
| **Orange** (warm, embodied, "physical") | `#C2410C` | `#FFF7ED` | `#FED7AA` | `#9A3412` | world-models, pi-agent |
| **Green** (organic, "data", success) | `#059669` | `#ECFDF5` | `#A7F3D0` | `#047857` | autoresearch |
| **Purple** (theoretical, "abstract") | `#7C3AED` | `#F5F0FF` | `#DDD6FE` | `#6D28D9` | (available) |
| **Red** (urgent, contrarian, alarm) | `#DC2626` | `#FEF2F2` | `#FECACA` | `#B91C1C` | (use sparingly) |
| **Teal** (cool, exploratory) | `#0891B2` | `#ECFEFF` | `#A5F3FC` | `#0E7490` | (available) |
| **Slate** (somber, formal, "history") | `#475569` | `#F1F5F9` | `#CBD5E1` | `#334155` | artemis-ii |

If unsure, default to **blue** for anything technical and **orange** for anything embodied or physical.

## Secondary / Concept Colors

When an article distinguishes 3-4 concepts (DSPy's signatures/modules/optimizers; pi-agent's three planning approaches; world-models' three modeling families), each concept gets its own three-color set: `--{name}-primary`, `--{name}-light`, `--{name}-border`. These show up in concept-card badges, comparison-cell headers, and figure colors.

Standard pairings (use these rather than picking freehand):

| Name | primary | light | border |
|------|---------|-------|--------|
| Amber | `#D97706` | `#FEF3C7` | `#FDE68A` |
| Purple | `#7C3AED` | `#F5F0FF` | `#DDD6FE` |
| Green | `#059669` | `#ECFDF5` | `#A7F3D0` |
| Blue | `#2563EB` | `#EFF6FF` | `#BFDBFE` |
| Pink | `#DB2777` | `#FDF2F8` | `#FBCFE8` |
| Teal | `#0891B2` | `#ECFEFF` | `#A5F3FC` |

Pick concept colors that **contrast with the accent** so the accent stays distinct. If the article accent is blue, don't make a concept blue too - use amber, purple, and green instead.

## How to Pick

1. **Start from the topic feel.** Embodied/physical → orange. Frameworks/code → blue. Organic/data → green. History/somber → slate.
2. **Check what's adjacent.** If the most recent article used blue, pick something else even if blue would fit - readers visit several articles in a row.
3. **Pick concept colors last.** Once you have an accent, the secondary palette falls out by avoiding it.

## CSS Snippet

Drop this into the article's `:root` (replace placeholders from the table):

```css
:root {
  /* ... universal tokens ... */

  --accent: #2563EB;
  --accent-light: #EFF6FF;
  --accent-border: #BFDBFE;
  --accent-dark: #1D4ED8;

  /* Optional concept palettes - only define the ones you use */
  --concept-a-primary: #D97706;
  --concept-a-light:   #FEF3C7;
  --concept-a-border:  #FDE68A;

  --concept-b-primary: #7C3AED;
  --concept-b-light:   #F5F0FF;
  --concept-b-border:  #DDD6FE;
}
```

## Background Color

The page background is `#FAF8F4` (warm off-white) for all articles. Don't change this - the warm tone is part of the look. White (`#fff`) is reserved for figure backgrounds (the "stage" the reader looks at) so they pop against the page.
