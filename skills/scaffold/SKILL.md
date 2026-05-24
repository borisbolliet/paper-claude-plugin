---
description: Scaffold a new scientific-paper repository: paper.tex skeleton, journal class file (MNRAS / A&A / ApJ), Makefile, .gitignore, README, LICENSE, empty references.bib, figures/ + scripts/ directories, and placeholder figure PNGs so the document compiles immediately.
disable-model-invocation: true
argument-hint: "<path> [--journal=mnras|aa|apj] [--title=\"...\"] [--author=\"...\"]"
allowed-tools: Read Write Edit Bash(mkdir *) Bash(git init *) Bash(cd * && git init *) Bash(curl -sSL -o *) Bash(ls *) Bash(latexmk *) Bash(python *)
---

# Scaffold a new paper repository

Bootstrap a new LaTeX paper repo with the recommended layout (see
`/paper:explain`). Output: a `<path>/` directory that compiles cleanly
to `paper.pdf` from minute zero, with all the build infrastructure in
place so the user can immediately start writing.

Arguments in `$ARGUMENTS`:

- `$0` — target directory (e.g. `~/GitHub/my-new-paper`). Created if
  missing.
- `--journal=mnras|aa|apj` — journal style (default `mnras`).
- `--title="..."` — paper title (defaults to placeholder).
- `--author="..."` — first-author name (defaults to `\\author{TBD}`).

## Steps

1. **Create the directory** + initialise `git` if not already.
   Create subdirs: `figures/`, `scripts/`.

2. **Fetch the journal class file + bibstyle**. From CTAN mirrors:
   ```bash
   # mnras
   curl -sSL -o mnras.cls "https://mirrors.ctan.org/macros/latex/contrib/mnras/mnras.cls"
   curl -sSL -o mnras.bst "https://mirrors.ctan.org/macros/latex/contrib/mnras/mnras.bst"
   # aa
   curl -sSL -o aa.cls    "https://mirrors.ctan.org/macros/latex/contrib/aa/aa.cls"
   curl -sSL -o aa.bst    "https://mirrors.ctan.org/macros/latex/contrib/aa/aa.bst"
   ```
   For ApJ / AAS, ask the user to manually download `aastex631.cls`
   from https://journals.aas.org/aastexguide/.

3. **Write `paper.tex`** from the journal-specific skeleton in
   `/paper:explain` (look there for the exact preamble). Default
   sections: Abstract (no citations), Introduction, Methods,
   Results, Discussion, Acknowledgements, Data Availability,
   Bibliography. Include the TikZ preamble even if not used yet.

4. **Write `references.bib`** — empty placeholder with a header
   comment.

5. **Write `Makefile`** with targets `all`, `figs`, `clean`,
   `distclean`, `help` (see `/paper:explain` for the template).

6. **Write `.gitignore`** with the standard LaTeX + Python ignore
   pattern.

7. **Write `LICENSE`** (default MIT, ask the user if they prefer
   otherwise).

8. **Write `README.md`** with build instructions (`make figs`,
   `make`, dependencies).

9. **Generate placeholder figures** so latex compiles. For each
   `\includegraphics` in the skeleton, emit a 6x4 PNG with the
   filename as the placeholder text. Use the recipe in
   `/paper:explain`.

10. **Verify** with `latexmk -pdf -interaction=nonstopmode paper.tex`.
    Expect zero errors, undefined-reference warnings (because
    references.bib is empty) are OK.

11. **Report back**:
    - Files written (absolute paths)
    - `paper.pdf` page count + size
    - Suggested next steps:
      * `/paper:refs <topic1> <topic2> ...` to populate bibtex
      * Edit `paper.tex` to fill in content
      * `/paper:build` for incremental rebuilds

## Pitfalls

- **CTAN mirror selection**: if `mirrors.ctan.org` times out, fall
  back to a specific mirror like `mirror.ctan.org` or
  `texlive.info`. The `mnras.cls` is in `texlive` so any TeX Live
  install also has it (`kpsewhich mnras.cls`) — vendoring is for
  reproducibility.
- **Existing directory**: if `<path>` already has a `paper.tex`,
  STOP and ask the user before overwriting.
- **Journal class options matter**: MNRAS wants
  `[a4paper, fleqn, usenatbib]`; the class compiles but renders
  weirdly without `usenatbib`.
