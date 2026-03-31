# Paper Sub-module

> Also read `modules/paper-common.md` for shared inputs/outputs, quality constraints, and verification criteria.

## Sub-phase: paper_write

### Template Selection Logic

Templates are in `templates/` (relative to PRS installation root). Select based on venue from `project.json`:

| Venue | Template File | Style Package | Key Setup |
|-------|--------------|---------------|-----------|
| ICLR | `templates/iclr2026.tex` | `iclr2026_conference.sty` | `\iclrfinalcopy` for camera-ready |
| NeurIPS | `templates/neurips2026.tex` | `neurips_2026.sty` | `[preprint]` or `[final]` option |
| ICML | `templates/icml2026.tex` | `icml2026.sty` | `[accepted]` for camera-ready |
| CVPR | `templates/cvpr2026.tex` | `cvpr.sty` | Also covers ICCV/ECCV |
| ACL | `templates/acl2026.tex` | `acl.sty` | Also covers EMNLP/NAACL |
| AAAI | `templates/aaai2026.tex` | `aaai2026.sty` | — |
| ACM | `templates/acm_mm2026.tex` | `acmart.cls` | ACM MM, SIGIR, KDD, CHI, etc. |
| PRL | `templates/prl.tex` | — | Physical Review Letters |
| Nature Comms | `templates/nature_comms.tex` | — | Nature Communications |

All ML venues (ICLR/NeurIPS/ICML) use `natbib` (`\citep`/`\citet`).

### Project Structure

Generate this file structure:

```
paper/
├── main.tex                    # master file (includes sections)
├── {venue}_style.sty           # venue-specific style file
├── math_commands.tex           # shared math macros
├── references.bib              # bibliography (filtered — only cited entries)
├── sections/
│   ├── 0_abstract.tex
│   ├── 1_introduction.tex
│   ├── 2_related_work.tex
│   ├── 3_method.tex            # or preliminaries, setup, etc.
│   ├── 4_experiments.tex
│   ├── 5_conclusion.tex
│   └── A_appendix.tex          # proof details, extra experiments
└── figures/                    # symlink or copy from project figures/
```

Section files are FLEXIBLE: If the paper plan has 6-8 sections, create corresponding files (e.g., `4_theory.tex`, `5_experiments.tex`, `6_analysis.tex`, `7_conclusion.tex`).

### Workflow

**Step 0: Backup and Clean.** If `paper/` already exists, back up to `paper-backup-{timestamp}/` before overwriting. Never silently destroy existing work. When changing section structure (e.g., 5 sections to 7 sections), delete section files that are no longer referenced by `main.tex`.

**Step 1: Initialize Project.** Create `paper/` directory. Copy venue template from `templates/`. Generate `math_commands.tex` with paper-specific notation. Create section files matching PAPER_PLAN structure.

Anonymous author block:
```latex
\author{Anonymous Authors}
```

**Step 2: Generate math_commands.tex.**
```latex
% math_commands.tex — shared notation
\newcommand{\R}{\mathbb{R}}
\newcommand{\E}{\mathbb{E}}
\DeclareMathOperator*{\argmin}{arg\,min}
\DeclareMathOperator*{\argmax}{arg\,max}
% Add paper-specific notation here
```

**Step 3: Write Each Section.** Process sections in order. For each section:
1. Read the plan — what claims, evidence, citations belong here
2. Read NARRATIVE_REPORT.md — extract relevant content, findings, and quantitative results
3. Draft content — write complete LaTeX (not placeholders)
4. Insert figures/tables — use snippets from `figures/latex_includes.tex`
5. Add citations — use `\citep{}` / `\citet{}` (all ML venues use `natbib`)

Before drafting the front matter, re-read the one-sentence contribution from `PAPER_PLAN.md`. The Abstract and Introduction should make that takeaway obvious before the reader reaches the full method.

### Section-by-Section Writing Guidelines

**§0 Abstract:**
- Use the 5-part flow from `references/writing-principles.md`: what, why hard, how, evidence, strongest result
- Must be self-contained (understandable without reading the paper)
- Start with the paper's specific contribution, not generic field-level background
- Include one concrete quantitative result
- 150-250 words (check venue limit)
- No citations, no undefined acronyms
- No `\begin{abstract}` — that's in main.tex

**§1 Introduction (4-paragraph structure):**
- Open with a compelling hook (1-2 sentences, problem motivation)
- State the gap clearly ("However, ...")
- Give a brief approach overview before the reader gets lost in details
- List 2-4 specific, falsifiable contributions as a numbered or bulleted list
- Preview the strongest result early instead of saving it for the experiments section
- End with a brief roadmap ("The rest of this paper is organized as...")
- Include the main result figure if space allows
- Target: 1-1.5 pages
- Methods should begin by page 2-3 at the latest

**§2 Related Work:**
- **MINIMUM 1 full page** (3-4 substantive paragraphs). Short related work sections are a common reviewer complaint.
- Organize by category using `\paragraph{Category Name.}`
- Organize methodologically, by assumption class, or by research question; do not write paper-by-paper mini-summaries
- Each category: 1 paragraph summarizing the line of work + 1-2 sentences positioning this paper
- Do NOT just list papers — synthesize and compare
- End each paragraph with how this paper relates/differs

**§3 Method / Preliminaries / Setup:**
- Define notation early (reference math_commands.tex)
- Use `\begin{definition}`, `\begin{theorem}` environments for formal statements
- For theory papers: include proof sketches of key results in main body, full proofs in appendix
- For theory papers: include a **comparison table** of prior bounds vs. this paper
- Include algorithm pseudocode if applicable (`algorithm2e` or `algorithmic`)
- Target: 1.5-2 pages

**§4 Experiments:**
- Start with experimental setup (datasets, baselines, metrics, implementation details)
- Main results table/figure first
- Then ablations and analysis
- Every claim from the introduction must have supporting evidence here
- For each major experiment, make explicit what claim it supports and what the reader should notice
- Target: 2.5-3 pages

**§5 Conclusion:**
- Summarize contributions (NOT copy-paste from intro — rephrase)
- Limitations (be honest — reviewers appreciate this)
- Future work (1-2 concrete directions)
- Ethics statement and reproducibility statement (if venue requires)
- Target: 0.5 pages

**Appendix:**
- Proof details (full proofs of main-body theorems)
- Additional experiments, ablations
- Implementation details, hyperparameter tables
- Additional visualizations

### DBLP -> CrossRef -> [VERIFY] Citation Fetch Chain

**CRITICAL: Only include BibTeX entries that are actually cited in the paper.**

1. Scan all `\citep{}` and `\citet{}` references in the drafted sections
2. Build a citation key list
3. For each citation key:
   - Check existing `.bib` files in the project/narrative docs
   - If not found, use the verified fetch chain below
   - **NEVER fabricate BibTeX entries** — mark unknown ones with `[VERIFY]` comment
4. Write `references.bib` containing ONLY cited entries (no bloat)

**Step A: DBLP (best quality — full venue, pages, editors)**
```bash
# 1. Search by title + first author
curl -s "https://dblp.org/search/publ/api?q=TITLE+AUTHOR&format=json&h=3"
# 2. Extract DBLP key from result (e.g., conf/nips/VaswaniSPUJGKP17)
# 3. Fetch real BibTeX
curl -s "https://dblp.org/rec/{key}.bib"
```

**Step B: CrossRef DOI (fallback — works for arXiv preprints)**
```bash
# If paper has a DOI or arXiv ID (arXiv DOI = 10.48550/arXiv.{id})
curl -sLH "Accept: application/x-bibtex" "https://doi.org/{doi}"
```

**Step C: Mark `[VERIFY]` (last resort)**
If both DBLP and CrossRef return nothing, mark the entry with `% [VERIFY]` comment. Do NOT fabricate.

**Why this matters:** LLM-generated BibTeX frequently hallucinates venue names, page numbers, or even co-authors. DBLP and CrossRef return publisher-verified metadata. Upstream skills (lit_survey, novelty check) may mention papers from LLM memory — this fetch chain is the gate that prevents hallucinated citations from entering the final `.bib`.

If the DBLP/CrossRef flow is not enough, load `references/citation-discipline.md` for stricter fallback rules before adding placeholders.

**Automated bib cleaning** — use this Python pattern to extract only cited entries:
```python
import re
# 1. Grep all \citep{...} and \citet{...} from all .tex files
# 2. Extract unique keys (handle multi-cite like \citep{a,b,c})
# 3. Parse the full .bib file, keep only entries whose key is in the cited set
# 4. Write the filtered bib
```
This prevents bib bloat (e.g., 948 lines to 215 lines in testing).

**Citation verification rules:**
1. Every BibTeX entry must have: author, title, year, venue/journal
2. Prefer published venue versions over arXiv preprints (if published)
3. Use consistent key format: `{firstauthor}{year}{keyword}` (e.g., `ho2020denoising`)
4. Double-check year and venue for every entry
5. Remove duplicate entries (same paper with different keys)

### De-AI Polish Rules

After drafting all sections, scan for common AI writing patterns and fix them.

First apply the sentence-level clarity rules from `references/writing-principles.md`:
- keep subject and verb close together,
- put familiar context first and new information later,
- place the most important information near the end of the sentence,
- let each paragraph do one job,
- use verbs for actions instead of nominalized nouns.

Then fix the common content patterns below:
- Significance inflation ("groundbreaking", "revolutionary" -> use measured language)
- Formulaic transitions ("In this section, we..." -> remove or vary)
- Generic conclusions ("This work opens exciting new avenues" -> be specific)

**Language patterns to fix (watch words):**

| Pattern | Action |
|---------|--------|
| delve | Replace with "examine", "investigate", or remove |
| pivotal | Replace with "important", "key", or remove |
| landscape | Replace with "field", "area", or remove |
| tapestry | Remove entirely |
| underscore | Replace with "highlight", "show" |
| noteworthy | Replace with "notable" or remove |
| intriguingly | Remove or replace with "surprisingly" |
| "It is worth noting that" | Remove filler, state directly |
| "Importantly," | Remove filler |
| "Notably," | Remove filler |
| Rule-of-three lists ("X, Y, and Z" appearing repeatedly) | Vary structure |
| Consecutive sentences starting with "This" or "We" | Vary sentence openings |
| Vague "this" references | Replace with concrete noun ("this result", "this ablation", "this theorem") |

### Reverse Outline Test

After drafting all sections:

1. **Extract topic sentences** — pull the first sentence of every paragraph
2. **Read them in sequence** — they should form a coherent narrative on their own
3. **Check claim coverage** — every claim from the Claims-Evidence Matrix must appear
4. **Check evidence mapping** — every experiment/figure must support a stated claim
5. **Fix gaps** — if a topic sentence doesn't advance the story, rewrite the paragraph

### Cross-Review with Reviewer Model

Send the complete draft to GPT-5.4 xhigh. Focus on:
1. Does each claim from the intro have supporting evidence?
2. Is the writing clear, concise, and free of AI-isms?
3. Any logical gaps or unclear explanations?
4. Does it fit within MAX_PAGES (to end of Conclusion)?
5. Is related work sufficiently comprehensive (>= 1 page)?
6. For theory papers: are proof sketches adequate?
7. Are figures/tables clearly described and properly referenced?
8. Would a skim reader understand the contribution from the title, abstract, introduction, and Figure 1?

For each issue, specify: severity (CRITICAL/MAJOR/MINOR), location, and fix. Apply CRITICAL and MAJOR fixes. Document MINOR issues for the user.

### paper_write Final Checks

- [ ] All `\ref{}` and `\label{}` match (no undefined references)
- [ ] All `\citep{}` / `\citet{}` have corresponding BibTeX entries
- [ ] No author information in anonymous mode
- [ ] Figure/table numbering is correct
- [ ] Page count within MAX_PAGES (main body to Conclusion end)
- [ ] No TODO/FIXME/XXX markers left in the text
- [ ] No `[VERIFY]` markers left unchecked
- [ ] Abstract is self-contained (understandable without reading the paper)
- [ ] Title is specific and informative (not generic)
- [ ] Related work is >= 1 full page
- [ ] references.bib contains ONLY cited entries (no bloat)
- [ ] **No stale section files** — every .tex in `sections/` is `\input`ed by `main.tex`
- [ ] **Section files match main.tex** — file numbering and `\input` paths are consistent
- [ ] Venue-specific required sections/checklists satisfied (read `references/venue-checklists.md` if needed)
- [ ] A skim reader can recover the main claim from the title, abstract, introduction, and Figure 1/captions

### paper_write Rules

- Do NOT generate author names, emails, or affiliations — use anonymous block or placeholder
- Write complete sections, not outlines — the output should be compilable LaTeX
- One file per section — modular structure for easy editing
- Every claim must cite evidence — cross-reference the Claims-Evidence Matrix
- Compile-ready — the output should compile with `latexmk` without errors (modulo missing figures)
- No over-claiming — use hedging language ("suggests", "indicates") for weak evidence
- Page limit = main body to Conclusion — references and appendix do NOT count
- Clean bib — references.bib must only contain entries that are actually `\cite`d
- Section count is flexible — match PAPER_PLAN structure, don't force into 5 sections
- Backup before overwrite — never destroy existing `paper/` directory without backing up
- Front-load the contribution — do not hide the payoff until the experiments or appendix

---

