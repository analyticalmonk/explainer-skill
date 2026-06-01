# explain-this Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the `explain-this` Claude Code plugin: three skills that produce distill-style, single-file, zero-dependency interactive HTML explainers from documents, web research, or a codebase, with a fact-checking gate so no incorrect claim ships.

**Architecture:** A hub skill, `creating-explainers`, owns the output format (HTML template, figure archetypes, voice, color palettes) and the `intake -> outline -> scaffold -> prose -> figures -> fact-check -> polish` workflow; it handles file and web-research intake (and the mix of both) inline. `explaining-codebases` adds code navigation and code-specific figures, deferring to the hub for everything about the output. `fact-checking-explainers` is a discipline-enforcing gate the other two must pass before delivery. Most reference material is ported as-is from the prior `explain-it` plugin.

**Tech Stack:** Markdown skills (SKILL.md + `references/` + `assets/`), a `.claude-plugin/` manifest pair, vanilla HTML/CSS/JS article template (no build step), JSON evals. Validation uses shell checks (`grep`, `diff`, `python3 -m json.tool`, `python3 -m http.server`).

**Source/destination shorthand used below:**
- `ARCHIVE` = `/home/analyticalmonk/open_source/interactive-explainer-skill-archive`
- `DEST` = `/home/analyticalmonk/open_source/explain-this` (the repo root, already git-initialized, branch `main`)

**Process note:** This is a documentation/skill-authoring project, not code-with-unit-tests. Each task follows **author -> validate -> commit**. "Validate" means concrete structural checks (frontmatter parses, no em-dashes, cross-references resolve, JSON valid, copied files byte-identical to source) plus, for the fact-checker, a smoke-test against a planted-error eval. There is no red-green-refactor cycle because there is no runtime to test.

**Global rules (apply to every task):**
- No em-dashes (`—`) anywhere, in any file. Use hyphens or rewrite.
- Cross-reference other skills by name (e.g. `fact-checking-explainers`), never with `@`-includes.
- Reference files load lazily: do not inline reference content into a `SKILL.md`, and do not cross-link reference files in ways that force-load extras.
- Commit messages end with the repo convention footer:
  `Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>`

---

## File Structure

Created by this plan (all under `DEST`):

```
.claude-plugin/
  plugin.json                      # plugin manifest (name: explain-this)
  marketplace.json                 # marketplace entry listing 3 skills
skills/
  creating-explainers/
    SKILL.md                       # hub: output format + workflow + gate wiring
    assets/
      article-template.html        # copied as-is from ARCHIVE
    references/
      intake-from-files.md         # ported from document-mode.md + mixed-intake pointer
      intake-from-research.md      # ported from research-mode.md + fact-check handoff + pointer
      figure-archetypes.md         # copied as-is
      voice-and-style.md           # copied as-is
      template-walkthrough.md      # copied as-is
      color-palettes.md            # copied as-is
  explaining-codebases/
    SKILL.md                       # code intake + code figures; defers to hub
    references/
      code-intake.md               # navigate repo, pick angle, map architecture, pull real snippets
      code-figure-archetypes.md    # architecture/data-flow/execution-trace/annotated-walkthrough figures
  fact-checking-explainers/
    SKILL.md                       # discipline-enforcing verification gate
    references/
      verification-report-format.md# claim-by-claim report format + pass/fail rule
evals/
  evals.json                       # 3 prior evals + 2 new (codebase, fact-check)
README.md                          # human-facing install/usage for 3 skills
CLAUDE.md                          # contributor guidance for 3-skill repo
LICENSE                            # MIT, copied from ARCHIVE
.gitignore                         # copied from ARCHIVE
docs/                              # already exists (spec + this plan)
```

Already present: `docs/specs/2026-06-01-explain-this-plugin-design.md`, `docs/plans/2026-06-01-explain-this-plugin.md`.

---

## Task 1: Repo scaffolding (LICENSE, .gitignore)

**Files:**
- Create: `DEST/LICENSE` (copy of `ARCHIVE/LICENSE`)
- Create: `DEST/.gitignore` (copy of `ARCHIVE/.gitignore`)

- [ ] **Step 1: Copy LICENSE and .gitignore**

```bash
cd /home/analyticalmonk/open_source/explain-this
cp /home/analyticalmonk/open_source/interactive-explainer-skill-archive/LICENSE LICENSE
cp /home/analyticalmonk/open_source/interactive-explainer-skill-archive/.gitignore .gitignore
```

- [ ] **Step 2: Validate the copies are byte-identical**

```bash
diff /home/analyticalmonk/open_source/interactive-explainer-skill-archive/LICENSE LICENSE && echo "LICENSE OK"
diff /home/analyticalmonk/open_source/interactive-explainer-skill-archive/.gitignore .gitignore && echo "gitignore OK"
```
Expected: both print `OK` with no diff output. (If the MIT LICENSE names a year/author you want updated, that is fine to leave as-is; it is already attributed to analyticalmonk.)

- [ ] **Step 3: Commit**

```bash
git add LICENSE .gitignore
git commit -m "chore: add LICENSE and gitignore

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>"
```

---

## Task 2: Port the as-is references and template into creating-explainers

These six files (four references + one template) are reused verbatim. The prior work is mature; do not rewrite them.

**Files:**
- Create: `DEST/skills/creating-explainers/references/figure-archetypes.md`
- Create: `DEST/skills/creating-explainers/references/voice-and-style.md`
- Create: `DEST/skills/creating-explainers/references/template-walkthrough.md`
- Create: `DEST/skills/creating-explainers/references/color-palettes.md`
- Create: `DEST/skills/creating-explainers/assets/article-template.html`

- [ ] **Step 1: Create directories and copy the four as-is references**

```bash
cd /home/analyticalmonk/open_source/explain-this
mkdir -p skills/creating-explainers/references skills/creating-explainers/assets
SRC=/home/analyticalmonk/open_source/interactive-explainer-skill-archive/skills/explain-it
cp "$SRC/references/figure-archetypes.md"    skills/creating-explainers/references/
cp "$SRC/references/voice-and-style.md"       skills/creating-explainers/references/
cp "$SRC/references/template-walkthrough.md"  skills/creating-explainers/references/
cp "$SRC/references/color-palettes.md"        skills/creating-explainers/references/
cp "$SRC/assets/article-template.html"        skills/creating-explainers/assets/
```

- [ ] **Step 2: Validate copies are byte-identical and contain no em-dashes**

```bash
SRC=/home/analyticalmonk/open_source/interactive-explainer-skill-archive/skills/explain-it
for f in references/figure-archetypes.md references/voice-and-style.md references/template-walkthrough.md references/color-palettes.md assets/article-template.html; do
  diff "$SRC/$f" "skills/creating-explainers/$f" && echo "OK $f"
done
grep -rn '—' skills/creating-explainers/ && echo "FOUND EM-DASH (investigate)" || echo "no em-dashes"
```
Expected: five `OK` lines, and `no em-dashes`. (The source files are already clean; this confirms the copy did not corrupt anything.)

- [ ] **Step 3: Validate the template still renders**

```bash
cd skills/creating-explainers/assets
python3 -m http.server 8011 >/dev/null 2>&1 &
SRV=$!; sleep 1
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8011/article-template.html
kill $SRV
cd /home/analyticalmonk/open_source/explain-this
```
Expected: `200`. If browser tooling is available, open the page and confirm no console errors and that the placeholder figure canvases initialize. The template is known-good from the prior project, so this is a smoke check only.

- [ ] **Step 4: Commit**

```bash
git add skills/creating-explainers/references skills/creating-explainers/assets
git commit -m "feat: port reused explainer references and HTML template

figure-archetypes, voice-and-style, template-walkthrough, color-palettes, and
the article-template.html skeleton, copied as-is from the explain-it prototype.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>"
```

---

## Task 3: Create the two intake references (ported + edited)

Port `document-mode.md` -> `intake-from-files.md` and `research-mode.md` -> `intake-from-research.md`, then apply the spec's edits: make them composable (mixed intake) and replace the soft hallucination warning with a fact-check handoff.

**Files:**
- Create: `DEST/skills/creating-explainers/references/intake-from-files.md`
- Create: `DEST/skills/creating-explainers/references/intake-from-research.md`

- [ ] **Step 1: Copy the two source files to their new names**

```bash
cd /home/analyticalmonk/open_source/explain-this
SRC=/home/analyticalmonk/open_source/interactive-explainer-skill-archive/skills/explain-it/references
cp "$SRC/document-mode.md" skills/creating-explainers/references/intake-from-files.md
cp "$SRC/research-mode.md" skills/creating-explainers/references/intake-from-research.md
```

- [ ] **Step 2: Edit `intake-from-files.md` - retitle and add the mixed-intake pointer**

Change the H1 `# Document Mode` to `# Intake From Files`.

Append this section at the end of the file (after the existing "## Multiple Sources" section):

```markdown
## Combining With Research (Mixed Intake)

The given files are not always enough. If the topic needs material the files do
not cover - recent developments, criticisms, corroboration, or context the
source omits - supplement with web research. Read `intake-from-research.md` and
combine the two: treat the given files as the spine (the narrative the article
tracks) and the researched sources as supporting. Synthesize across everything
into a single outline. Run the research-time fact-check on the researched
sources only; the given files are user-provided ground truth.
```

- [ ] **Step 3: Edit `intake-from-research.md` - retitle, add mixed-intake pointer, replace the hallucination warning with a fact-check handoff**

Change the H1 `# Research Mode` to `# Intake From Research`.

In "## Step 3: Read and synthesize", append this paragraph at the end of the section (after the existing "Where do they agree? Where do they disagree?" text):

```markdown
Before moving to the outline, run the research-time fact-check (see
`fact-checking-explainers`): confirm each gathered source actually exists and
supports the specific points you plan to build on. Drop or replace any source
that does not hold up. Verifying sources now is cheaper than discovering at the
post-draft gate that a load-bearing claim has no support.
```

Replace the existing "Hallucinated facts" pitfall bullet:

```markdown
- **Hallucinated facts.** The biggest risk. Anything specific (numbers, dates, names, quotes) must come from a source you actually read. If you're tempted to write "the model achieved 94.3% accuracy", check that 94.3% appears in your notes from the source.
```

with:

```markdown
- **Unverified facts.** The biggest risk, and the reason fact-checking is a hard gate, not advice. Anything specific (numbers, dates, names, quotes) must come from a source you actually read. This is enforced at two points by `fact-checking-explainers`: a research-time pass over your gathered sources before drafting, and a post-draft pass that audits every claim before delivery. The explainer is not done until it passes.
```

Add a mixed-intake pointer. After the existing "## When the user has a draft idea but wants you to fill it in" section, append:

```markdown
## When the User Also Provided Files

If the user handed over source documents *and* wants research, read
`intake-from-files.md` and combine via spine-vs-supporting: the given files are
usually the spine, the researched sources are supporting. One outline, one
article. The research-time fact-check applies to the researched sources.
```

- [ ] **Step 4: Validate**

```bash
cd /home/analyticalmonk/open_source/explain-this
# H1s updated
head -1 skills/creating-explainers/references/intake-from-files.md     # -> # Intake From Files
head -1 skills/creating-explainers/references/intake-from-research.md  # -> # Intake From Research
# the soft warning is gone, the handoff is present
grep -c "Hallucinated facts" skills/creating-explainers/references/intake-from-research.md   # -> 0
grep -c "fact-checking-explainers" skills/creating-explainers/references/intake-from-research.md  # -> >=2
grep -c "intake-from-research" skills/creating-explainers/references/intake-from-files.md     # -> >=1
# no em-dashes
grep -rn '—' skills/creating-explainers/references/intake-from-files.md skills/creating-explainers/references/intake-from-research.md && echo "EM-DASH" || echo "clean"
```
Expected: the two H1 lines, `0`, a count >= 2, a count >= 1, and `clean`.

- [ ] **Step 5: Commit**

```bash
git add skills/creating-explainers/references/intake-from-files.md skills/creating-explainers/references/intake-from-research.md
git commit -m "feat: add composable file and research intake references

Ported from document-mode/research-mode. Intakes now compose (mixed intake via
spine-vs-supporting), and the soft hallucination warning is replaced by a hard
handoff to fact-checking-explainers at research-time and post-draft.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>"
```

---

## Task 4: Write `creating-explainers/SKILL.md` (restructured hub)

Start from the prior `explain-it` SKILL.md (`ARCHIVE/skills/explain-it/SKILL.md`, already read) and produce a restructured version. Most sections (Overview, The Article Skeleton, Figures, Voice, Common Mistakes) carry over nearly unchanged. The specific changes are listed below; everything not mentioned is reused verbatim from the prior SKILL.md.

**Files:**
- Create: `DEST/skills/creating-explainers/SKILL.md`

- [ ] **Step 1: Write the frontmatter (verbatim)**

```yaml
---
name: creating-explainers
description: Use when creating an interactive educational article, explainer, or distill-style essay - a single self-contained HTML page with hand-built canvas figures the reader can interact with. Covers articles built from user-provided files (paper, blog post, transcript, research report), topic-driven articles needing web research, and the mix of both. Trigger phrases include "make an explainer", "turn this paper into an interactive article", "build a distill-style essay", "explain X visually", or any request to produce an interactive HTML walkthrough with hand-rendered canvas figures. For explaining a codebase or source files, use explaining-codebases instead. Use even when the user describes the goal without naming the format - if they want an interactive, narrative article with figures the reader can play with, this skill applies.
---
```

- [ ] **Step 2: Write the body**

Title: `# Creating Explainers`.

Reuse verbatim from the prior SKILL.md these sections (they are correct as-is): **Overview**, **When to Use** (but add one "Don't use for" bullet: `- Explaining a codebase or source files - use explaining-codebases`), **The Article Skeleton**, **Figures**, **Voice**, **Common Mistakes**.

Replace the prior "## The Two Input Modes" section with this **## Intake: Gather Your Source Material**:

```markdown
## Intake: Gather Your Source Material

Intake is one phase: gather what the article is built from. The material can be
given files, web research, or both. Pick the references that apply and read them
when you reach this phase, not upfront.

| You have | Read | Source of truth |
|----------|------|-----------------|
| Files (paper, URL, paste, transcript, report) | `references/intake-from-files.md` | The provided document(s) |
| Only a topic | `references/intake-from-research.md` | Web research, then synthesis |
| Files **and** a topic that needs more | both | Files as spine, research as supporting |

**Mixed intake.** When the user provides files and the topic also needs material
the files do not cover (recent developments, criticisms, corroboration), read
both references and combine them via spine-vs-supporting. Synthesize across
everything into one outline and one article.

If you only have a topic and web search is unavailable, ask the user to paste
sources. Do not write a technical explainer from training data alone.
```

Replace the prior "## Workflow" diagram block with this one (adds the two fact-check gates):

```markdown
## Workflow

The article is too rich to one-shot. Work in stages and check in with the user.

```
1. Intake            -> gather material (files, research, or both)
2. Research-time fact-check  -> if you researched, verify sources before drafting (REQUIRED)
3. Outline           -> propose sections + figure list, get user approval
4. Scaffold          -> copy template, fill metadata
5. Prose pass        -> write all sections with figure placeholders
6. Figures pass      -> implement each interactive figure
7. Post-draft fact-check  -> audit every claim against its source (REQUIRED, blocking)
8. Polish            -> captions, cross-links, mobile check
```

**Pause for user approval after the outline (step 3).** This is the
highest-leverage check-in.

**Steps 2 and 7 are gates, not suggestions.** Use `fact-checking-explainers`.
The article is not done until the post-draft fact-check returns zero unresolved
claims - every checkable claim is supported. See the Quality Checklist.
```

In the "## Quality Checklist", add this as the FIRST checklist item (before "index.html is fully self-contained"):

```markdown
- [ ] **Fact-check passed (blocking).** Ran `fact-checking-explainers` on the finished article; every checkable claim is supported by its cited source (or the real code), with zero unresolved unsupported or contradicted claims. This gate must pass before delivery.
```

Replace the "## Reference Files" list with this updated one:

```markdown
## Reference Files

Read these as needed during the workflow - don't load them all upfront.

- `assets/article-template.html` - the starting skeleton, copy this into the new article directory
- `references/template-walkthrough.md` - what every block in the template does
- `references/figure-archetypes.md` - code patterns for the common figure types
- `references/voice-and-style.md` - writing patterns, paragraph examples, callout/epigraph usage
- `references/color-palettes.md` - accent color sets per article, how to pick a palette
- `references/intake-from-files.md` - extracting structure from a provided paper, blog post, or transcript
- `references/intake-from-research.md` - web research workflow when only a topic is given

Related skills:
- `fact-checking-explainers` - REQUIRED gate at research-time and before delivery
- `explaining-codebases` - use instead of this skill when the subject is a codebase or source files
```

- [ ] **Step 3: Validate**

```bash
cd /home/analyticalmonk/open_source/explain-this
F=skills/creating-explainers/SKILL.md
# frontmatter present, name matches directory
python3 - "$F" <<'PY'
import sys,re
t=open(sys.argv[1]).read()
m=re.match(r'^---\n(.*?)\n---\n',t,re.S); assert m,"no frontmatter"
fm=m.group(1)
assert re.search(r'^name:\s*creating-explainers\s*$',fm,re.M),"name wrong"
assert 'description:' in fm,"no description"
print("frontmatter OK")
PY
# gate wiring present
grep -c "fact-checking-explainers" "$F"   # -> >=2
grep -c "explaining-codebases" "$F"       # -> >=1
grep -ci "post-draft fact-check" "$F"     # -> >=1
# no em-dashes, no @-includes of other skills
grep -n '—' "$F" && echo "EM-DASH" || echo "clean"
grep -n '@skills/' "$F" && echo "BAD @-include" || echo "no @-includes"
```
Expected: `frontmatter OK`, counts >= as noted, `clean`, `no @-includes`.

- [ ] **Step 4: Commit**

```bash
git add skills/creating-explainers/SKILL.md
git commit -m "feat: add creating-explainers hub skill

Restructured from explain-it: composable intake section, fact-check gates wired
into the workflow and quality checklist, and routing to explaining-codebases.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>"
```

---

## Task 5: Write the `fact-checking-explainers` skill

A discipline-enforcing skill (the load-bearing new piece). It guarantees the hard requirement: no incorrect claim ships.

**Files:**
- Create: `DEST/skills/fact-checking-explainers/SKILL.md`
- Create: `DEST/skills/fact-checking-explainers/references/verification-report-format.md`

- [ ] **Step 1: Write `SKILL.md` frontmatter (verbatim)**

```yaml
---
name: fact-checking-explainers
description: Use before delivering any explainer, and whenever asked to fact-check, verify, or audit an explainer or article against its sources. Trigger phrases include "fact-check this explainer", "verify the claims", "check this article against its sources", "is this accurate", or reaching the end of researching or drafting an explainer. Applies both at research-time (are the gathered sources real and on-point?) and post-draft (does every claim trace to its cited source or, for code, the actual implementation?). Invoked directly or as a required gate by creating-explainers and explaining-codebases.
---
```

- [ ] **Step 2: Write `SKILL.md` body to this section spec**

Title: `# Fact-Checking Explainers`. Target ~150-220 lines. Required sections, in order:

1. **## Overview** - one principle: an explainer's authority comes from being right; a single confident wrong claim discredits the whole piece. This skill makes "no incorrect claim ships" enforceable by tying every checkable claim to evidence.

2. **## The Iron Law** - in a fenced block, verbatim:
   ```
   NO EXPLAINER SHIPS WITH AN UNVERIFIED FACTUAL CLAIM.
   ```
   Followed by: every checkable claim either traces to a source that supports it, gets corrected to match the source, or gets cut. Then a **No exceptions** list:
   - Not for "it's obviously true" - obvious things are wrong often enough; verify.
   - Not for "I'm pretty sure" - pretty sure is the unsupported verdict; find the source or cut it.
   - Not for "it's just background" - background facts are still facts and still wrong sometimes.
   - Not for "the draft is due" - a wrong explainer is worse than a late one.
   - "Verify" means against an actual source you can point to, not memory.

3. **## When to Use** - the two gates (research-time, post-draft) and direct invocation. **When NOT to use:** non-explainer content, or pure opinion/editorial pieces with no factual claims.

4. **## What Counts as a Checkable Claim** - reuse the spec's definition: specific factual/empirical assertions (numbers, dates, names, attributions, quotes, historical events, performance results, mechanism claims "X uses/causes/does Y"). Not checkable: clearly-labeled interpretation, analogies, the author's framing. Heuristic, stated explicitly: **if unsure whether something is a checkable claim, treat it as one.**

5. **## The Verification Process** - numbered:
   1. Extract every checkable claim, with its location in the article.
   2. For each, find the supporting passage in the cited source (or, for codebase explainers, the exact code at `file:line`). Read the source; do not trust the citation's existence as support.
   3. Assign a verdict (table below).
   4. Resolve everything that is not `supported`.
   Include the **verdict table** verbatim:
   ```markdown
   | Verdict | Meaning | Allowed at delivery? |
   |---------|---------|----------------------|
   | supported | A source passage directly backs the claim | Yes |
   | needs-source | Plausible but no citation yet | No - add a source or downgrade |
   | unsupported | No source backs it | No - source it, soften, or cut |
   | contradicted | A source says otherwise | No - correct it or cut it |
   ```

6. **## The Two Gates** -
   - Research-time (inside `creating-explainers` research intake): before drafting, confirm each gathered source exists and supports the points it will be used for. Catch fabricated or misremembered sources here.
   - Post-draft (every explainer, before delivery): audit every checkable claim against its cited source. **For codebase explainers, the source of truth is the real code** - every "this function does X" claim is checked against the actual implementation at a specific path and line.

7. **## Resolution** - at delivery every checkable claim must be `supported`. Four options: add the supporting citation; correct the claim to match the source; soften to clearly-labeled interpretation if genuinely interpretive; or cut it. State plainly: shipping with an unresolved `needs-source`, `unsupported`, or `contradicted` claim violates the Iron Law.

8. **## The Report** - produce a claim-by-claim report plus an overall PASS/FAIL verdict. Point to `references/verification-report-format.md` for the exact format. FAIL until every claim is `supported`.

9. **## Rationalization Table** - verbatim:
   ```markdown
   | Excuse | Reality |
   |--------|---------|
   | "I wrote it, so I know it's right" | You know what you intended. Verify what you wrote. |
   | "It's a well-known fact" | Well-known facts are wrong often enough to check. 30 seconds. |
   | "The source probably says this" | Probably is not a verdict. Open the source and confirm. |
   | "Close enough" | Numbers, dates, and names are exact or they are wrong. |
   | "It's only one claim" | One confident wrong claim discredits the whole explainer. |
   | "I'll flag it and let the reader decide" | The gate is your job, not the reader's. |
   ```

10. **## Red Flags** - thoughts that mean stop and verify: "I'm fairly sure...", "If I recall...", "roughly...", "it's basically...", pasting a statistic without re-reading its source, citing a URL you did not open.

11. **## Common Mistakes** - verifying the citation exists instead of reading whether it supports the claim; checking the easy claims and waving through the hard ones; for code, trusting a comment or a name instead of the implementation; treating "needs-source" as shippable.

- [ ] **Step 3: Write `references/verification-report-format.md` to this spec**

Target ~60-100 lines. Contents:
- The report is a markdown table, one row per checkable claim, columns: `#`, `Claim (quoted)`, `Location` (section/figure or `file:line` for code), `Verdict`, `Source / evidence` (URL + quoted passage, or `file:line` + quoted code), `Severity` (high = numeric/named/mechanism error a reader would act on; low = minor imprecision), `Recommended fix`.
- Below the table: an **overall verdict** line, `PASS` only if every row is `supported`, else `FAIL` with the count of unresolved claims by verdict.
- A short **worked example** table with 3 rows: one `supported`, one `contradicted` (with the corrected value), one `unsupported` (recommend cut or source).
- A note: when invoked as a gate inside another skill, return the report inline and do not deliver the article until PASS. When invoked directly by a user, present the report and offer to apply the fixes.

- [ ] **Step 4: Validate**

```bash
cd /home/analyticalmonk/open_source/explain-this
F=skills/fact-checking-explainers/SKILL.md
python3 - "$F" <<'PY'
import sys,re
t=open(sys.argv[1]).read()
m=re.match(r'^---\n(.*?)\n---\n',t,re.S); assert m,"no frontmatter"
assert re.search(r'^name:\s*fact-checking-explainers\s*$',m.group(1),re.M),"name wrong"
print("frontmatter OK")
PY
grep -c "NO EXPLAINER SHIPS WITH AN UNVERIFIED FACTUAL CLAIM" "$F"   # -> 1
grep -ci "contradicted" "$F"   # -> >=1
test -f skills/fact-checking-explainers/references/verification-report-format.md && echo "report ref exists"
grep -rn '—' skills/fact-checking-explainers/ && echo "EM-DASH" || echo "clean"
```
Expected: `frontmatter OK`, `1`, a count >= 1, `report ref exists`, `clean`.

- [ ] **Step 5: Commit**

```bash
git add skills/fact-checking-explainers
git commit -m "feat: add fact-checking-explainers gate skill

Discipline-enforcing verification: Iron Law, checkable-claim definition, verdict
table, two gates (research-time and post-draft, with code as source of truth for
codebase explainers), rationalization table, and the claim-by-claim report format.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>"
```

---

## Task 6: Write the `explaining-codebases` skill

Adds code intake and code-specific figures; defers to `creating-explainers` for the output.

**Files:**
- Create: `DEST/skills/explaining-codebases/SKILL.md`
- Create: `DEST/skills/explaining-codebases/references/code-intake.md`
- Create: `DEST/skills/explaining-codebases/references/code-figure-archetypes.md`

- [ ] **Step 1: Write `SKILL.md` frontmatter (verbatim)**

```yaml
---
name: explaining-codebases
description: Use when creating an interactive explainer about a codebase, repository, or set of source files - an onboarding guide to how a project is structured, or a deep-dive on how a specific algorithm or mechanism is implemented in real code. Trigger phrases include "explain this codebase", "interactive guide to this repo", "walk through how X works in the code", "onboarding explainer for this project", "visualize this architecture". Produces the same single self-contained HTML explainer as creating-explainers, but with code navigation and code-specific figures (architecture diagrams, data-flow, execution traces, annotated code walkthroughs). For a paper, topic, or non-code source, use creating-explainers instead.
---
```

- [ ] **Step 2: Write `SKILL.md` body to this section spec**

Title: `# Explaining Codebases`. Target ~120-180 lines. Required sections:

1. **## Overview** - this skill explains code as it actually exists. The output is the same distill-style single-file HTML explainer; what differs is intake (navigating real code) and the figure types (architecture, data-flow, execution traces, annotated walkthroughs).

2. **## Required Background** - verbatim marker:
   ```markdown
   **REQUIRED:** Use `creating-explainers` for everything about the output - the
   HTML template, voice and style, base figure archetypes, color palettes, the
   outline/scaffold/prose/polish workflow, and the quality checklist. This skill
   only adds what is specific to code. Do not duplicate that material here.
   ```

3. **## When to Use** - both supported angles: whole-system architecture/onboarding overview, and single-mechanism deep-dive. The skill picks the angle from the request (and confirms with the user). **When NOT to use:** a paper, topic, or non-code source -> `creating-explainers`.

4. **## Code Intake** - point to `references/code-intake.md`. Summarize: pick the angle, navigate the repo, find the spine concept, map the architecture, and pull real code snippets anchored to `file:line`.

5. **## Code-Specific Figures** - point to `references/code-figure-archetypes.md`. List the patterns: architecture/module diagram, data-flow/sequence, execution-trace stepper, annotated code walkthrough. Note that mechanics (initCanvas, IIFE pattern, the base archetypes) come from `creating-explainers/references/figure-archetypes.md`.

6. **## The Fact-Check Gate for Code** - verbatim marker:
   ```markdown
   **REQUIRED before delivery:** run `fact-checking-explainers`. For a codebase
   explainer the source of truth is the code itself. Every claim about what the
   code does, and every quoted snippet, is checked against the actual
   implementation at a specific path and line. Code drifts; a snippet that was
   accurate yesterday may be wrong today. The explainer is not done until it passes.
   ```

7. **## Workflow** - same as `creating-explainers`, with code intake substituted for steps 1-2. Reproduce the 8-step workflow but with step 1 = "Code intake (navigate, pick angle, map architecture, pull snippets)" and step 7 = "Post-draft fact-check against the real code (REQUIRED, blocking)".

8. **## Common Mistakes** - table: explaining code that does not exist or that you imagined; snippets that drift from the real source (always quote, never paraphrase as if quoting); architecture diagrams that do not match the actual module structure; scope too broad (a whole framework in one article) - pick one subsystem or one mechanism.

- [ ] **Step 3: Write `references/code-intake.md` to this spec**

Target ~90-140 lines. Numbered workflow:
1. **Pick the angle (with the user).** Overview vs mechanism. Overview = how the pieces fit; mechanism = how one thing works end to end. State the one-sentence throughline (same discipline as `intake-from-files.md`).
2. **Navigate.** Find entry points (main, server bootstrap, CLI, exported API), the build/config files, and the top-level module structure. Tools: directory listing, `grep`/search for definitions and call sites, read key files. For a large repo, use a read-only exploration pass before deciding scope.
3. **Find the spine.** The single path or concept the article tracks (e.g. "an HTTP request from socket to handler", "how the scheduler picks the next task").
4. **Map the architecture.** Modules, their responsibilities, the data that flows between them, and the key dependencies. This becomes the architecture figure.
5. **Pull real snippets.** Quote actual code, short (5-20 lines), each tagged with its `path:line`. Trim with `...` but never invent or "clean up" code into something the repo does not contain.
6. **Outline and get approval** (same format and pause as the file/research intakes), with the snippet list and figure plan.
Pitfalls: snippets drifting from the real code; explaining generated/vendored code as if hand-written; diagrams that assert a structure the code does not have; over-broad scope.
Add a closing line: every claim and snippet produced here is later checked by `fact-checking-explainers` against the code, so capture exact paths and lines as you go.

- [ ] **Step 4: Write `references/code-figure-archetypes.md` to this spec**

Target ~120-180 lines. For each pattern: what it shows, when to use it, which base archetype it builds on, and interaction. Patterns:
- **Architecture / module diagram** - boxes for modules, arrows for dependencies/calls; clicking a module highlights its connections. Include one short worked canvas IIFE sketch (boxes laid out on a grid, arrows drawn between them, a click handler that highlights neighbors) using `initCanvas`. This is the most code-specific pattern, so it gets the worked example.
- **Data-flow / sequence diagram** - an item (request, message, token) animated through stages; builds on the continuous-animation archetype. Describe; reference the base archetype for the loop mechanics.
- **Execution-trace stepper** - step through an algorithm: show state plus the current line/operation highlighted; builds on the stepped-simulation archetype. Describe the state-per-step approach and pairing the canvas with a code panel where the active line is highlighted.
- **Annotated code walkthrough** - a code block where stepping or hovering reveals annotations on specific lines. Describe markup (lines with data attributes) plus a small controller; may not need canvas.
- **Dependency graph** - nodes and edges for module/package dependencies; builds on continuous-animation or a static layout. Describe briefly.
Open with a one-line note: mechanics (DPR-aware `initCanvas`, the IIFE pattern, requestAnimationFrame loops) come from `creating-explainers/references/figure-archetypes.md`; this file only covers how to apply them to code. Close with the rule from `creating-explainers`: every figure must illustrate something the prose just set up - no decorative diagrams.

- [ ] **Step 5: Validate**

```bash
cd /home/analyticalmonk/open_source/explain-this
F=skills/explaining-codebases/SKILL.md
python3 - "$F" <<'PY'
import sys,re
t=open(sys.argv[1]).read()
m=re.match(r'^---\n(.*?)\n---\n',t,re.S); assert m,"no frontmatter"
assert re.search(r'^name:\s*explaining-codebases\s*$',m.group(1),re.M),"name wrong"
print("frontmatter OK")
PY
grep -c "creating-explainers" "$F"        # -> >=2 (required background + when-not)
grep -c "fact-checking-explainers" "$F"   # -> >=1
test -f skills/explaining-codebases/references/code-intake.md && echo "code-intake exists"
test -f skills/explaining-codebases/references/code-figure-archetypes.md && echo "code-figures exists"
grep -rn '—' skills/explaining-codebases/ && echo "EM-DASH" || echo "clean"
```
Expected: `frontmatter OK`, counts >= noted, both `exists` lines, `clean`.

- [ ] **Step 6: Commit**

```bash
git add skills/explaining-codebases
git commit -m "feat: add explaining-codebases skill

Code intake (navigate, pick angle, map architecture, pull real snippets) and
code-specific figure archetypes. Defers to creating-explainers for the output
and requires the fact-check gate against the real code before delivery.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>"
```

---

## Task 7: Plugin manifests, README, CLAUDE.md, and evals

**Files:**
- Create: `DEST/.claude-plugin/plugin.json`
- Create: `DEST/.claude-plugin/marketplace.json`
- Create: `DEST/README.md`
- Create: `DEST/CLAUDE.md`
- Create: `DEST/evals/evals.json`

- [ ] **Step 1: Write `.claude-plugin/plugin.json` (verbatim)**

```json
{
  "name": "explain-this",
  "description": "Create distill-style interactive explainers - single-file, zero-dependency HTML with hand-built canvas figures - from documents, web research, or a codebase, with a fact-checking gate so no incorrect claim ships.",
  "version": "0.1.0",
  "author": {
    "name": "analyticalmonk"
  },
  "homepage": "https://github.com/analyticalmonk/explain-this",
  "repository": "https://github.com/analyticalmonk/explain-this",
  "license": "MIT"
}
```

- [ ] **Step 2: Write `.claude-plugin/marketplace.json` (verbatim)**

```json
{
  "name": "explain-this",
  "owner": {
    "name": "analyticalmonk"
  },
  "plugins": [
    {
      "name": "explain-this",
      "source": "./",
      "skills": [
        "./skills/creating-explainers",
        "./skills/explaining-codebases",
        "./skills/fact-checking-explainers"
      ],
      "description": "Create distill-style interactive explainers from documents, web research, or a codebase, with a fact-checking gate so no incorrect claim ships."
    }
  ]
}
```

- [ ] **Step 3: Write `README.md` to this spec**

Adapt the prior README. Sections: a one-paragraph intro (distill-style interactive explainers, single self-contained `index.html`, zero deps); a "## The skills" section briefly describing the three (`creating-explainers`, `explaining-codebases`, `fact-checking-explainers`) and when each fires; "## Install" with `/plugin marketplace add analyticalmonk/explain-this` then `/plugin install explain-this@explain-this`; "## Usage" with example trigger phrases for all three use cases (files, research, codebase) and a note that fact-checking runs automatically as a gate; "## What's in this repo" tree reflecting the 3-skill layout; "## License" MIT. No em-dashes.

- [ ] **Step 4: Write `CLAUDE.md` to this spec**

Adapt the prior CLAUDE.md for three skills. Sections:
- **What this repo is** - a plugin with three skills producing single-file HTML explainers; the universal `skills/<name>/SKILL.md` layout; no build/test/lint.
- **The three skills and how they relate** - `creating-explainers` is the hub (output format + workflow); `explaining-codebases` extends it for code; `fact-checking-explainers` is the gate both must pass. Cross-references are by skill name, never `@`-includes.
- **Editing these skills** - where substantive edits go (each SKILL.md frontmatter triggers the skill; references are lazy-loaded; keep them lazy; the template holds the structural invariants).
- **House constraints (skills fail if these break)** - zero dependencies in produced articles; no em-dashes anywhere; single-file articles; **and the new one: no explainer ships with an unverified factual claim - fact-checking-explainers is a hard gate, not advisory.**
- **Testing changes** - no automated runner; the five eval prompts in `evals/evals.json`; `python3 -m http.server` to view a produced article; the per-skill quality checklists are the pre-delivery checks.

- [ ] **Step 5: Write `evals/evals.json` (verbatim)**

```json
{
  "plugin_name": "explain-this",
  "evals": [
    {
      "id": 1,
      "name": "topic-mode-rlhf",
      "skill": "creating-explainers",
      "prompt": "Make an interactive explainer about Reinforcement Learning from Human Feedback (RLHF) - the technique behind ChatGPT's helpfulness training. Aim for around a 12-15 minute read with 3-4 interactive figures. Save the article as rlhf/index.html. The accent color and specific figure choices are up to you.",
      "expected_output": "A complete rlhf/index.html that matches the distill-style two-column layout, has a sticky sidebar TOC with scroll-tracking, uses the initCanvas() utility, has 3-4 interactive figures each implemented as a self-contained IIFE, follows the conversational second-person voice, and passes the post-draft fact-check (every claim sourced).",
      "files": []
    },
    {
      "id": 2,
      "name": "document-mode-pasted-excerpt",
      "skill": "creating-explainers",
      "prompt": "Turn the following excerpt into an interactive explainer. Save as transformers/index.html.\n\n=== SOURCE ===\n\nThe Transformer architecture, introduced in 'Attention is All You Need' (Vaswani et al., 2017), abandons recurrence in favor of self-attention. At its core: each position in a sequence computes a weighted sum over all other positions, with weights derived from learned query-key similarities. This lets every token attend to every other token in parallel, eliminating the sequential bottleneck of RNNs.\n\nThe key mechanism is multi-head attention. Rather than computing a single attention pattern, the model runs attention multiple times in parallel with different learned projections - each 'head' learns to focus on different relationships. One head might track syntactic dependencies; another might track coreference. Heads are concatenated and projected back to the model dimension.\n\nPosition is injected via positional encodings (sinusoidal in the original paper, learned in most modern variants). Without them, the model would be permutation-invariant - 'dog bites man' and 'man bites dog' would look identical.\n\n=== END SOURCE ===\n\nFocus on the multi-head attention mechanism - that's the most figure-worthy part. ~10 minute read, 3 figures.",
      "expected_output": "A transformers/index.html that explains the Transformer (focusing on multi-head attention) using the skill's patterns. 3 figures implemented as IIFEs - likely an attention-pattern visualizer, a multi-head comparison, and something showing positional encoding effect. Passes the post-draft fact-check against the provided excerpt.",
      "files": []
    },
    {
      "id": 3,
      "name": "topic-mode-minimal-pagerank",
      "skill": "creating-explainers",
      "prompt": "Make an interactive explainer about how Google's PageRank algorithm works. You decide the angle, length, and figures. Save it as pagerank/index.html.",
      "expected_output": "A pagerank/index.html article. Tests whether the outline-first workflow surfaces an angle and gets confirmation. Figure choices well-motivated (likely a graph with iterative score propagation, plus a damping-factor comparison). Passes the post-draft fact-check.",
      "files": []
    },
    {
      "id": 4,
      "name": "codebase-mode-self",
      "skill": "explaining-codebases",
      "prompt": "Create an interactive explainer that walks a new contributor through how this plugin's three skills fit together - the creating-explainers hub, explaining-codebases, and the fact-checking gate. Use the actual skill files in this repo as the source. Save as how-explain-this-works/index.html.",
      "expected_output": "A how-explain-this-works/index.html built via explaining-codebases: distill layout with structure-specific figures (e.g. a module/architecture diagram of the three skills and their cross-references, and a stepper through the intake -> fact-check -> deliver workflow), real snippets quoted from the SKILL.md files with path references, and a passing post-draft fact-check where every claim about the skills matches the actual files.",
      "files": []
    },
    {
      "id": 5,
      "name": "fact-check-planted-errors",
      "skill": "fact-checking-explainers",
      "prompt": "Fact-check this explainer draft against its cited source and flag every unsupported or contradicted claim.\n\nSOURCE CITED: 'Attention is All You Need' (Vaswani et al., 2017), which introduced the Transformer architecture from Google Brain and Google Research, replacing recurrence with self-attention. It reports BLEU scores on WMT 2014 English-to-German (28.4) and English-to-French (41.8).\n\nDRAFT:\nThe Transformer, introduced in 2015, abandoned recurrence for self-attention. It was developed by researchers at OpenAI. On release it achieved 99.9% accuracy on all translation benchmarks, a result that reshaped the field. Its core mechanism is multi-head attention, where several attention patterns are computed in parallel.",
      "expected_output": "A claim-by-claim report that flags: (1) 'introduced in 2015' = contradicted (source says 2017); (2) 'developed by researchers at OpenAI' = contradicted (source says Google Brain and Google Research); (3) '99.9% accuracy on all translation benchmarks' = unsupported/contradicted (source reports BLEU 28.4/41.8, not accuracy, and makes no such claim). The multi-head attention claim = supported. Overall gate verdict: FAIL until the three are corrected or cut.",
      "files": []
    }
  ]
}
```

- [ ] **Step 6: Validate**

```bash
cd /home/analyticalmonk/open_source/explain-this
python3 -m json.tool .claude-plugin/plugin.json >/dev/null && echo "plugin.json valid"
python3 -m json.tool .claude-plugin/marketplace.json >/dev/null && echo "marketplace.json valid"
python3 -m json.tool evals/evals.json >/dev/null && echo "evals.json valid"
# marketplace lists all three skills
python3 - <<'PY'
import json
m=json.load(open('.claude-plugin/marketplace.json'))
sk=m['plugins'][0]['skills']
assert sk==["./skills/creating-explainers","./skills/explaining-codebases","./skills/fact-checking-explainers"], sk
print("3 skills listed")
PY
grep -rn '—' README.md CLAUDE.md .claude-plugin/ evals/ && echo "EM-DASH" || echo "clean"
```
Expected: three `valid` lines, `3 skills listed`, `clean`.

- [ ] **Step 7: Commit**

```bash
git add .claude-plugin README.md CLAUDE.md evals/evals.json
git commit -m "feat: add plugin manifests, README, CLAUDE.md, and evals

explain-this manifest pair listing all three skills, human-facing README,
contributor CLAUDE.md with the no-unverified-claim house rule, and evals.json
extended with a codebase eval and a planted-error fact-check eval.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>"
```

---

## Task 8: Whole-plugin validation

A final pass confirming the plugin is internally consistent and the critical guarantee works.

**Files:** none created; this task only validates and, if needed, fixes.

- [ ] **Step 1: Structural sweep**

```bash
cd /home/analyticalmonk/open_source/explain-this
echo "== no em-dashes anywhere =="
grep -rn '—' skills/ .claude-plugin/ README.md CLAUDE.md evals/ && echo "FAIL: em-dash found" || echo "PASS"
echo "== every SKILL.md has matching name =="
for d in skills/*/; do
  n=$(basename "$d")
  grep -q "^name: $n$" "$d/SKILL.md" && echo "PASS $n" || echo "FAIL $n"
done
echo "== referenced reference files all exist =="
python3 - <<'PY'
import re,glob,os
bad=[]
for sk in glob.glob('skills/*/SKILL.md'):
    base=os.path.dirname(sk)
    txt=open(sk).read()
    for ref in re.findall(r'(?<!/)references/[A-Za-z0-9_-]+\.md', txt):  # local refs only, not cross-skill paths
        if not os.path.exists(os.path.join(base,ref)):
            bad.append((sk,ref))
print("PASS" if not bad else ("FAIL "+str(bad)))
PY
```
Expected: `PASS` for em-dashes, `PASS <name>` for all three skills, `PASS` for references.

- [ ] **Step 2: Cross-reference sanity**

```bash
cd /home/analyticalmonk/open_source/explain-this
# the three skill names should each be referenced by at least one other skill
for n in creating-explainers explaining-codebases fact-checking-explainers; do
  c=$(grep -rl "$n" skills/*/SKILL.md | grep -v "skills/$n/SKILL.md" | wc -l)
  echo "$n referenced by $c other skill(s)"
done
# no @-includes of skills/ anywhere
grep -rn '@skills/' skills/ && echo "FAIL: @-include" || echo "PASS: no @-includes"
```
Expected: each name referenced by >= 1 other skill; `PASS: no @-includes`.

- [ ] **Step 3: Fact-checker smoke test (validates the hard requirement)**

Run the planted-error eval (id 5 in `evals/evals.json`) against the `fact-checking-explainers` skill. Use the skill to verify the draft.

Expected result: the skill produces a claim-by-claim report that flags all three planted errors with the correct verdicts (2015 = contradicted; OpenAI = contradicted; 99.9% accuracy = unsupported/contradicted), marks the multi-head attention claim supported, and returns an overall FAIL verdict. If it misses any planted error or passes the draft, the `fact-checking-explainers` SKILL.md needs strengthening (revisit Task 5) before this plan is complete.

- [ ] **Step 4: Optional end-to-end smoke test**

If time allows, run one article-producing eval (id 2, the Transformers excerpt) through `creating-explainers` and confirm: the outline pause happens, the article is a single self-contained `index.html`, figures render, and the post-draft fact-check gate runs and passes against the provided excerpt. This is optional because it is expensive; the structural checks plus the fact-checker smoke test are the required gates.

- [ ] **Step 5: Final commit (only if Steps 1-3 required fixes)**

```bash
git add -A
git commit -m "fix: resolve whole-plugin validation findings

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>"
```
If no fixes were needed, skip this commit.

---

## Spec Coverage

Every spec section maps to a task:

| Spec section | Task(s) |
|---|---|
| Goal; three intakes; no incorrect claim ships | Whole plan; gate in Task 5 |
| Output artifact (reused conventions) | Task 2 |
| Plugin structure | File Structure section; all tasks |
| Skill 1: creating-explainers (hub; composable + mixed intake) | Tasks 2, 3, 4 |
| Skill 2: explaining-codebases | Task 6 |
| Skill 3: fact-checking-explainers (Iron Law, verdicts, two gates, report) | Task 5 |
| Reuse mapping (refs, template, intakes, SKILL, manifests, README, CLAUDE, evals) | Tasks 1, 2, 3, 4, 7 |
| Cross-referencing convention (names, no @-includes) | Tasks 4-6; validated in Task 8 |
| Evals (3 reused + 2 new) | Task 7; smoke tests in Task 8 |
| Naming decisions | Frontmatter in Tasks 4-6; manifests in Task 7 |
| Out of scope (YAGNI) | Respected; no orchestrator, harness, deploy, or index |

No spec requirement is left without a task.
