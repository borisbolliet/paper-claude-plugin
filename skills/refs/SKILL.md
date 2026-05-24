---
description: Populate references.bib with bibtex entries for a list of topics or arxiv IDs, using the valency-mcp tools (search_by_title, search_by_author, get_paper_by_id, export_papers_bibtex). Falls back to hand-written @misc entries for papers not on arxiv (JMLR, software releases, books, IAU symposia).
argument-hint: "<topic1> <topic2> ... | <arxiv_id> ... | --author=Name"
allowed-tools: Read Edit Write Bash(grep *) Bash(cat *)
---

# Populate references.bib via valency-mcp

Drive the `valency-mcp` server to find and fetch bibtex entries for a
list of topics or arxiv IDs, then merge them into `references.bib`
in the current paper repo.

User input in `$ARGUMENTS`: a free-form list of topics, arxiv IDs,
or author names with optional flags. Examples:

- `/paper:refs CosmoPower emulators Tinker mass function FFTLog Hamilton`
- `/paper:refs 2106.03846 1712.00788 0803.2706`
- `/paper:refs --author="Spurio Mancini" --start-date=2020-01-01`

## Steps

1. **Read the current `references.bib`** to see what's already there;
   you'll merge by cite key, never duplicate.

2. **For each topic / ID**:
   - If it looks like an arxiv ID (`\d{4}\.\d{4,5}` or
     `[a-z\-]+/\d{7}`) → `get_paper_by_id`.
   - Otherwise → `search_by_title` (with `limit=3, include_abstract=false`)
     and pick the top hit. If unsure, ask the user.
   - For author searches → `search_by_author`.

3. **Batch fetch bibtex** via `export_papers_bibtex(paper_ids=[...])`
   — much faster than one-at-a-time calls.

4. **Rename cite keys** to short, memorable forms before merging
   (`FirstAuthorYearKeyword`, e.g. `SpurioMancini2022`, `Tinker2008`,
   `ACTDR62025`). The valency tool returns keys like
   `spurioMancini2022cosmopoweremulating...` which are correct but ugly.

5. **Fix common bibtex issues** before merging:
   - `@article` entries without `journal=` → convert to `@misc` with
     `eprint = ..., archivePrefix = {arXiv}, primaryClass = {...}`,
     OR add the journal field if you know it.
   - UTF-8 accents → LaTeX form (`{\\'e}` etc.).
   - URLs with `<i>` or `<sub>` HTML tags from CrossRef → strip them.

6. **Hand-write entries** for sources not on arxiv:
   - JMLR papers (e.g.\ NUTS, Hoffman-Gelman 2014).
   - Software releases (`\\misc` with `howpublished = {\\url{...}}`).
   - IAU symposia, conference proceedings, observatory tech notes.
   - PhD theses.

7. **Append to `references.bib`** under a sensible section header
   (e.g. `% ---- Halo model ----`, `% ---- Surveys ----`,
   `% ---- JAX / SBI ----`). Group thematically, not chronologically.

8. **Rebuild** to verify (`latexmk -pdf paper.tex`); report any
   remaining `Citation .* undefined` warnings.

## Pitfalls

- **Generic title searches return junk**: "Hamilton FFTLog" might
  return the wrong Hamilton paper. Always show the user the top hit
  + abstract before committing if you're not 100% sure.
- **Wrong CrossRef metadata**: `\\textit{i}` and `<sub>` HTML tags
  occasionally leak through. Scan `references.bib` after writing.
- **Cite-key collisions**: never have two `@article{Smith2020, ...}`
  entries with the same key — bibtex silently uses the first.
- **Group editor with care**: if the user runs `/paper:refs` repeatedly
  and we keep adding to references.bib, deduplicate by key.
