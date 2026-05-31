# explain-this: Plugin Design Spec

**Date:** 2026-06-01
**Status:** Approved

## Goal

A zero-dependency Claude Code plugin whose skills produce distill-style
interactive explainers: single self-contained `index.html` pages with
hand-built Canvas figures, a two-column sticky layout, and conversational prose
that teaches a topic deeply. The plugin covers three intake situations:

1. Explaining from a fixed set of files the user provides (paper, blog,
   transcript, report).
2. Explaining after doing web-based research on a topic.
3. Explaining a codebase or set of code files.

A hard requirement cutting across all three: **no incorrect factual claim ships
in any explainer.**

This supersedes the earlier `explain-it` prototype (a single skill), reusing its
mature reference material and restructuring it into three skills.

## The output artifact (reused conventions, unchanged)

Every explainer is one self-contained `index.html`:

- Vanilla HTML/CSS/JS, zero dependencies, no build step (Google Fonts allowed).
- All CSS in one `<style>` block; all JS in one script block at the end of body.
- HTML5 Canvas figures, each a self-contained IIFE, sharing a DPR-aware
  `initCanvas(id)` utility.
- Two-column sticky layout: ~220px sidebar table of contents that scroll-tracks,
  ~720px content column. Responsive breakpoints at 1060px and 600px.
- Crimson Pro serif body, JetBrains Mono for code, warm `#FAF8F4` background.
- Conversational, second-person voice. No em-dashes anywhere.
- Target: roughly 2000-3500 words, 3-5 interactive figures, ~15 minute read.

These conventions live in the reused reference docs and the HTML template; they
are not being redesigned.

## Plugin structure

```
explain-this/
├── .claude-plugin/
│   ├── plugin.json                 # name: explain-this
│   └── marketplace.json
├── skills/
│   ├── creating-explainers/        # CORE / hub: owns output format + workflow
│   │   ├── SKILL.md
│   │   ├── references/
│   │   │   ├── intake-from-files.md       (from prior document-mode.md)
│   │   │   ├── intake-from-research.md     (from prior research-mode.md)
│   │   │   ├── figure-archetypes.md        (reused as-is)
│   │   │   ├── voice-and-style.md          (reused as-is)
│   │   │   ├── template-walkthrough.md     (reused as-is)
│   │   │   └── color-palettes.md           (reused as-is)
│   │   └── assets/
│   │       └── article-template.html       (reused as-is)
│   ├── explaining-codebases/       # code intake + code-specific figures
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── code-intake.md
│   │       └── code-figure-archetypes.md
│   └── fact-checking-explainers/   # discipline-enforcing verification gate
│       ├── SKILL.md
│       └── references/
│           └── verification-report-format.md
├── evals/
│   └── evals.json
├── CLAUDE.md
├── README.md
├── LICENSE
└── docs/
    └── specs/
        └── 2026-06-01-explain-this-plugin-design.md   (this file)
```

## Skills

### 1. creating-explainers (core / hub)

The skill that owns the *output*: the HTML template, the figure archetypes, the
voice and style, the color palettes, and the end-to-end workflow:

```
intake (files, research, or both) -> outline (approval gate) -> scaffold
       -> prose -> figures -> post-draft fact-check (gate) -> polish -> deliver
```

Intake is one phase: gather the source material, which may be given files, web
research, or both. Two references cover the two material sources, and they
compose:

- `intake-from-files.md`: the user hands over a paper, blog post, transcript, or
  report. Read it, extract the spine concept, key ideas, concrete moments, and
  figure opportunities.
- `intake-from-research.md`: the user gives only a topic. Gather 3-5 high-quality
  sources via web search/fetch, then synthesize. Hands gathered sources to the
  research-time fact-check before drafting.

**Mixed intake (files and research together):** when the user provides files
*and* the topic needs material the files do not cover (recent developments,
criticisms, corroboration), read both references and combine them. Designate a
spine (usually the given files) and treat the rest as supporting; synthesize
across everything into one outline and one article. The research-time fact-check
runs on the researched sources only - given files are user-provided ground
truth. Everything after intake is identical regardless of where the material
came from.

Triggers: "make an explainer", "turn this paper into an interactive article",
"build a distill-style essay", "explain X visually", and similar.

Skill type: technique/workflow, with one hard gate (the fact-check before
delivery).

### 2. explaining-codebases

Adds what is genuinely different about explaining code, then defers to
`creating-explainers` for everything about the output.

What it adds:

- `code-intake.md`: navigate the repository, decide the angle (whole-system
  architecture overview vs single-mechanism deep-dive - the skill supports
  both), identify the spine concept, map the architecture, and pull *real* code
  snippets to anchor the explanation.
- `code-figure-archetypes.md`: figure patterns specific to code - architecture
  and module diagrams, data-flow and sequence diagrams, execution-trace steppers
  that walk an algorithm, and annotated code walkthroughs.

What it reuses (by cross-reference, not duplication): the HTML template, voice
and style, base figure archetypes, color palettes, the outline/draft/polish
workflow, and the quality checklist from `creating-explainers`.

Triggers: "explain this codebase", "interactive guide to this repo", "walk
through how X works in the code", "onboarding explainer for this project".

Skill type: technique/workflow that extends `creating-explainers`.

### 3. fact-checking-explainers (discipline-enforcing gate)

Guarantees the hard requirement: no incorrect factual claim ships. This is a
discipline-enforcing skill, not advisory.

**Iron Law:** No explainer ships with an unverified factual claim. Every
checkable claim either traces to a source that supports it, gets corrected to
match the source, or gets cut.

**What counts as a checkable claim:** specific factual or empirical assertions -
numbers, dates, names, attributions, quotes, historical events, performance
results, and mechanism claims ("X uses/causes/does Y"). Not every sentence:
clearly-labeled interpretation, analogies, and the author's framing are not
claims to verify. Heuristic: if unsure whether something is a checkable claim,
treat it as one and verify it.

**Verdicts per claim:** supported / needs-source (plausible but uncited) /
unsupported (no source backs it) / contradicted (a source says otherwise).

**Resolution (at delivery every checkable claim must be `supported`; none may
remain needs-source, unsupported, or contradicted):** add the supporting
citation, correct it to match the source, soften it to clearly-labeled
interpretation if it is genuinely interpretive, or cut it.

**Two integration points (the "both phases" requirement):**

- Research-time gate, inside `creating-explainers` research intake: before
  drafting, confirm the gathered sources actually exist and support the points
  they will be used for. Catches fabricated or misremembered sources early.
- Post-draft gate, for every explainer: before delivery, audit every checkable
  claim in the finished article against its cited source. For codebase
  explainers, the source of truth is the real code - every "this function does
  X" claim is checked against the actual implementation.

**Enforcement in the other skills:** `creating-explainers` and
`explaining-codebases` list the post-draft fact-check as a REQUIRED, blocking
item in their quality checklist. The explainer is not "done" and is not
delivered until the fact-check returns zero unresolved claims - that is, every
checkable claim is `supported`.

**Honest boundary:** the gate guarantees that every *checkable* claim is
traceable to a supporting source or removed, using verification-against-source
plus an adversarial default-to-unsupported-when-uncertain stance. It does not
claim omniscience about the world; it makes "no incorrect claim" enforceable by
tying every claim to evidence.

**Output:** a claim-by-claim report (claim, location in the article, verdict,
source/evidence, severity, recommended fix) plus an overall pass/fail gate
verdict. Format defined in `verification-report-format.md`.

Triggers: invoked by the other two skills at the two gates above, and directly
on "fact-check this explainer", "verify the claims", "check this article against
its sources".

Skill type: discipline-enforcing (Iron Law, no-exceptions list, rationalization
table, red flags).

## Reuse mapping (from interactive-explainer-skill-archive)

| Prior artifact | New location | Change |
|---|---|---|
| skills/explain-it/SKILL.md | creating-explainers/SKILL.md | Restructured: core workflow only; intake split into two refs; add fact-check gate |
| references/document-mode.md | creating-explainers/references/intake-from-files.md | Light edit; add cross-pointer to intake-from-research for mixed (files + research) cases |
| references/research-mode.md | creating-explainers/references/intake-from-research.md | Light edit; replace the soft "hallucinated facts" warning with a handoff to the research-time fact-check gate; add cross-pointer to intake-from-files for mixed cases |
| references/figure-archetypes.md | creating-explainers/references/figure-archetypes.md | As-is |
| references/voice-and-style.md | creating-explainers/references/voice-and-style.md | As-is |
| references/template-walkthrough.md | creating-explainers/references/template-walkthrough.md | As-is |
| references/color-palettes.md | creating-explainers/references/color-palettes.md | As-is |
| assets/article-template.html | creating-explainers/assets/article-template.html | As-is |
| evals/evals.json | evals/evals.json | Extend with 2 new evals |
| .claude-plugin/plugin.json | .claude-plugin/plugin.json | Rename to explain-this, update |
| .claude-plugin/marketplace.json | .claude-plugin/marketplace.json | Update plugin + skills list |
| README.md | README.md | Rewrite for 3-skill structure |
| CLAUDE.md | CLAUDE.md | Update for 3-skill structure |

New files (no prior art):

- explaining-codebases/SKILL.md
- explaining-codebases/references/code-intake.md
- explaining-codebases/references/code-figure-archetypes.md
- fact-checking-explainers/SKILL.md
- fact-checking-explainers/references/verification-report-format.md

Before copying any reused file, read it directly to confirm its contents match
the inventory above; edit only where this spec says to.

## Cross-referencing convention

Skills reference each other by name with explicit requirement markers, never with
`@`-includes (which force-load and burn context eagerly). Examples:

- In `explaining-codebases`: "REQUIRED: use `creating-explainers` for the output
  format, template, voice, and base figure archetypes. This skill only adds code
  intake and code-specific figures."
- In `creating-explainers` and `explaining-codebases`: "REQUIRED before delivery:
  run `fact-checking-explainers`. The explainer is not done until it passes."

Within a single skill, reference files are loaded lazily (read only when that
phase is reached), matching the prior work's progressive-disclosure design.

## Evals (light)

Keep `evals.json` as a set of reference prompts (not an automated harness). Reuse
the prior three:

- topic-mode RLHF (research intake)
- document-mode Transformers excerpt (files intake)
- topic-mode PageRank (research intake, skill picks the angle)

Add two:

- A codebase explainer eval (point the skill at a small repo or set of files and
  ask for an interactive guide).
- A fact-check eval: a short drafted explainer with a few planted factual errors;
  the skill must catch and flag them with correct verdicts.

## Out of scope (YAGNI)

- No bootstrap/orchestrator skill (the three descriptions route on their own).
- No automated pressure-test or subagent eval harness.
- No publish/deploy/hosting step.
- No multi-article index or collection management.
- No change to the single-file vanilla HTML output format.

## Naming decisions

- Plugin name: `explain-this` (matches the directory).
- Fact-checker skill: `fact-checking-explainers` (scoped, so it will not fire on
  unrelated fact-check requests).
- All skill names use gerund/active form per superpowers conventions:
  `creating-explainers`, `explaining-codebases`, `fact-checking-explainers`.

## Build and commit plan

The user asked for relevant commits as the work proceeds. Rough cadence (the
implementation plan will finalize task granularity):

1. Repo init + this design spec + LICENSE + .gitignore.
2. Port reused core into `creating-explainers/references` and `assets`.
3. `creating-explainers/SKILL.md` (restructured workflow + fact-check gate) and
   the two intake references.
4. `fact-checking-explainers` skill + report-format reference.
5. `explaining-codebases` skill + its two references.
6. Plugin manifests, README, CLAUDE.md, and extended evals.

Each step is a focused commit. Commit messages follow the repo convention.
