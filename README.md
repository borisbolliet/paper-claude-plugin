# Scientific paper writing (Claude Code plugin)

Specialised [Claude Code](https://code.claude.com) assistance for
writing scientific papers in LaTeX (MNRAS, A&A, ApJ, JCAP, PRD).
Captures the recipes and pitfalls learned writing real submissions.

## What you get

- **`/paper:explain`** — always-on knowledge skill. Auto-loads when the
  conversation is about writing a paper. Covers project layout,
  citation style, figure pipelines (matplotlib + native TikZ),
  bibtex, the Makefile/latexmk build, and common pitfalls (abstract
  citations, TikZ key clashes, matplotlib `\textbf` without usetex,
  bibtex `@article` needing `journal=`).

- **`/paper:scaffold <path> [--journal=mnras|aa|apj]`** — bootstrap a
  new paper repo with the recommended layout: class file vendored
  from CTAN, paper.tex skeleton, Makefile, placeholder figures so
  the document compiles immediately, README, LICENSE, .gitignore.

- **`/paper:refs <topic1> <topic2> ... | <arxiv_id> ...`** — populates
  `references.bib` via the [`valency-mcp`](https://github.com/valency-ai/valency-mcp)
  server (arxiv + CrossRef-backed search + bibtex export). Handles
  rename to short cite keys, fixes common bibtex issues
  (`@article` without `journal=`, HTML tags in titles, etc.).

- **`/paper:build [--pages=N-M] [--clean]`** — runs `latexmk`,
  surfaces only the errors that matter, reports PDF page count
  and undefined-reference count. Can render specific pages to
  PNG for visual review.

- **`paper-writer` subagent** — specialist for end-to-end paper
  writing tasks. Use for heavy multi-step work where draft +
  figure-generation + build-iteration output would otherwise
  flood the main thread.

## Compose with

This plugin pairs naturally with
[`valency-mcp`](https://github.com/valency-ai/valency-mcp) (for
references) and your favourite plotting/code plugin.

## Install

In any Claude Code session, run these three commands:

```
/plugin marketplace add https://github.com/borisbolliet/paper-claude-plugin.git
/plugin install paper@paper-claude-plugin
/reload-plugins
```

After `/reload-plugins`, `/paper:explain`, `/paper:scaffold`,
`/paper:refs`, `/paper:build` show in `/help` and the
`paper-writer` subagent appears in the Agent picker.

To update later (after I push a new commit):

```
/plugin uninstall paper@paper-claude-plugin
/plugin marketplace remove paper-claude-plugin
/plugin marketplace add https://github.com/borisbolliet/paper-claude-plugin.git
/plugin install paper@paper-claude-plugin
/reload-plugins
```

## Environment

- LaTeX: TeX Live 2024 or newer (MacTeX on macOS is fine).
  `latexmk`, `pdflatex`, `bibtex`, `pdfinfo`, `pdftoppm`,
  `pdftotext` should be on `$PATH`.
- Python (for figure generation): your favourite venv, with at least
  `matplotlib`.
- `valency-mcp` connected (for `/paper:refs`).

## Try it

```
/paper:scaffold ~/GitHub/my-paper --journal=mnras --title="My great paper"
/paper:refs CosmoPower Tinker mass function FFTLog Hamilton no-U-turn sampler
/paper:build --pages=1-3
```

## Layout

```
.claude-plugin/marketplace.json    # single-plugin marketplace
plugins/paper/
  .claude-plugin/plugin.json       # plugin manifest
  skills/
    explain/SKILL.md               # always-on knowledge
    explain/reference.md           # loaded on demand
    scaffold/SKILL.md              # /paper:scaffold
    refs/SKILL.md                  # /paper:refs
    build/SKILL.md                 # /paper:build
  agents/
    paper-writer.md                # subagent
```

## License

MIT
