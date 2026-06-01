# Code Intake

Use this when the explainer's subject is a codebase or a set of source files. The goal of intake is the same as for a paper - arrive at an approved outline with a clear spine - but the source is live code you navigate, not prose you read.

## Step 1: Pick the angle (with the user)

Two angles, and they want different articles:

- **Architecture / overview**: how the pieces fit. Good for onboarding. The spine is the system's shape - its modules and the data that moves between them.
- **Mechanism / deep-dive**: how one thing works, end to end. The spine is a single path through the code - a request, a query, a training step, a frame.

State the spine as one sentence, the same one-sentence-throughline discipline the file and research intakes use: "An HTTP request from socket to handler", "How the scheduler picks the next task to run". If you cannot write that sentence, you do not understand the code well enough yet. Confirm the angle with the user before going deep - overview and deep-dive produce different figures and different lengths.

## Step 2: Navigate

Build a map before you decide what to explain:

- **Entry points**: `main`, the server bootstrap, the CLI handler, the exported public API, or the test that exercises the feature.
- **Build and config**: what the project is (language, framework, package manifest) and how it is wired together.
- **Module structure**: the top-level directories and what each is responsible for.

Use directory listings, search for definitions and call sites, and read the key files. For a large or unfamiliar repo, do a read-only exploration pass first (a subagent or a broad search) so you scope from knowledge, not from the first file you happened to open.

## Step 3: Find the spine

Pick the single path or concept the article tracks. For an overview, it is the backbone the other modules hang off. For a deep-dive, it is the call chain from the entry point to the result. Everything in the article serves the spine; cut what does not.

## Step 4: Map the architecture

For the spine, write down: the modules involved, what each is responsible for, the data that flows between them, and the key dependencies. This map becomes the architecture figure and the section structure. Keep it to the parts the reader needs, not the whole dependency graph.

## Step 5: Pull real snippets

Quote actual code, short (5-20 lines), each tagged with its `path:line`. Trim the middle with `...`, but never invent code, and never tidy a snippet into something the repo does not contain. The snippets are the article's ground truth; the fact-check gate will check every one against the file it came from.

## Step 6: Outline and get approval

Propose the outline in the same format as the other intakes (title, overline, subtitle, sections, figures), plus the snippet list. Pause for user approval before drafting. Cheap to fix the outline; expensive to rewrite a finished article.

## Pitfalls

- **Snippets drifting from the real code.** Re-quote from the file; do not work from memory of what the function looked like.
- **Explaining generated or vendored code** as if the project hand-wrote it. Note when code is generated, and prefer explaining the source it is generated from.
- **Diagrams that assert structure the code does not have.** Build every diagram from real imports and call sites.
- **Over-broad scope.** One subsystem or one mechanism per article. A whole framework does not fit in 3000 words.

Every claim and snippet you produce here is later checked by `fact-checking-explainers` against the code, so capture exact paths and lines as you go.
