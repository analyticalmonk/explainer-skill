# Intake From Files

Use this workflow when the user has provided source material - a paper, blog post, transcript, research report, slide deck, or doc dump. The article is a *reading* of that source, not original research.

## Step 1: Read the source thoroughly

Before outlining anything, read the whole source. Use the `Read` tool for files. For URLs, use `WebFetch`. For long PDFs, read in 20-page chunks (the `pages` parameter on `Read`).

While reading, take notes - in your head or in a scratch file - on:

- **The central tension or claim.** What problem does this work address? What is the surprising thing it shows?
- **3-5 key concepts.** What terms or ideas does the reader need to understand? Each will likely become a section or a figure.
- **Concrete moments.** Specific numbers, examples, scenarios that would land well in prose. (E.g., "8000% improvement on accuracy" or "the agent waited 47 minutes before giving up".)
- **What would benefit from a figure?** Look for: a process the reader could step through, a comparison between alternatives, a parameter whose effect is interesting, a spatial or geometric structure.
- **Quotes worth pulling.** One or two crisp lines from the source for an `.epigraph`.

If the source is dense or technical, you may need to read it twice - the second pass with the goal of "what would I show the reader?".

## Step 2: Identify the central narrative

Every good explainer has a single throughline. The reader should be able to summarize the article in one sentence after finishing. Examples:

- DSPy: "Stop tuning prompts by hand; treat LM calls as compilable programs."
- World Models: "There are three competing approaches to teaching neural networks an intuitive sense of how the world works."
- Pi-Agent: "Robotic manipulation works best when planning, perception, and control are co-designed."

If you can't write that one sentence yet, you don't understand the source well enough. Re-read.

## Step 3: Outline the article

Propose to the user:

```
TITLE: <punchy, claim-shaped, 3-12 words>
OVERLINE: <attribution, e.g., "Stanford NLP / DSPy">
SUBTITLE: <one italic sentence promising what the reader gets>

SECTIONS:
1. <Section title> - <one-sentence summary>
2. <Section title> - <one-sentence summary>
3. <Section title> - <one-sentence summary>
...

FIGURES:
1. <Figure title> - <what it shows> - <how the reader interacts> - <archetype>
2. <Figure title> - ...
...
```

Aim for 4-7 sections and 3-5 figures. If the source has a natural numbered structure (e.g., a paper with three method variants), follow it. Otherwise, design your own arc:

- **Hook** (the tension or scene) → **the problem** → **the proposed approach** → **how it works** (often the densest, with figures) → **why it matters** / **caveats**.

Pause here. Get the user's approval on the outline. Cheap to fix the outline; expensive to rewrite a finished article.

## Step 4: Map source content to sections

For each section in the outline, list the specific paragraphs/slides/passages from the source that feed it. This gives you a checklist while drafting and prevents you from forgetting key material.

## Step 5: Draft prose, then figures

Write the prose pass first, with figure placeholder divs in the right positions. Each figure placeholder should have a comment naming its archetype:

```html
<!-- FIG 1: Tabbed comparison of signature variants -->
<div class="figure" id="fig-sig"></div>
```

Then go back and implement each figure, removing the placeholder comments as you go. This separation prevents you from getting lost in figure code and forgetting to write the prose around it.

## Pitfalls When Working From Files

- **Over-summarizing.** A good explainer is not a précis of the source. You're reading the source aloud, not abridging it. Pull on the threads that excite *you*.
- **Quoting too much.** One or two epigraphs is plenty. Long quotes lose the conversational voice.
- **Inheriting jargon.** Papers use jargon for shorthand. Articles need to define every term on first use, with an analogy.
- **Following the source's structure too closely.** Papers are organized for peer reviewers; articles are organized for curious readers. Don't be afraid to reorder sections.
- **Skipping the figure-design step.** Source documents rarely come with figures suitable for interactive rendering. Most figures need to be designed from scratch based on the underlying concepts.

## Multiple Sources

If the user provided several documents, do a first pass identifying which is the "spine" (the primary source the article tracks) and which are supporting (additional context, criticisms, alternative framings). The article should follow the spine's narrative, with supporting sources cited as comparisons or footnotes.

## Combining With Research (Mixed Intake)

The given files are not always enough. If the topic needs material the files do not cover - recent developments, criticisms, corroboration, or context the source omits - supplement with web research. Read `intake-from-research.md` and combine the two: treat the given files as the spine (the narrative the article tracks) and the researched sources as supporting. Synthesize across everything into a single outline. Run the research-time fact-check on the researched sources only; the given files are user-provided ground truth.
