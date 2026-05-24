---
description: Build the current paper repo's PDF via latexmk, surface only the errors that matter (undefined citations, missing figures, TikZ key clashes, bibtex problems), and report the page count + size. Optionally render selected pages to PNG for visual review.
argument-hint: "[--pages=1-4] [--clean]"
allowed-tools: Read Bash(latexmk *) Bash(pdflatex *) Bash(bibtex *) Bash(pdfinfo *) Bash(pdftoppm *) Bash(pdftotext *) Bash(grep *) Bash(rm *) Bash(ls *)
---

# Build the paper

Run `latexmk -pdf -interaction=nonstopmode paper.tex` and surface only
what matters: errors, undefined references, missing figures, and any
bibtex warnings.

Arguments in `$ARGUMENTS`:

- `--pages=N` or `--pages=N-M` — render those pages to PNG via
  `pdftoppm` at 150 dpi into `/tmp/<basename>-N.png` and report the
  paths. Useful for quick visual review after a TikZ change.
- `--clean` — `latexmk -C` first to nuke all aux files. Use when the
  build is in a weird state (e.g. after a class-file swap).

## Steps

1. If `--clean` was passed: `latexmk -C paper.tex`.
2. Run `latexmk -pdf -interaction=nonstopmode paper.tex`.
3. Parse output: tail the latexmk-collected error summary.
4. If `paper.pdf` exists, report:
   - page count: `pdfinfo paper.pdf | grep Pages`
   - file size: `ls -la paper.pdf`
   - undefined-citation count: `grep -c "Citation .* undefined" paper.log`
   - bibtex warnings: `grep -iE "warning|error" paper.blg | head`
5. If `--pages` was passed: render and list the PNGs.
6. Report concisely.

## What to surface vs ignore

| Surface | Ignore |
| --- | --- |
| `! ...` pdflatex error lines | `Overfull \\hbox` warnings (cosmetic) |
| `Citation .* undefined` (from .log) | `pdfTeX warning (dest)` (harmless) |
| `! Package pgfkeys Error` (TikZ key clash) | `Underfull \\vbox` warnings |
| `Bibtex errors` from .blg | `Font shape ... undefined` (usually fine) |
| Missing `\\includegraphics` files | TeX-live default font substitution notes |

## Common error patterns + fixes

| Error | Fix |
| --- | --- |
| `File 'figures/X.pdf' not found` | Create a placeholder PNG/PDF, or run `make figs` first |
| `Citation 'X' undefined` | Add the entry to `references.bib` via `/paper:refs` |
| `! Package pgfkeys Error: The key '/tikz/out' requires a value` | Rename your TikZ style — don't use `out`, `in`, `at` |
| `--- they aren't the same literal types for entry X` (bibtex) | `@article` is missing the `journal=` field; either add it or switch to `@misc` |
| `Empty journal in X` (bibtex warning) | Same as above |
| Latexmk says "errors" but `paper.pdf` exists with the right page count | Stale `.fls` / `.fdb_latexmk`; rerun with `--clean` |

## Pitfalls

- **`latexmk` exit code is unreliable**: it sometimes reports errors
  even when the PDF built fine (stale error caches). Always check
  the actual `paper.pdf` mtime / page count after a "failed" run.
- **Bibtex needs two latex passes**: latexmk handles this, but if you
  run `pdflatex` directly you need to interleave with `bibtex`.
