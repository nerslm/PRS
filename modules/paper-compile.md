# Paper Sub-module

> Also read `modules/paper-common.md` for shared inputs/outputs, quality constraints, and verification criteria.

## Sub-phase: paper_compile

### Prerequisites Check

```bash
# Check LaTeX installation
which pdflatex && which latexmk && which bibtex

# If not installed:
# macOS: brew install --cask mactex-no-gui
# Ubuntu: sudo apt-get install texlive-full
# Server: conda install -c conda-forge texlive-core
```

Verify all required files exist: `paper/main.tex`, `paper/references.bib`, `paper/sections/*.tex`, figures in `paper/figures/`.

### latexmk Compilation Recipe

```bash
cd paper/

# Clean previous build artifacts
latexmk -C

# Full compilation (pdflatex + bibtex + pdflatex x 2)
latexmk -pdf -interaction=nonstopmode -halt-on-error main.tex 2>&1 | tee compile.log
```

Engine options: `pdflatex` (default), `xelatex` (for CJK/custom fonts), `lualatex`.

### Common Error Fixes

| Error | Fix |
|-------|-----|
| `! LaTeX Error: File 'somepackage.sty' not found.` | Install via `tlmgr install somepackage` or remove unused `\usepackage` |
| `LaTeX Warning: Reference 'fig:xyz' on page 3 undefined` | Check `\label{fig:xyz}` exists in the correct figure environment |
| `! LaTeX Error: File 'figures/fig1.pdf' not found.` | Check for different extension (.png vs .pdf). Update `\includegraphics` path |
| `LaTeX Warning: Citation 'smith2024' undefined` | Add missing entry to `references.bib` or fix citation key |
| `[VERIFY]` markers in text | Search for correct information or flag to user |
| `Overfull \hbox (>20pt too wide)` | Rephrase text or adjust figure width. Minor (<20pt) usually ignorable |
| `I was expecting a ',' or a '}'` (BibTeX) | Fix BibTeX syntax: missing comma, unmatched braces, special characters in title |
| `\crefname` undefined for custom theorem types | Ensure `\crefname{assumption}{Assumption}{Assumptions}` in preamble after `\newtheorem` |

### Iterative Fix Loop

```
for attempt in 1..MAX_COMPILE_ATTEMPTS (default 3):
    compile()
    if success: break
    parse_errors()
    auto_fix()
```

For each error: read the error message from `compile.log`, locate the source file and line number, apply the fix, recompile.

### Page Count Verification

**CRITICAL**: Verify main body fits within MAX_PAGES.

Main body = first page through end of Conclusion section (not necessarily §5 — could be §6, §7, or §8 depending on structure). References and appendix are NOT counted.

**Precise check using pdftotext:**
```bash
pdftotext main.pdf - | python3 -c "
import sys
text = sys.stdin.read()
pages = text.split('\f')
for i, page in enumerate(pages):
    if 'Ethics Statement' in page or 'Reproducibility' in page:
        print(f'Conclusion ends on page {i+1}')
    if any(w in page for w in ['References', 'Bibliography']):
        lines = [l for l in page.split('\n') if l.strip()]
        for l in lines[:3]:
            if 'References' in l or 'Bibliography' in l:
                print(f'References start on page {i+1}')
                break
"
```

If over limit:
- Identify which sections are longest
- Suggest specific cuts (move proofs to appendix, compress tables, tighten writing)
- Report: "Main body is X pages (limit: MAX_PAGES). Suggestion: move [specific content] to appendix."

### Stale File Detection

Check for orphaned section files not referenced by `main.tex`:

```bash
for f in paper/sections/*.tex; do
    base=$(basename "$f")
    if ! grep -q "$base" paper/main.tex; then
        echo "WARNING: $f is not referenced by main.tex — consider removing"
    fi
done
```

### Post-Compilation Checks

```bash
# Check PDF exists and has content
ls -la main.pdf
# Check page count
pdfinfo main.pdf | grep Pages
```

Automated checks:
- [ ] PDF file exists and is > 100KB (not empty/corrupt)
- [ ] Total page count is reasonable (MAX_PAGES + appendix + references)
- [ ] No "??" in the PDF (undefined references — grep the log)
- [ ] No "[?]" in the PDF (undefined citations — grep the log)
- [ ] Figures are rendered (not missing image placeholders)

```bash
# Check for undefined references
grep -c "LaTeX Warning.*undefined" compile.log
# Check for missing citations
grep -c "Citation.*undefined" compile.log
```

### Submission Readiness Checks

- [ ] **Anonymous**: no author names, affiliations, or self-citations that reveal identity
- [ ] **Page limit**: main body within MAX_PAGES (to end of Conclusion)
- [ ] **Font embedding**: all fonts embedded in PDF
  ```bash
  pdffonts main.pdf | grep -v "yes"  # should return nothing (or only header)
  ```
- [ ] **No supplementary mixed in**: appendix clearly after `\newpage\appendix`
- [ ] **File size**: reasonable (< 50MB for most venues, < 10MB preferred)
- [ ] **No `[VERIFY]` markers**: search the PDF text for leftover markers

### Common Venue Requirements

| Venue | Style File | Citation | Page Limit (main body) | Submission |
|-------|-----------|----------|------------------------|------------|
| ICLR 2026 | `iclr2026_conference.sty` | `natbib` (`\citep`/`\citet`) | 9 pages (to Conclusion end) | OpenReview |
| NeurIPS 2026 | `neurips_2026.sty` | `natbib` (`\citep`/`\citet`) | 9 pages (to Conclusion end) | OpenReview |
| ICML 2026 | `icml2026.sty` | `natbib` (`\citep`/`\citet`) | 8 pages (to Conclusion end) | OpenReview |
| PRL | `revtex4-2.cls` (`aps,prl`) | `\cite{}` (RevTeX built-in) | 4 journal pages (~3750 words) | APS Journals |
| Nature Comms | `sn-jnl.cls` (`sn-nature`) | `\cite{}` (Springer style) | ~8-10 pages (flexible) | Springer Nature |

### Compilation Report Output Format

```markdown
## Compilation Report

- **Status**: SUCCESS / FAILED
- **PDF**: paper/main.pdf
- **Pages**: X (main body to Conclusion) + Y (references) + Z (appendix)
- **Within page limit**: YES/NO (MAX_PAGES = N)
- **Errors fixed**: [list of auto-fixed issues]
- **Warnings remaining**: [list of non-critical warnings]
- **Undefined references**: 0
- **Undefined citations**: 0
```

### paper_compile Rules

- Never delete the user's source files — only modify to fix errors
- Keep compile.log — useful for debugging
- Don't suppress warnings — report them, let the user decide
- If LaTeX is not installed, provide clear installation instructions rather than failing silently
- Font embedding is critical — some venues reject PDFs with non-embedded fonts
- Page count = main body to Conclusion — this is the metric that matters for submission

---

