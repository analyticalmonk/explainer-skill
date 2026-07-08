---
name: fact-checking-explainers
description: Use to fact-check an explainer against its sources before delivery. Trigger on "fact-check this explainer", "verify the claims", "check this against its sources", research-time source checks, or the post-draft gate. Ensures every checkable claim traces to a cited source or, for code, the real implementation. Required by creating-explainers and explaining-codebases.
---

# Fact-Checking Explainers

## Overview

An explainer's authority comes from being right. A reader who catches one confident, wrong claim stops trusting the rest of the piece, figures and all. This skill makes "no incorrect claim ships" enforceable by tying every checkable claim to evidence you can point to. It is not a proofreading pass and not advisory: it is the gate an explainer passes before it is delivered.

## The Iron Law

```
NO EXPLAINER SHIPS WITH AN UNVERIFIED FACTUAL CLAIM.
```

Every checkable claim either traces to a source that supports it, gets corrected to match the source, or gets cut.

**No exceptions:**
- Not for "it's obviously true" - obvious things are wrong often enough; verify.
- Not for "I'm pretty sure" - pretty sure is the `unsupported` verdict. Find the source or cut the claim.
- Not for "it's just background" - background facts are still facts, and still wrong sometimes.
- Not for "the draft is due" - a wrong explainer is worse than a late one.
- "Verify" means against an actual source you can point to, not your memory of one.

## When to Use

Run this skill:
- **At research-time**, over the sources gathered for an explainer, before any drafting.
- **Post-draft**, over the finished article, before delivery. This is a blocking gate.
- **On request**, whenever asked to fact-check, verify, or audit an explainer or article.

`creating-explainers` and `explaining-codebases` both call this skill at the gates above. When they do, the article is not done until this skill returns PASS.

**When NOT to use:** content that is not an explainer, or a pure opinion or editorial piece that makes no factual claims. (Most explainers make many factual claims. Default to running it.)

## What Counts as a Checkable Claim

A checkable claim is a specific factual or empirical assertion:
- numbers, dates, measurements, percentages
- names, attributions, who-did-what, who-built-what
- direct quotes
- historical events and their order
- performance results and benchmarks
- mechanism claims: "X uses Y", "X causes Y", "X is faster than Y because Z"

Not checkable (do not flag these as factual claims):
- clearly-labeled interpretation or opinion ("the most elegant part is...")
- analogies and illustrative scenarios the reader knows are illustrative
- the author's framing and transitions

**Heuristic: if you are unsure whether something is a checkable claim, treat it as one and verify it.** The cost of over-checking is small; the cost of a wrong claim shipping is the whole article's credibility.

## The Verification Process

1. **Extract** every checkable claim, with its location in the article (section, figure caption, or `file:line` for code).
2. **Find the support.** For each claim, locate the passage in the cited source that backs it, and read that passage. For codebase explainers, open the actual code at the exact path and line. Do not treat the existence of a citation as support; a cited source often does not say what the draft claims.
3. **Assign a verdict** (table below).
4. **Resolve** everything that is not `supported`.

| Verdict | Meaning | Allowed at delivery? |
|---------|---------|----------------------|
| supported | A source passage directly backs the claim | Yes |
| needs-source | Plausible but no citation yet | No - add a source or downgrade |
| unsupported | No source backs it | No - source it, soften, or cut |
| contradicted | A source says otherwise | No - correct it or cut it |

## The Two Gates

**Research-time gate** (inside `creating-explainers` research intake). Before drafting, go through the gathered sources and confirm each one exists and actually supports the points it will be used for. Fabricated, misremembered, or misread sources are cheapest to catch here, before they are baked into prose.

**Post-draft gate** (every explainer, before delivery). Audit every checkable claim in the finished article against its cited source. **For codebase explainers, the source of truth is the real code.** Every "this function does X", every quoted snippet, every architecture claim is checked against the actual implementation at a specific path and line. Code drifts; a snippet that was right yesterday may be wrong today.

## Resolution

At delivery, every checkable claim must be `supported`. For anything that is not, pick one:
- **Add the supporting citation** (turns `needs-source` into `supported`).
- **Correct the claim** to match what the source actually says.
- **Soften** to clearly-labeled interpretation, if the statement is genuinely interpretive rather than factual.
- **Cut it.**

Shipping with an unresolved `needs-source`, `unsupported`, or `contradicted` claim violates the Iron Law. There is no "ship it with a caveat" option for a factual claim you could not support.

## The Report

Produce a claim-by-claim report plus an overall PASS/FAIL verdict. The exact format is in `references/verification-report-format.md`. The verdict is FAIL until every checkable claim is `supported`.

When invoked as a gate by another skill, return the report inline and do not let the article be delivered until PASS. When invoked directly by a user, present the report and offer to apply the fixes.

## Rationalization Table

| Excuse | Reality |
|--------|---------|
| "I wrote it, so I know it's right" | You know what you intended. Verify what you actually wrote. |
| "It's a well-known fact" | Well-known facts are wrong often enough to check. It takes 30 seconds. |
| "The source probably says this" | Probably is not a verdict. Open the source and confirm. |
| "Close enough" | Numbers, dates, and names are exact or they are wrong. |
| "It's only one claim" | One confident wrong claim discredits the whole explainer. |
| "I'll flag it and let the reader decide" | The gate is your job, not the reader's. |

## Red Flags

These thoughts mean stop and verify before the claim goes in:
- "I'm fairly sure that..."
- "If I recall correctly..."
- "roughly..." / "about..." / "something like..." attached to a specific number
- "it's basically the same as..."
- pasting a statistic without re-opening the source it came from
- citing a URL you have not actually opened in this session

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Verifying that the citation exists, not that it supports the claim | Read the cited passage. A real source can still fail to back the claim. |
| Checking the easy claims, waving through the hard ones | The hard-to-verify claims are exactly where errors hide. |
| For code: trusting a comment, a function name, or a variable name | Read the implementation. Names and comments can lie; the code does not. |
| Treating `needs-source` as shippable | `needs-source` is a to-do, not a pass. Resolve it. |
| Stopping at the first source that agrees | If sources disagree, that disagreement belongs in the article, not buried. |
