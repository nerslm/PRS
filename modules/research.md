# Research Context Module — Literature Survey, Idea Generation, Novelty Verification

This module covers the `lit_survey` and `idea_generation` phases. It is loaded by `/prs` when `current_stage` is `idea`.

---

## Inputs

- `project.json` — direction, venue, config (pilot_max_hours, arxiv_download, etc.)
- Local paper library: `papers/` or `literature/` in project directory (optional)
- Zotero MCP (optional), Obsidian MCP (optional)

## Outputs

- **lit_survey phase**: Literature survey notes (saved to `notes/lit_survey.md` or project notes)
- **idea_generation phase**: `IDEA_REPORT.md` — ranked, validated, pilot-tested ideas

---

## Phase: lit_survey

### Data Sources (in priority order)

All sources are optional — if not configured, skip silently.

| Priority | Source | Detection | What it provides |
|----------|--------|-----------|-----------------|
| 1 | Zotero (via MCP) | Try `mcp__zotero__*` — skip if unavailable | Collections, annotations, PDF highlights, BibTeX |
| 2 | Obsidian (via MCP) | Try `mcp__obsidian-vault__*` — skip if unavailable | Research notes, processed understanding, wikilinks |
| 3 | Local PDFs | `Glob: papers/**/*.pdf, literature/**/*.pdf` | Raw PDF content (first 3 pages each) |
| 4 | Web search | Always available (WebSearch, WebFetch) | arXiv, Semantic Scholar, Google Scholar |

**Graceful degradation**: If no MCP servers configured, use local PDFs + web search only.

### Local Paper Scanning

- Check PAPER_LIBRARY paths for PDFs
- De-duplicate against Zotero results if available
- Filter by filename/first-page relevance to topic
- Read first 3 pages of up to 20 relevant papers
- Extract: title, authors, year, core contribution, relevance

### Zotero Integration (if available)

- Search by topic, read collections, extract PDF annotations/highlights
- Export BibTeX for relevant papers (reusable in paper writing)
- Zotero annotations are high-value — they show what the user personally highlighted

### Obsidian Integration (if available)

- Search vault for topic-relevant notes
- Check tags (#topic), follow wikilinks
- Extract user's own summaries and insights — more valuable than raw paper content

### Web Search Strategy

**Note on parallelism**: If you are a sub-agent, you cannot spawn your own sub-agents. In that case, search sub-directions sequentially. If you are the main agent doing lit_survey directly, you CAN spawn parallel search agents — one per sub-direction.

1. **Decompose** research topic into all relevant sub-directions (don't limit count)
2. **Parallel search** (if you can spawn agents) or **sequential search** (if you're already a sub-agent):
   Per sub-direction:
   - WebSearch with 3+ query formulations (arXiv, Google Scholar, Semantic Scholar)
   - arXiv API if available: `python3 arxiv_fetch.py search "QUERY" --max 10`
   - Read top 5-10 abstracts per sub-direction
   - Return structured list: title, authors, year, venue, contribution, relevance
3. **Merge**: De-duplicate across agents/searches and local sources, tag by sub-direction, rank by relevance
4. **Optional PDF download** (when `arxiv_download=true` in project.json): Download top 3-5 arXiv PDFs

### Analysis

For each relevant paper extract:
- **Problem**: What gap does it address?
- **Method**: Core technical contribution (1-2 sentences)
- **Results**: Key numbers/claims
- **Relevance**: How does it relate to our direction?
- **Source**: Zotero/Obsidian/local/web

### Synthesis

- Group papers by sub-direction and approach
- Identify consensus vs disagreements
- Find cross-cutting gaps visible only when multiple sub-directions are combined
- Incorporate user's own insights from Obsidian notes if available
- Identify structural gaps: methods from domain A untried in domain B, contradictory findings, untested assumptions, unexplored scaling regimes

### Output Format

Structured literature table:
```
| Paper | Venue | Method | Key Result | Relevance to Us | Source |
|-------|-------|--------|------------|-----------------|--------|
```
Plus 3-5 paragraph narrative summary of the landscape. Include BibTeX snippet from Zotero if available.

---

## Phase: idea_generation

### Brainstorming via External LLM

Use Codex MCP with xhigh reasoning to generate ideas:

```
mcp__codex__codex:
  model: [reviewer_model from project.json]
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    You are a senior researcher brainstorming research ideas for a [VENUE] paper.

    Research direction: [direction from project.json]

    Here is the current landscape:
    [paste landscape summary from lit_survey]

    Key gaps identified:
    [paste gaps]

    Generate 8-12 concrete research ideas. For each:
    1. One-sentence summary
    2. Core hypothesis (what you expect to find and why)
    3. Minimum viable experiment (cheapest way to test)
    4. Expected contribution type: empirical / method / theory / diagnostic
    5. Risk level: LOW / MEDIUM / HIGH
    6. Estimated effort: days / weeks / months

    Prioritize ideas that are:
    - Testable with moderate compute
    - Produce clear positive OR negative result (both publishable)
    - Not "apply X to Y" unless it reveals surprising insights
    - Differentiated from the papers above
```

Save the threadId for follow-up conversations.

### First-Pass Filtering

For each idea, evaluate:
1. **Feasibility**: Can we run this with available resources? Estimate GPU-hours, check data availability. Skip ideas > 1 week GPU time or requiring unavailable datasets.
2. **Novelty quick-check**: 2-3 targeted searches per idea. Full novelty check comes later.
3. **Impact**: "So what?" test — if it succeeds, does it change how people think?

Typically 8-12 → 4-6 survivors.

### Deep Validation (top ideas)

For each surviving idea:

1. **Multi-source novelty verification**:
   - Extract 3-5 core technical claims per idea
   - For EACH claim: WebSearch with 3+ query formulations, check 2024-2026 literature
   - Cross-verify via Codex MCP (xhigh):
     ```
     config: {"model_reasoning_effort": "xhigh"}
     prompt: "[method description] + [all overlapping papers found]. Is this novel? Closest prior work? Delta?"
     ```
   - Produce per-claim novelty score and overall assessment

#### Novelty Report Format (per idea)

```markdown
## Novelty Check Report

### Proposed Method
[1-2 sentence description]

### Core Claims
1. [Claim 1] — Novelty: HIGH/MEDIUM/LOW — Closest: [paper]
2. [Claim 2] — Novelty: HIGH/MEDIUM/LOW — Closest: [paper]
...

### Closest Prior Work
| Paper | Year | Venue | Overlap | Key Difference |
|-------|------|-------|---------|----------------|

### Overall Novelty Assessment
- Score: X/10
- Recommendation: PROCEED / PROCEED WITH CAUTION / ABANDON
- Key differentiator: [what makes this unique, if anything]
- Risk: [what a reviewer would cite as prior work]

### Suggested Positioning
[How to frame the contribution to maximize novelty perception]
```

2. **Devil's advocate review** via Codex MCP (same thread):
   ```
   mcp__codex__codex-reply:
     threadId: [saved from brainstorm]
     config: {"model_reasoning_effort": "xhigh"}
     prompt: |
       Top ideas after filtering: [list with novelty results]
       For each, play devil's advocate:
       - Strongest reviewer objection?
       - Most likely failure mode?
       - Rank for [VENUE] submission?
       - Which 2-3 would you actually work on?
   ```

3. Merge rankings: combine own assessment + GPT review. Select top 2-3 for pilots.

### Pilot Experiments (if GPU available)

Run cheap pilot experiments for empirical signal before committing:

**Constraints** (from project.json config):
- `pilot_max_hours`: max hours per pilot per GPU (default: 2)
- Hard timeout: 3 hours, kill and collect partial results
- Max 3 ideas piloted in parallel
- Total GPU budget: 8 hours across all pilots

**Design**: For each top idea, define minimum viable experiment:
- Single seed, small scale (dataset subset, fewer epochs)
- Clear success metric defined upfront (e.g., "> 1% improvement = positive signal")

**Deploy in parallel**: Launch on different GPUs simultaneously using screen/tmux. Use `run_in_background: true`.

**Re-rank**: Update idea ranking based on empirical evidence. Positive pilot signal outranks theoretical appeal.

Skip pilots if ideas are purely theoretical or no GPU available. Flag as "needs pilot validation".

### IDEA_REPORT.md Format

```markdown
# Research Idea Report

**Direction**: [direction]
**Generated**: [date]
**Ideas evaluated**: X generated → Y survived filtering → Z piloted → W recommended

## Landscape Summary
[3-5 paragraphs]

## Recommended Ideas (ranked)

### Idea 1: [title]
- **Hypothesis**: [one sentence]
- **Minimum experiment**: [concrete description]
- **Expected outcome**: [success/failure criteria]
- **Novelty**: X/10 — closest: [paper], delta: [difference]
- **Feasibility**: [compute, data, implementation estimates]
- **Risk**: LOW/MEDIUM/HIGH
- **Contribution type**: empirical / method / theory / diagnostic
- **Pilot result**: POSITIVE: +X% / NEGATIVE: no signal / SKIPPED
- **Reviewer's likely objection**: [strongest counterargument]
- **Why we should do this**: [1-2 sentences]

## Eliminated Ideas
| Idea | Reason eliminated |
|------|-------------------|

## Pilot Experiment Results
| Idea | GPU | Time | Key Metric | Signal |
|------|-----|------|------------|--------|

## Suggested Execution Order
1. [top idea]
2. [backup]
```

---

## Quality Constraints

- **Direction specificity**: If user's direction is too broad (e.g., "NLP", "computer vision"), STOP and ask to narrow. A good direction is 1-2 sentences specifying problem, domain, and constraint.
- **"Apply X to Y" is the lowest form of research idea.** Push for deeper questions.
- **Empirical signal > theoretical appeal.** An idea with positive pilot outranks untested ideas.
- **A good negative result is publishable.** Prioritize ideas where the answer matters regardless of direction.
- **Kill ideas early and often.** Better to kill 10 ideas in filtering than implement one and fail.
- **Always estimate compute cost.** Ideas needing 1000 GPU-hours are not actionable.
- **Include eliminated ideas in the report** — they save future time by documenting dead ends.
- **Never fail because MCP is unavailable** — fall back gracefully to next data source.
- **Always include paper citations** (authors, year, venue). Distinguish peer-reviewed from preprints.
- **Be brutally honest about novelty** — false claims waste months of research time.
- **Check last 6 months of arXiv** — the field moves fast.

---

## Verification Criteria

- **lit_survey → idea_generation**: Literature survey notes exist with substantive content (tables, narrative)
- **idea_generation → refine**: IDEA_REPORT.md exists with at least one ranked idea that has:
  - Novelty assessment (score + closest work)
  - Feasibility estimate
  - Either pilot result or explicit "needs pilot" flag
