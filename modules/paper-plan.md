# Paper Sub-module

> Also read `modules/paper-common.md` for shared inputs/outputs, quality constraints, and verification criteria.

## Sub-phase: paper_plan

### Step 1: Extract Claims and Evidence

Read all available narrative documents and extract:

1. **Core claims** (3-5 main contributions)
2. **One-sentence contribution** (the single sentence that best states what the paper contributes)
3. **Evidence** for each claim (which experiments, which metrics, which figures)
4. **Known weaknesses** (from reviewer feedback)
5. **Suggested framing** (from review conclusions)

Build a **Claims-Evidence Matrix**:

```markdown
| Claim | Evidence | Status | Section |
|-------|----------|--------|---------|
| [claim 1] | [exp A, metric B] | Supported | §3.2 |
| [claim 2] | [exp C] | Partially supported | §4.1 |
```

The Claims-Evidence Matrix is the backbone of the entire paper — every claim must map to evidence, every experiment must support a claim.

### Step 2: Determine Paper Type and Structure

Based on venue (from `project.json`) and paper content, classify and select structure.

Before committing to a structure, apply narrative principles from `references/writing-principles.md`:

- The paper should tell one coherent technical story.
- By the end of the Introduction, the outline should make the **What**, **Why**, and **So What** explicit.
- Front-load the most important material: title, abstract, introduction, and hero figure. Reviewers often form a judgment before reading the full method.

**IMPORTANT**: Section count is FLEXIBLE (5-8 sections). Choose what fits the content best. Templates below are starting points, not rigid constraints.

**Empirical/Diagnostic paper:**
```
1. Introduction (1.5 pages)
2. Related Work (1 page)
3. Method / Setup (1.5 pages)
4. Experiments (3 pages)
5. Analysis / Discussion (1 page)
6. Conclusion (0.5 pages)
```

**Theory + Experiments paper:**
```
1. Introduction (1.5 pages)
2. Related Work (1 page)
3. Preliminaries & Modeling (1.5 pages)
4. Experiments (1.5 pages)
5. Theory Part A (1.5 pages)
6. Theory Part B (1.5 pages)
7. Conclusion (0.5 pages)
— Total: 9 pages
```
Theory papers often need 7 sections (splitting theory into estimation + optimization, or setup + analysis). The total page budget MUST sum to MAX_PAGES (from `project.json`).

Theory papers should:
- Include **proof sketch** locations (not just theorem statements)
- Plan a **comparison table** of prior theoretical bounds vs. this paper's bounds
- Identify which proofs go in appendix vs. main body

**Method paper:**
```
1. Introduction (1.5 pages)
2. Related Work (1 page)
3. Method (2 pages)
4. Experiments (2.5 pages)
5. Ablation / Analysis (1 page)
6. Conclusion (0.5 pages)
```

### Step 3: Section-by-Section Planning

For each section, specify:

```markdown
### §0 Abstract
- **What we achieve**: [the paper's specific contribution, not field-level background]
- **Why it matters / is hard**: [why this problem is important and non-trivial]
- **How we do it**: [approach in one sentence]
- **Evidence**: [what supports the claim]
- **Most remarkable result**: [strongest quantitative or theoretical result]
- **Estimated length**: 150-250 words
- **Self-contained check**: can a reader understand this without the paper?

### §1 Introduction
- **Opening hook**: [1-2 sentences that motivate the problem]
- **Gap / challenge**: [what's missing in prior work, and why prior work is insufficient]
- **One-sentence contribution**: [the main takeaway of the paper]
- **Approach overview**: [what we do differently]
- **Key questions**: [the research questions this paper answers]
- **Contributions**: [2-4 numbered bullets, specific and falsifiable, matching Claims-Evidence Matrix]
- **Results preview**: [the strongest result or comparison to surface early]
- **Hero figure**: [describe what Figure 1 should show — MUST include clear comparison if applicable]
- **Estimated length**: 1.5 pages
- **Key citations**: [3-5 papers to cite here]
- **Front-loading check**: [would a skim reader know the main claim before reaching the method?]

### §2 Related Work
- **Subtopics**: [2-4 categories of related work]
- **Positioning**: [how this paper differs from each category]
- **Minimum length**: 1 full page (at least 3-4 paragraphs with substantive synthesis)
- **Organization rule**: organize by methodological family / assumption / question, not paper-by-paper
- **Must NOT be just a list** — synthesize, compare, and position

### §3 Method / Setup / Preliminaries
- **Notation**: [key symbols and their meanings]
- **Problem formulation**: [formal setup]
- **Method description**: [algorithm, model, or experimental design]
- **Formal statements**: [theorems, propositions if applicable]
- **Proof sketch locations**: [which key steps appear here vs. appendix]
- **Estimated length**: 1.5-2 pages

### §4 Experiments / Main Results
- **Figures planned**:
  - Fig 1: [description, type: bar/line/table/architecture, WHAT COMPARISON it shows]
  - Fig 2: [description]
  - Table 1: [what it shows, which methods/baselines compared]
- **Data source**: [which JSON files / experiment results]

### §5 Conclusion
- **Restatement**: [contributions rephrased, not copy-pasted from intro]
- **Limitations**: [honest assessment — reviewers value this]
- **Future work**: [1-2 concrete directions]
- **Estimated length**: 0.5 pages
```

### Step 4: Figure Plan

List every figure and table:

```markdown
## Figure Plan

| ID | Type | Description | Data Source | Priority |
|----|------|-------------|-------------|----------|
| Fig 1 | Hero/Architecture | System overview + comparison | manual | HIGH |
| Fig 2 | Line plot | Training curves comparison | figures/exp_A.json | HIGH |
| Fig 3 | Bar chart | Ablation results | figures/ablation.json | MEDIUM |
| Table 1 | Comparison table | Main results vs. baselines | figures/main_results.json | HIGH |
| Table 2 | Theory comparison | Prior bounds vs. ours | manual | HIGH (theory papers) |
```

**CRITICAL for Figure 1 / Hero Figure**: Describe in detail what the figure should contain, including:
- Which methods are being compared
- What the visual difference should demonstrate
- Caption draft that clearly states the comparison
- Why the figure helps a skim reader understand the paper before reading the full method

### Step 5: Citation Scaffolding

For each section, list required citations:

```markdown
## Citation Plan
- §1 Intro: [paper1], [paper2], [paper3] (problem motivation)
- §2 Related: [paper4]-[paper10] (categorized by subtopic)
- §3 Method: [paper11] (baseline), [paper12] (technique we build on)
```

**Citation rules:**
1. NEVER generate BibTeX from memory — always verify via search or existing .bib files
2. Every citation must be verified: correct authors, year, venue
3. Flag any citation you're unsure about with `[VERIFY]`
4. Prefer published versions over arXiv preprints when available

### Step 6: Cross-Review with Reviewer Model

Send the complete outline to GPT-5.4 xhigh for feedback. Score 1-10 on:

1. Logical flow — does the story build naturally?
2. Claim-evidence alignment — every claim backed?
3. Missing experiments or analysis
4. Positioning relative to prior work
5. Page budget feasibility (MAX_PAGES = main body to Conclusion end, excluding refs/appendix)
6. Front-matter strength — are the abstract, introduction, and hero figure plan strong enough for skim-reading reviewers?

For each weakness, require the MINIMUM fix. Be specific and actionable — "add X" not "consider more experiments".

Apply feedback before finalizing.

### Step 7: PAPER_PLAN.md Output Format

Save the final outline to `PAPER_PLAN.md` in the project root:

```markdown
# Paper Plan

**Title**: [working title]
**One-sentence contribution**: [single-sentence statement of the paper's core takeaway]
**Venue**: [target venue]
**Type**: [empirical/theory/method]
**Date**: [today]
**Page budget**: [MAX_PAGES] pages (main body to Conclusion end, excluding references & appendix)
**Section count**: [N] (must match the number of section files that will be created)

## Claims-Evidence Matrix
[from Step 1]

## Structure
[from Step 2-3, section by section]

## Figure Plan
[from Step 4, with detailed hero figure description]

## Citation Plan
[from Step 5]

## Reviewer Feedback
[from Step 6, summarized]

## Next Steps
- [ ] paper_figures to generate all figures
- [ ] paper_write to draft LaTeX
- [ ] paper_compile to build PDF
```

### paper_plan Rules

- Do NOT generate author information — leave author block as placeholder or anonymous
- Be honest about evidence gaps — mark claims as "needs experiment" rather than overclaiming
- Page budget is hard — if content exceeds MAX_PAGES, suggest what to move to appendix
- MAX_PAGES counts main body only — from first page to end of Conclusion. References and appendix are NOT counted.
- Venue-specific norms — ICLR/NeurIPS/ICML use `natbib` (`\citep`/`\citet`)
- Front-load the story — the outline should make the contribution clear in the title, abstract, introduction, and hero figure before the reader reaches the full method
- Figures need detailed descriptions — especially the hero figure, which must clearly specify comparisons and visual expectations
- Section count is flexible — 5-8 sections depending on paper type. Don't force content into a rigid 5-section template.

---

