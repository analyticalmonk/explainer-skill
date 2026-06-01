# Voice and Style

The look of these articles is half the work; the voice is the other half. A pixel-perfect layout with neutral, encyclopedic prose is *not* what this skill produces. Read this before drafting any prose.

## The Voice in One Paragraph

Conversational, second-person, technical-but-accessible. You're explaining a thing you find genuinely interesting to a smart friend who hasn't seen it before. You don't condescend, but you also don't assume they've read the paper. You favor concrete scenes over definitions, analogies over jargon, and short paragraphs over long ones. You build tension - "here's the problem nobody can solve, now here's how X solves it" - and pay it off.

## The First Sentence

Open with a scene or a tension, never a definition. The opening should make the reader curious, not pre-load them with vocabulary.

**Bad** (definition-first):
> Diffusion models are a class of generative models that learn to invert a forward noising process.

**Good** (scene-first, from `world-models/`):
> Every LLM application starts the same way. You write a prompt. You test it. It doesn't work. You add "think step by step." You rearrange the instructions. You sprinkle in examples. Eventually it works - until you switch models and everything breaks.

**Good** (tension-first, paraphrased style):
> Imagine pushing a ball off a table. You don't need to solve the equations of motion - you already know it will fall, bounce, settle. That intuition is a world model. The question is whether a neural network can have one.

## Paragraph Length

Aim for **80-180 words per paragraph**. Vary it. A long paragraph followed by a one-sentence punchline lands harder than two medium paragraphs.

Bullet lists in body prose are a smell. If you find yourself writing one, try narrating instead:

**Smell:**
> DSPy is built on three abstractions:
> - Signatures
> - Modules
> - Optimizers

**Better:**
> DSPy is built on three core abstractions. Signatures declare input/output behavior. Modules implement strategies for calling LMs. Optimizers tune prompts and weights to maximize a metric. Together, they form a programming model for language models that mirrors how PyTorch works for neural networks.

Bullet lists *are* fine inside concept cards, comparison cells, and quick-reference tables. The rule is for body prose.

## Sentence Length

Vary it deliberately. Long sentences carry nuance; short sentences carry punch. A wall of medium-length sentences reads as machine-generated.

```
Long, then short:
"DSPy shifts your focus from tinkering with prompt strings to programming
with structured and declarative natural-language modules - and once you
internalize the metaphor, prompt engineering starts to feel like assembly
language. You stop writing it."
```

## Second Person

Address the reader as "you". Not "we", not "the user", not "one". The whole point of an interactive article is that the reader is doing something.

- "**You** drag the camera to rotate the model."
- "**Try** changing temperature and watch the distribution flatten."
- Not: "The user can adjust temperature to see the distribution flatten."

## Analogies and Concrete Examples

Every abstract concept should land on a concrete image. Some real ones from the project:

- DSPy's optimizer is like a compiler: you write the program, it figures out the prompts.
- Pi-agent's tree search is like exploring a maze with a flashlight.
- Gravity in the world-models simulation is like pushing a ball off a table.

When introducing a new term, pair it with the analogy first, definition second:

> A **signature** is the type declaration of an LLM call - inputs in, outputs out, like a function signature in any other language. Concretely: `question -> answer`.

## Setting Up Figures

Every figure needs to be set up by the prose immediately before it. The reader should know, before they look at the figure:

1. What it shows
2. What they can do with it
3. What to look for

Don't just drop a figure in. Use a sentence like *"To see this in action, try stepping through the simulation below - watch the orange ghost diverge from the ball as the model's prediction degrades."*

The figure caption then reinforces (1) and (3), with `<strong>` on the term to track:

```html
<div class="figure-caption">
  A ball falls under gravity. The <strong>orange ghost</strong> shows the model's
  prediction for the next frame. Watch how it diverges as the ball nears the floor.
</div>
```

## Callouts

Use `.callout` for genuine "key insight" moments - the things you'd want a reader to remember if they only remembered three things. **3-5 per article maximum**. If every paragraph has a callout, none of them stand out.

```html
<div class="callout">
  <strong>The key insight:</strong> the model isn't predicting pixels - it's
  predicting the latent code that decodes to pixels. That's why it can hallucinate
  futures it never saw in training.
</div>
```

Phrasings that work:
- "The key insight:"
- "What's surprising:"
- "The crucial move:"
- "Notice that:"

## Epigraphs

Use `.epigraph` for one or two pull-quotes from the source material - usually near the start of a section, to set context. Keep them short. One per article is plenty; two is the cap.

```html
<div class="epigraph">
  <p>DSPy shifts your focus from tinkering with prompt strings to programming
  with structured and declarative natural-language modules.</p>
  <cite>DSPy Documentation</cite>
</div>
```

## Things to Avoid

| Avoid | Why | Use instead |
|-------|-----|-------------|
| Em-dashes (`—`) | Project convention - akash@ specifically dislikes them | Hyphens `-`, or rewrite the sentence |
| "We will see..." | Filler, sounds academic | Just describe what's there |
| "It is important to note..." | Filler | Note it directly, or cut |
| "In this article, we explore..." | Throat-clearing | Open with the actual content |
| "Simply" / "just" | Patronizing | Drop the adverb |
| "Obviously" | Reader-condescending | Drop, or replace with "of course" if conversational |
| Over-hedging ("might possibly maybe") | Reads as uncertain | Pick a stance |
| Wikipedia-tone | "X, also known as Y, is a method that..." | Conversational tone |
| Unexplained jargon | Loses the reader | Define on first use, with an analogy |

## Length Targets

A typical article runs **2000-3500 words** of prose, with **3-5 figures**. Read time on the article header is the prose-only word count divided by ~250 wpm, rounded.

If you're at 5000+ words, you have an essay, not an explainer. Cut.

If you're at 800 words, you have a blog post. Either expand with more concrete examples, or this might not be the right format.

## A Worked Example

Here's the opening of `world-models/` (verbatim) with annotations:

```
Imagine pushing a ball off a table. You don't need to solve the equations of    [scene-first opener]
motion to know what happens next - it falls, bounces a few times, settles.       [tension/setup]
You have an intuition for how the world works.                                  [pivots to concept]

This intuition is what AI researchers call a world model. The question that's   [defines term, sources it]
animated the field for decades: can a neural network have one?

In the last two years, three approaches have emerged, each making a different   [previews structure]
trade-off between fidelity and generality. Genie generates frames directly,
treating the world as a sequence of images. JEPA predicts in a latent space,
trading visual fidelity for abstract reasoning. World Labs builds explicit
geometry, sacrificing scope for grounded simulation.

The figures below let you play with each.                                       [foreshadows interactivity]
```

Things this does well:
- Opens with a scene (ball off a table) anyone can picture
- Builds to the concept ("world model") only after the reader has the intuition
- Introduces the structure of the article (three approaches) so the reader has a roadmap
- Ends the intro by promising interactivity - the reader knows they'll be doing something soon

## Checklist for Prose

Before declaring the prose pass done:

- [ ] Opens with a scene or tension, not a definition
- [ ] No bullet lists in body paragraphs (concept cards and comparisons are fine)
- [ ] Every figure has a setup paragraph immediately before it
- [ ] Every figure caption tells the reader what to look for, with a `<strong>` term
- [ ] 3-5 callouts max, marking genuinely-important takeaways
- [ ] Zero em-dashes (search for `—` and replace)
- [ ] Voice is second-person throughout
- [ ] Sentence length varies - no walls of medium-length sentences
- [ ] Defines jargon on first use, paired with an analogy where possible
