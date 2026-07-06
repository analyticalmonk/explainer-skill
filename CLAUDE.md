# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a **Claude Code and Codex plugin package** with three skills that produce single self-contained interactive HTML explainer pages in the style of distill.pub. The repo root holds `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, and `.codex-plugin/plugin.json`; each skill lives at `skills/<name>/SKILL.md` plus its `references/` (and `assets/` for the hub). That layout also satisfies the universal `skills/<name>/SKILL.md` convention, so the same checkout loads as plain skills in Codex, Cursor, Gemini CLI, etc. There is no source code to build, no test suite, no lint config.

The output of each skill is a generic standalone HTML file. The skills do not assume any parent project, multi-article index, or shared infrastructure.

## The three skills and how they relate

- **creating-explainers** is the hub. It owns the output format (the HTML template, figure archetypes, voice, color palettes) and the `intake -> outline -> scaffold -> prose -> figures -> fact-check -> polish` workflow. It handles file intake, research intake, and the mix of both.
- **explaining-codebases** extends the hub for code. It adds code intake and code-specific figures, and defers to `creating-explainers` for everything about the output.
- **fact-checking-explainers** is the gate both must pass before delivery. It is discipline-enforcing: no explainer ships with an unverified factual claim.

Skills cross-reference each other **by name** (`fact-checking-explainers`), never with `@`-includes (which force-load files and burn context).

## Editing these skills

- `skills/<name>/SKILL.md` - workflow, constraints, when-to-use rules. The frontmatter `description` is what triggers the skill, so be precise when editing it. Keep descriptions to when-to-use, not a workflow summary.
- `skills/*/references/*.md` - lazy-loaded deep references. The SKILL.md files tell Claude to read these only when relevant, so the skills stay light on context. Keep that property: don't inline reference content into a SKILL.md, and don't cross-link references in ways that force-load extras.
- `skills/creating-explainers/assets/article-template.html` - the canonical article skeleton with `{{PLACEHOLDERS}}`. Structural invariants (DPR-aware `initCanvas()`, sidebar TOC scroll-tracking, 1060px and 600px responsive breakpoints) live here. If output articles keep getting a layout invariant wrong, fix the template, not the article.
- `.claude-plugin/plugin.json` - the plugin manifest. Bump `version` on releases.
- `.claude-plugin/marketplace.json` - the marketplace entry. Keep its `skills` list and `description` in sync with the actual skills and with `plugin.json`.
- `.codex-plugin/plugin.json` - the Codex plugin manifest. Keep its `skills`, `version`, description, and interface metadata in sync with the Claude plugin metadata and README.
- `skills/<name>/agents/openai.yaml` - Codex UI metadata for each skill. Keep display names, short descriptions, and default prompts aligned with the matching `SKILL.md`.

`README.md` is for humans installing the plugin - keep it in sync when the skill layout changes.

## House constraints (the skills fail if these break)

- **Zero dependencies in produced articles.** Vanilla HTML/CSS/JS only. No npm, React, D3, Three.js, Tailwind, SVG libraries. The only external network call in a produced article is the Google Fonts stylesheet.
- **No em-dashes** (the long dash) anywhere - in any SKILL.md, reference, the template, or article output. Use regular hyphens or rewrite. This is a hard style rule.
- **Single-file articles**: all CSS in one `<style>` block, all JS at the bottom of `<body>`. Each interactive figure is a self-contained IIFE; the only shared utility is `initCanvas(id)`.
- **No unverified factual claim ships.** `fact-checking-explainers` is a hard gate, not advisory. `creating-explainers` and `explaining-codebases` both run it before delivery, and the article is not done until every checkable claim is supported.

## Testing changes

There is no automated test runner. To validate changes:

1. The five eval prompts live in `evals/evals.json` (three for `creating-explainers`, one for `explaining-codebases`, one for `fact-checking-explainers` with planted errors). Run a skill against one and inspect the output.
2. To iterate locally on a produced article: from the directory containing the produced `index.html`, run `python3 -m http.server 8000` and open it in a browser. The per-skill quality checklists are the canonical pre-delivery checks.
3. For Codex packaging changes, run `python3 /home/analyticalmonk/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .` and the skill validator for each folder under `skills/`.
