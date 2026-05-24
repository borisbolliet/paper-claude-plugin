---
description: Scientific-paper writing knowledge (MNRAS, A&A, ApJ, JCAP). Covers LaTeX class setup, bibtex via valency-mcp, figure pipelines (matplotlib + TikZ), the Makefile/latexmk build, citation style, common pitfalls (abstract citations, TikZ key clashes, matplotlib without usetex, bibtex requirements), and the recipe to scaffold a new paper repo end-to-end. Auto-loads when the conversation is about writing a paper, formatting LaTeX, building a manuscript PDF, drafting an abstract, or working with references.bib.
when_to_use: User mentions writing a paper, LaTeX template, MNRAS / A&A / ApJ / JCAP / Letter, mnras.cls, latexmk, references.bib, bibtex, citation style, \citep / \citet, figure captions, abstract, draft, manuscript, paper.tex, scaffold a paper, journal submission, arxiv.
allowed-tools: Read Grep Glob Bash(latexmk *) Bash(pdflatex *) Bash(bibtex *) Bash(pdftoppm *) Bash(pdfinfo *) Bash(pdftotext *) Bash(curl -sSL -o *) Bash(ls *) Bash(cat *) Bash(grep *) Bash(find *)
---

# Scientific paper writing — knowledge skill

Help with writing and building scientific papers in LaTeX. The recipes
below are battle-tested from real submissions; the pitfalls section
captures everything we've watched go wrong.

## Project layout (recommended)

```
your-paper/
├── paper.tex            # main manuscript
├── <journal>.cls        # journal class file (vendored, see Recipes below)
├── <journal>.bst        # journal bibstyle (vendored)
├── references.bib       # bibtex (populated via valency-mcp)
├── figures/             # all paper figures (PDF or PNG)
├── scripts/             # python that produces the figures
├── Makefile             # `make` → paper.pdf via latexmk; `make figs`
├── README.md            # build instructions, dependencies
├── .gitignore           # ignore .aux/.bbl/.log/__pycache__
└── LICENSE
```

Use `/paper:scaffold` to bootstrap this layout for a given journal.

## Build

Default toolchain: `latexmk -pdf -interaction=nonstopmode paper.tex`.
The Makefile should provide:

- `make`         → build paper.pdf
- `make figs`    → regenerate everything in `figures/`
- `make clean`   → remove aux files
- `make distclean` → also remove paper.pdf

Use `/paper:build` for a quick build + error summary; the skill reads
`paper.log` and `paper.blg` and reports only what matters.

## References (via valency-mcp)

The `valency-mcp` server provides arxiv/CrossRef-backed paper search +
bibtex export. The cleanest workflow:

1. `search_by_title(query, limit=3)` — to find a paper if you don't
   know the arxiv ID. Returns paper objects with `id`, `url`,
   `citation_count`.
2. `search_by_author(author, start_date=..., limit=10)` — to find
   all relevant papers by a researcher.
3. `export_papers_bibtex(paper_ids=[...])` — batch fetch bibtex.
4. For papers not on arxiv (JMLR, software, books), write the
   `@misc` / `@book` / `@article` entry by hand.

Use `/paper:refs` to drive this end-to-end given a list of topics.

### Common pitfalls with bibtex

- **`@article` entries MUST have a `journal=` field**, otherwise the
  bst will complain. For arxiv preprints use `@misc` with
  `eprint = {...}, archivePrefix = {arXiv}, primaryClass = {astro-ph.CO}`.
- Author names with accents: prefer LaTeX form (`{\'e}`) over UTF-8.
- `@book` and `@inproceedings` need a `publisher=` / `booktitle=`
  respectively.

## Figures

Two recipes, picked by complexity:

### A. Matplotlib script → PNG/PDF

For plots driven by code/data. Pattern:

```python
# scripts/make_<name>.py
import matplotlib
matplotlib.use("Agg")
import matplotlib.pyplot as plt
fig, ax = plt.subplots(figsize=(7, 4.5), dpi=300)
# ...
fig.savefig("figures/<name>.png", dpi=300, bbox_inches="tight")
```

**Pitfall**: matplotlib does NOT process LaTeX macros (`\textbf{...}`,
`\sigma`, etc.) unless `matplotlib.rcParams['text.usetex'] = True` —
which itself requires a working TeX install on the runtime path. Without
it, `\textbf{foo}` appears verbatim. Safe defaults:

- Plain text + matplotlib's mathtext: `r"$\sigma_8$"` works, but
  `r"\textbf{label}"` does NOT.
- Use `weight="bold"` keyword instead of `\textbf{}`.

### B. Native TikZ in paper.tex

For schematics / diagrams. Always vector, always matches paper fonts.
Use this when:

- The figure is a flowchart / box-and-arrow diagram
- Text consistency with paper body matters
- Sparse data can be hand-coded

Minimal TikZ preamble in `paper.tex`:

```latex
\usepackage{tikz}
\usetikzlibrary{shapes.geometric,arrows.meta,positioning,fit,calc}
```

Minimal flow:

```latex
\begin{tikzpicture}[
  font=\small, node distance=8mm and 11mm,
  every node/.style={align=center, inner sep=4pt},
  box/.style={rectangle, rounded corners=2pt, draw=black!50,
              line width=0.4pt, minimum height=10mm, minimum width=20mm},
  inp/.style={box, fill=blue!8},
  arr/.style={-{Stealth[length=2mm]}, line width=0.5pt, draw=black!65},
]
  \node[inp]                       (input) {input};
  \node[box, right=of input]       (mid)   {middle};
  \node[inp, right=of mid]         (out)   {output};
  \draw[arr] (input) -- (mid);
  \draw[arr] (mid)   -- (out);
\end{tikzpicture}
```

**Pitfall**: do NOT name a TikZ style `out` — it clashes with the
built-in `/tikz/out` key (used by arrow path-bending). Use `outbox`,
`output_box`, etc.

## Citation style

- Use `\citep{key}` for parenthetical (Author 2024), `\citet{key}`
  for inline "Author (2024)". For multiple cites:
  `\citep{a, b, c}` renders as "(a; b; c)" in MNRAS.
- `\citealt{key}` strips the parentheses entirely (for use inside
  larger parens).
- `\citeauthor{key}` and `\citeyear{key}` if you only want one piece.

## Domain-knowledge gotchas (cosmology / SZ)

When writing about the halo-model code lineage, citation
attributions matter:

- `szfastdks` (the `dks` is **Dolag-Komatsu-Sunyaev**) is from
  Dolag, Komatsu \& Sunyaev 2016 (MNRAS 463, 1797). NOT
  Dunkley+13 (which is the ACT likelihood paper).
- `class_sz` v1 (Bolliet+18) computes the GNFW profile Fourier
  transform via **explicit numerical Hankel transforms**, not
  FFTLog. FFTLog is what `classy_szlite` uses (via `mcfit`).
- The `v2` CosmoPower emulators in the
  `cosmopower-organization/ede` repository are the emulators used
  in the ACT DR6 extended-cosmology analysis (Calabrese et al.
  2025, arXiv:2503.14454) and in the ACT DR6 + DESI DR2 EDE
  analysis by Poulin et al.\ 2025 (arXiv:2505.08051).

Before claiming "code X uses method Y", verify against the
upstream source --- it is easy to confuse close-but-different
techniques (FFTLog vs explicit Hankel, NFW vs GNFW, Tinker 2008
vs 2010, etc.).

## What NOT to do

- **No citations in the abstract** (MNRAS, A&A, ApJ all discourage
  this — readers don't have the bibliography in front of them). The
  abstract should be self-contained.
- **No floating equation numbers**: prefer `\label{eq:foo}` only on
  equations you actually reference.
- **No subsection-level TBDs in a near-submission draft** — fill
  the whole section even with a 2-line placeholder rather than
  leaving `TBD`.
- **No oversized PNGs**: matplotlib at `dpi=300` is plenty for a
  column-width figure. Above 300dpi the file gets large with no
  visual benefit.
- **Don't `\input` figure scripts** — generate the figure file via
  `make figs` and `\includegraphics` it.

## MNRAS-specific tips

- Use the official `mnras.cls` from CTAN (see Recipes).
- Class options: `[a4paper, fleqn, usenatbib]` for two-column A4 with
  flush-left equations and natbib citation commands.
- `\bibliographystyle{mnras}`; `\bibliography{references}`.
- The class auto-formats sections like `\section*{Acknowledgements}`
  and `\section*{Data Availability}` — use the exact words.

## A&A-specific tips

- Use the `aa.cls` from CTAN (`aastex` package).
- Author block: `\author{...\inst{1} \and ...\inst{2}}` +
  `\institute{...}`.
- Bibstyle: `aa.bst`.

## ApJ / AAS-specific tips

- Use `aastex631.cls` (or whatever the current version is) from
  https://journals.aas.org.
- Use `\paragraph*{Acknowledgements}` style sections.
- bibstyle: `aasjournal`.

## Recipes

### Fetch MNRAS class + bibstyle

```bash
curl -sSL -o mnras.cls "https://mirrors.ctan.org/macros/latex/contrib/mnras/mnras.cls"
curl -sSL -o mnras.bst "https://mirrors.ctan.org/macros/latex/contrib/mnras/mnras.bst"
```

Then `\documentclass[a4paper,fleqn,usenatbib]{mnras}` in `paper.tex`.

### Make placeholder figures so paper.tex compiles early

LaTeX errors out on missing `\includegraphics` files. Generate stubs:

```bash
python -c "
import matplotlib; matplotlib.use('Agg')
import matplotlib.pyplot as plt
for name in ['fig1','fig2','fig3']:
    fig, ax = plt.subplots(figsize=(6, 4))
    ax.text(0.5, 0.5, f'[placeholder: {name}]', ha='center', va='center')
    ax.set_axis_off()
    fig.savefig(f'figures/{name}.png', bbox_inches='tight')
"
```

### Diagnosing a build failure

1. `pdflatex -interaction=nonstopmode paper.tex 2>&1 | grep -E '^!|Error'`
2. `grep -E "Citation .* undefined" paper.log | head -5`
3. `cat paper.blg | grep -iE "error|warning"`

Common causes (in order of frequency):

- Missing `\citep` keys → bibtex hasn't run, or the key is misspelled.
- `\includegraphics` filename missing → check `figures/`.
- TikZ key clash (e.g. `out` / `in` / `at`) → rename the style.
- `@article` without `journal=` → either add the field or change to `@misc`.

## Detailed reference

For full lists of citation commands, journal-specific keywords, and
worked examples, see [reference.md](reference.md).
