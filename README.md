# explain-this

A Claude Code plugin for creating distill-style interactive explainers - single self-contained `index.html` pages with a sticky two-column layout, hand-built Canvas figures, and conversational prose, like the articles at [distill.pub](https://distill.pub). Zero dependencies, no build step.

## The skills

- **creating-explainers** - the hub. Turns a paper, blog post, transcript, or research report into an explainer, or researches a topic from scratch, or both. Owns the template, figures, voice, and the staged workflow.
- **explaining-codebases** - explains a repository or set of source files: an onboarding overview of how a project is structured, or a deep-dive on how one mechanism works. Same output format, with code navigation and code-specific figures.
- **fact-checking-explainers** - a gate the other two pass before delivery. Every checkable claim must trace to a source (or, for code, the real implementation), or be corrected or cut. No incorrect claim ships.

Each skill triggers automatically when you describe the matching goal. You can also invoke fact-checking on its own.

## Install

### Claude Code (recommended)

```
/plugin marketplace add analyticalmonk/explain-this
/plugin install explain-this@explain-this
```

To pick up updates later:

```
/plugin marketplace update explain-this
```

### Other tools (Codex, Cursor, Gemini CLI, project-local `.claude/skills/`, etc.)

The repo follows the universal `skills/<name>/SKILL.md` layout, so any tool that loads skills from a directory can use it. Clone and point your loader at the inner skill folders:

```bash
git clone https://github.com/analyticalmonk/explain-this.git
# point your loader at:
#   <clone-path>/skills/creating-explainers/
#   <clone-path>/skills/explaining-codebases/
#   <clone-path>/skills/fact-checking-explainers/
```

## Usage

Claude picks the right skill up automatically when you say things like:

- "Make an interactive explainer about RLHF" (research intake)
- "Turn this paper into a distill-style article" (files intake)
- "Explain how this repo's scheduler works, as an interactive guide" (codebase)

Whichever path you take, the explainer is fact-checked before it is delivered: every claim is traced to its source or the real code, and anything unsupported is corrected or cut.

## What's in this repo

```
.claude-plugin/
  plugin.json                       # Claude Code plugin manifest
  marketplace.json                  # marketplace entry for /plugin marketplace add
skills/
  creating-explainers/
    SKILL.md
    assets/article-template.html    # complete HTML skeleton, copy and fill in {{PLACEHOLDERS}}
    references/                      # intake (files / research), figures, voice, template, palettes
  explaining-codebases/
    SKILL.md
    references/                      # code intake, code-specific figure archetypes
  fact-checking-explainers/
    SKILL.md
    references/verification-report-format.md
evals/
  evals.json                        # 5 reference prompts the skills are developed against
```

The references load lazily - Claude reads them only when relevant, so the skills don't burn context up front.

## License

MIT - see [LICENSE](LICENSE).
