# Verification Report Format

The fact-check produces one report: a table with a row per checkable claim, then an overall verdict.

## The Table

| # | Claim (quoted) | Location | Verdict | Source / evidence | Severity | Recommended fix |
|---|----------------|----------|---------|-------------------|----------|-----------------|

Column notes:
- **Claim (quoted):** the exact words from the article, kept short. Trim the middle with `...` if long.
- **Location:** the section heading, the figure caption, or `path:line` for a codebase explainer.
- **Verdict:** `supported` / `needs-source` / `unsupported` / `contradicted`.
- **Source / evidence:** for prose, the URL or title plus the quoted passage that settles it. For code, the `path:line` plus the quoted code. Write "not found" when nothing supports it.
- **Severity:** `high` for a numeric, named, dated, or mechanism error a reader could act on or repeat; `low` for a minor imprecision that does not change the reader's understanding.
- **Recommended fix:** the concrete edit (correct to X, add citation Y, soften, or cut).

## The Overall Verdict

After the table, one line:
- `PASS` only if every row is `supported`.
- `FAIL` otherwise, with a count of unresolved rows by verdict, for example: `FAIL - 1 contradicted, 2 unsupported, 1 needs-source`.

## Worked Example

| # | Claim (quoted) | Location | Verdict | Source / evidence | Severity | Recommended fix |
|---|----------------|----------|---------|-------------------|----------|-----------------|
| 1 | "introduced in 2017" | Intro | supported | Vaswani et al., "Attention is All You Need" (2017) | low | none |
| 2 | "developed at OpenAI" | Intro | contradicted | Same paper: authors are at Google Brain and Google Research | high | Correct to "Google Brain and Google Research" |
| 3 | "achieved 99.9% accuracy" | Results | unsupported | No source reports this; the paper gives BLEU 28.4 / 41.8, not accuracy | high | Cut, or replace with the real BLEU figures and cite them |

Overall: `FAIL - 1 contradicted, 1 unsupported`.

## Using the Report

- **As a gate** (called by `creating-explainers` or `explaining-codebases`): return the report inline. Do not deliver the article until the verdict is `PASS`. Apply the recommended fixes, then re-run on the changed claims.
- **On direct request:** present the report to the user and offer to apply the fixes.
