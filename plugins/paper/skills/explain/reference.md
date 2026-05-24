# Paper writing — detailed reference

Load when the SKILL.md summary isn't enough.

## Class-file URLs

| Journal | URL |
| --- | --- |
| MNRAS | `https://mirrors.ctan.org/macros/latex/contrib/mnras/mnras.cls`<br>`https://mirrors.ctan.org/macros/latex/contrib/mnras/mnras.bst` |
| A&A | `https://mirrors.ctan.org/macros/latex/contrib/aa/aa.cls`<br>`https://mirrors.ctan.org/macros/latex/contrib/aa/aa.bst` |
| ApJ / AAS | https://journals.aas.org/aastexguide/ (download `aastex631.cls`) |
| JCAP | uses `iopart.cls`; submit via the IOP web form which compiles from raw .tex |
| PRD | `https://journals.aps.org/files/revtex4-2.zip` |

## Citation command reference (natbib)

| Command | Renders as |
| --- | --- |
| `\citep{Smith2020}` | (Smith 2020) |
| `\citet{Smith2020}` | Smith (2020) |
| `\citealt{Smith2020}` | Smith 2020 |
| `\citep{a,b,c}` | (a; b; c) |
| `\citeauthor{Smith2020}` | Smith |
| `\citeyear{Smith2020}` | 2020 |
| `\citep[e.g.][]{Smith2020}` | (e.g. Smith 2020) |
| `\citep[][p.~12]{Smith2020}` | (Smith 2020, p. 12) |

## bibtex template entries

```bibtex
@article{Smith2024,
  title   = {The thing}, author = {Smith, A. and Jones, B.},
  year    = {2024}, journal = {MNRAS}, volume = {500}, pages = {1--10},
  doi     = {10.1093/mnras/...}, eprint = {2401.12345},
}

@misc{arxiv_only,                       % for arxiv preprints
  title  = {...}, author = {...}, year = {2024},
  eprint = {2401.12345}, archivePrefix = {arXiv}, primaryClass = {astro-ph.CO},
}

@inproceedings{conference,
  title     = {...}, author = {...}, year = {2024},
  booktitle = {Proc. ...}, pages = {...},
}

@book{book,
  title = {...}, author = {...}, year = {2024},
  publisher = {Cambridge University Press},
}

@misc{software,                         % for software releases
  title  = {...}, author = {...}, year = {2024},
  howpublished = {\url{https://github.com/...}},
}
```

## Makefile template

```makefile
PAPER       := paper
LATEXMK     := latexmk -pdf -interaction=nonstopmode -halt-on-error

.PHONY: all figs clean distclean help

all: $(PAPER).pdf

$(PAPER).pdf: $(PAPER).tex references.bib
	$(LATEXMK) $(PAPER).tex

figs: $(wildcard figures/*.pdf figures/*.png)

clean:
	$(LATEXMK) -c $(PAPER).tex
	rm -f *.bbl *.blg

distclean: clean
	rm -f $(PAPER).pdf
```

## .gitignore template

```
*.aux
*.bbl
*.blg
*.fdb_latexmk
*.fls
*.log
*.out
*.synctex.gz
*.toc
*.lof
*.lot
__pycache__/
*.pyc
.DS_Store
```

## TikZ recipes

### Box-and-arrow flowchart

See SKILL.md — minimal example. Style key clashes to avoid:
`in`, `out`, `at`, `to`, `from`, `pos`, `angle`. Suffix with `box`,
`node`, `_block` to disambiguate.

### Annotated equation flow

```latex
\node[box] (a) {input};
\node[box, right=of a] (b) {middle};
\draw[arr] (a) -- node[above, font=\scriptsize, midway] {\(\sigma(R)\)} (b);
```

### Subgraph with rounded fit

```latex
\node[fit=(a)(b)(c), draw=black!30, rounded corners,
      inner sep=8pt, label=above:{stage 1}] {};
```

## valency-mcp cheat sheet

The MCP server provides arxiv + CrossRef-backed paper search. Most
useful tool calls when populating `references.bib`:

```python
# Direct lookup when you know the arxiv ID — fastest path
get_paper_by_id(paper_id="2106.03846")

# Keyword search when you don't
search_by_title(query="CosmoPower neural emulators", limit=3)

# Author search (with optional date range)
search_by_author(author="Spurio Mancini", start_date="2020-01-01", limit=10)

# Semantic search for conceptual queries
semantic_search_papers(query="differentiable cosmology JAX", limit=10)

# Batch export bibtex once you have the IDs
export_papers_bibtex(paper_ids=["2106.03846", "1712.00788", ...])
```

## Pre-submission checklist (per journal)

### MNRAS
- [ ] Abstract has no citations
- [ ] Keywords include 4--6 entries from the official MNRAS keyword list
- [ ] Acknowledgements section present (use the exact heading)
- [ ] Data Availability section present
- [ ] All figures have captions ending in a full stop
- [ ] No `??` in the PDF (undefined refs)
- [ ] Word count check via `texcount paper.tex` (MNRAS letters have a hard limit)

### A&A
- [ ] Abstract structured as 5 paragraphs (Aims, Methods, Results, ...)
- [ ] Keywords from the A&A keyword list
- [ ] No more than 6 keywords

### ApJ
- [ ] Plain abstract OK; structured abstract optional
- [ ] Use `\affiliation{}` not `\institute{}`
- [ ] AAS keywords (different from MNRAS / A&A!)
