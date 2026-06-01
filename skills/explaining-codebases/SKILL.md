---
name: explaining-codebases
description: Use when creating an interactive explainer about a codebase, repository, or set of source files - an onboarding guide to how a project is structured, or a deep-dive on how a specific algorithm or mechanism is implemented in real code. Trigger phrases include "explain this codebase", "interactive guide to this repo", "walk through how X works in the code", "onboarding explainer for this project", "visualize this architecture". Produces the same single self-contained HTML explainer as creating-explainers, but with code navigation and code-specific figures (architecture diagrams, data-flow, execution traces, annotated code walkthroughs). For a paper, topic, or non-code source, use creating-explainers instead.
---

# Explaining Codebases

## Overview

This skill explains code as it actually exists - a repository, a service, or a set of source files. The output is the same distill-style, single self-contained `index.html` explainer that `creating-explainers` produces. What differs is the intake (navigating real code instead of reading a paper) and the figure types (architecture diagrams, data-flow, execution traces, annotated code walkthroughs). It handles both onboarding overviews of a whole codebase and deep-dives on one mechanism.

## Required Background

**REQUIRED:** Use `creating-explainers` for everything about the output - the HTML template, voice and style, base figure archetypes, color palettes, the outline/scaffold/prose/polish workflow, and the quality checklist. This skill only adds what is specific to code. Do not duplicate that material here.

## When to Use

Use when the subject is code:
- **Architecture / onboarding overview** - how a project is structured: its modules, how data flows between them, the key abstractions, how the pieces fit. For a new contributor who needs the map.
- **Single-mechanism deep-dive** - how one algorithm, feature, or subsystem works, traced through the real implementation.

The skill picks the angle from the request and confirms it with the user during intake (overview and deep-dive want different figures and different depth).

**When NOT to use:** a paper, a topic, or any non-code source. Use `creating-explainers` instead.

## Code Intake

See `references/code-intake.md`. In short: pick the angle, navigate the repository to find entry points and module structure, identify the spine concept (the one path or idea the article tracks), map the architecture, and pull real code snippets anchored to `path:line`. Quote actual code; never paraphrase code as if quoting it.

## Code-Specific Figures

See `references/code-figure-archetypes.md` for the patterns:
- **Architecture / module diagram** - modules as boxes, dependencies as arrows
- **Data-flow / sequence diagram** - an item (request, message, token) moving through stages
- **Execution-trace stepper** - step through an algorithm with state and the current line highlighted
- **Annotated code walkthrough** - a code block whose lines reveal annotations as you step or hover

The mechanics (DPR-aware `initCanvas`, the IIFE pattern, animation loops) come from the base `figure-archetypes.md` in `creating-explainers`. The code-figure reference only covers how to apply them to code.

## The Fact-Check Gate for Code

**REQUIRED before delivery:** run `fact-checking-explainers`. For a codebase explainer the source of truth is the code itself. Every claim about what the code does, and every quoted snippet, is checked against the actual implementation at a specific path and line. Code drifts; a snippet that was accurate yesterday may be wrong today. The explainer is not done until it passes.

## Workflow

Same staged workflow as `creating-explainers`, with code intake in place of the text intakes:

```
1. Code intake           -> navigate, pick angle, find the spine, map architecture, pull real snippets
2. Outline               -> propose sections + figure list, get user approval
3. Scaffold              -> copy the article template, fill metadata
4. Prose pass            -> write all sections with figure placeholders
5. Figures pass          -> implement each interactive figure
6. Post-draft fact-check -> verify every claim and snippet against the real code (REQUIRED, blocking)
7. Polish                -> captions, cross-links, mobile check
```

Pause for user approval after the outline. Everything about scaffolding, prose, figures, and polish follows `creating-explainers`.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Explaining code that does not exist, or that you imagined | Open the files. Every claim traces to real code at a real path. |
| Snippets drift from the real source | Quote exactly, with `path:line`. Never tidy code into something the repo does not contain. |
| Architecture diagram does not match the real module structure | Build the diagram from the actual imports and call sites, not a guess. |
| Scope too broad (a whole framework in one article) | Pick one subsystem or one mechanism. Breadth dilutes; depth teaches. |
| Pasting a huge file as a "figure" | Figures illustrate a point the prose just set up. Trim to the lines that matter. |
