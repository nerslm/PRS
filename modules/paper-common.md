# Paper Context Module — Planning, Figures, Writing, Compilation, Improvement

This module covers `paper_plan`, `paper_figures`, `paper_write`, `paper_compile`, and `paper_improve` phases. It merges the domain constraints from five original skills into a single context reference. Constants (REVIEWER_MODEL, TARGET_VENUE, MAX_PAGES, etc.) are now in `project.json`; venue profiles live at `references/venues/{venue}.md`; writing principles at `references/writing-principles.md`; citation discipline at `references/citation-discipline.md`.

---

## Inputs

Across all sub-phases, the pipeline expects these files (accumulated progressively):

| Sub-phase | Required Inputs |
|-----------|----------------|
| paper_plan | NARRATIVE_REPORT.md or STORY.md; GPT54_AUTO_REVIEW.md (optional); experiment results in `figures/` or `results/`; IDEA_REPORT.md (optional) |
| paper_figures | PAPER_PLAN.md (figure plan table); experiment data (JSON/CSV) in `figures/` or project root; existing manually-created figures |
| paper_write | PAPER_PLAN.md; NARRATIVE_REPORT.md; generated figures in `figures/`; `figures/latex_includes.tex`; existing `.bib` file (optional) |
| paper_compile | `paper/main.tex`; `paper/sections/*.tex`; `paper/references.bib`; figures |
| paper_improve | Compiled `paper/main.pdf` + all LaTeX source files |

If PAPER_PLAN.md is missing when entering paper_write, ask the user to run paper_plan first.
If none of the narrative documents exist when entering paper_plan, ask the user to describe the paper's contribution in 3-5 sentences.

## Outputs

| Sub-phase | Output Files |
|-----------|-------------|
| paper_plan | `PAPER_PLAN.md` |
| paper_figures | `figures/paper_plot_style.py`, `figures/gen_fig*.py`, `figures/*.pdf`, `figures/latex_includes.tex`, `figures/TABLE_*.tex` |
| paper_write | `paper/main.tex`, `paper/sections/*.tex`, `paper/math_commands.tex`, `paper/references.bib` |
| paper_compile | `paper/main.pdf`, `paper/compile.log` |
| paper_improve | `paper/main_round0_original.pdf`, `paper/main_round1.pdf`, `paper/main_round2.pdf`, `paper/PAPER_IMPROVEMENT_LOG.md`, `paper/PAPER_IMPROVEMENT_STATE.json` |

---


## Quality Constraints

These constraints apply across ALL sub-phases:

1. **Claims-Evidence Matrix is the backbone** — every claim must map to evidence, every experiment must support a claim. This matrix is created in paper_plan and referenced in every subsequent phase.

2. **Anti-hallucination citations** — NEVER fabricate BibTeX. Use the DBLP -> CrossRef -> `[VERIFY]` chain. LLM-generated BibTeX frequently hallucinates venue names, page numbers, or co-authors.

3. **Page budget is hard** — MAX_PAGES counts main body only (first page through end of Conclusion). References and appendix are NOT counted. If content exceeds MAX_PAGES, move material to appendix.

4. **Anonymous submission handling** — when ANONYMOUS mode is active (default for submission), use `\author{Anonymous Authors}`. No author names, emails, affiliations, or self-citations that reveal identity.

5. **Front-load the story** — the paper's contribution should be clear from title, abstract, introduction, and hero figure before the reader reaches the full method. Reviewers often form a judgment before reading past the introduction.

6. **De-AI polish is mandatory** — every draft must be scanned for the watch-word list and common AI writing patterns before finalization.

7. **Clean bib** — `references.bib` must contain ONLY entries that are actually `\cite`d in the paper. No bloat.

8. **No stale files** — every `.tex` file in `sections/` must be `\input`ed by `main.tex`. When restructuring sections, delete orphaned files.

9. **Large file handling** — if the Write tool fails due to file size, immediately retry using Bash (`cat << 'EOF' > file`) to write in chunks. Do NOT ask the user for permission — just do it silently.

10. **Backup before overwrite** — never silently destroy existing `paper/` directory. Back up to `paper-backup-{timestamp}/` first.

---

## Verification Criteria

### paper_plan -> paper_figures transition
- PAPER_PLAN.md exists in the project root
- Contains Claims-Evidence Matrix, section plan, and figure plan

### paper_figures -> paper_write transition
- `figures/` directory has at least one figure file
- `figures/latex_includes.tex` exists with LaTeX include snippets

### paper_write -> paper_compile transition
- `paper/main.tex` exists
- `paper/sections/` contains at least one `.tex` file
- `paper/references.bib` exists

### paper_compile -> paper_improve transition
- `paper/main.pdf` exists (latexmk succeeded with 0 errors)
- Page count verified against MAX_PAGES

### paper_improve -> done transition
- Improvement rounds complete (max rounds from `project.json`)
- `paper/PAPER_IMPROVEMENT_LOG.md` exists with score progression
- Final PDF compiled successfully

### Cross-phase verification (run at each transition)
- No `[VERIFY]` markers remaining in any `.tex` file
- All `\ref{}` and `\label{}` pairs resolve
- All `\citep{}` / `\citet{}` have BibTeX entries
- No TODO/FIXME/XXX markers in text
- references.bib contains only cited entries
