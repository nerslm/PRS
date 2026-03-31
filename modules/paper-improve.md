# Paper Sub-module

> Also read `modules/paper-common.md` for shared inputs/outputs, quality constraints, and verification criteria.

## Sub-phase: paper_improve

### Context

This phase runs **after** the paper_plan -> paper_figures -> paper_write -> paper_compile pipeline. It takes a compiled paper and iteratively improves it through external LLM review.

Unlike the review_loop (which iterates on **research** — running experiments, collecting data, rewriting narrative), this phase iterates on **paper writing quality** — fixing theoretical inconsistencies, softening overclaims, adding missing content, and improving presentation.

### State Persistence (Compact Recovery)

If the context window fills up mid-loop, Claude Code auto-compacts. To recover, this phase writes `PAPER_IMPROVEMENT_STATE.json` after each round:

```json
{
  "current_round": 1,
  "threadId": "019ce736-...",
  "last_score": 6,
  "status": "in_progress",
  "timestamp": "2026-03-13T21:00:00"
}
```

**On startup**: if `PAPER_IMPROVEMENT_STATE.json` exists with `"status": "in_progress"` AND `timestamp` is within 24 hours, read it + `PAPER_IMPROVEMENT_LOG.md` to recover context, then resume from the next round. Otherwise start fresh.

**After each round**: overwrite the state file. **On completion**: set `"status": "completed"`.

### Paper Improvement Loop Structure

**Step 0: Preserve Original**
```bash
cp paper/main.pdf paper/main_round0_original.pdf
```

**Step 1: Collect Paper Text**
```bash
for f in paper/sections/*.tex; do
    echo "% === $(basename $f) ==="
    cat "$f"
done > /tmp/paper_full_text.txt
```

**Step 2: Round 1 Review.** Before composing the prompt, read `references/venues/{venue_slug}.md` and use its **Prompt Fragment** section as the reviewer instructions. If the file does not exist, fall back to `references/venues/neurips.md`.

Send the full paper text to the reviewer model:

```
mcp__codex__codex:
  model: [reviewer_model from project.json]
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    You are reviewing a [VENUE] paper. Please provide a detailed, structured review.

    ## Full Paper Text:
    [paste concatenated sections]

    ## Review Instructions
    [INSERT the Prompt Fragment from the venue profile file here]
```

**Save the threadId** for subsequent rounds.

**Round 2+ Review** — use `mcp__codex__codex-reply` with the saved threadId:

```
mcp__codex__codex-reply:
  threadId: [saved from Round 1]
  model: [reviewer_model from project.json]
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [Round N update]

    Since your last review, we have implemented:
    1. [Fix 1]: [description]
    2. [Fix 2]: [description]
    ...

    Please re-score and re-assess. Same format:
    Score, Summary, Strengths, Weaknesses, Actionable fixes, Verdict.
```

**Loop control**: Continue until score >= `score_threshold_review` OR `max_paper_improve_rounds` exhausted. Only the numeric score decides — ignore verdict text.

**Step 2b: Checkpoint (if autonomy requires it).** When `autonomy` is `gates` or `manual`, pause after each round. When `full`, run autonomously.

**Step 3: Implement Fixes.** Parse the review and implement fixes by severity:

Priority order:
1. CRITICAL fixes (assumption mismatches, internal contradictions)
2. MAJOR fixes (overclaims, missing content, notation issues)
3. MINOR fixes (if time permits)

**Common fix patterns:**

| Issue | Fix Pattern |
|-------|-------------|
| Assumption-model mismatch | Rewrite assumption to match the model, add formal proposition bridging the gap |
| Overclaims | Soften language: "validate" -> "demonstrate practical relevance", "comparable" -> "qualitatively competitive" |
| Missing metrics | Add quantitative table with honest parameter counts and caveats |
| Theorem not self-contained | Add "Interpretation" paragraph listing all dependencies |
| Notation confusion | Rename conflicting symbols globally, add Notation paragraph |
| Missing references | Add to `references.bib`, cite in appropriate locations |
| Theory-practice gap | Explicitly frame theory as idealized; add synthetic validation subsection |

**Step 4: Recompile.** After each round of fixes:
```bash
cd paper && latexmk -C && latexmk -pdf -interaction=nonstopmode -halt-on-error main.tex
cp main.pdf main_roundN.pdf
```
Verify: 0 undefined references, 0 undefined citations.

**Step 5: Format Check (after final round).** Run format compliance check:

```bash
# 1. Page count vs venue limit
PAGES=$(pdfinfo paper/main.pdf | grep Pages | awk '{print $2}')
echo "Pages: $PAGES (limit: MAX_PAGES for venue)"

# 2. Overfull hbox warnings
OVERFULL=$(grep -c "Overfull" paper/main.log 2>/dev/null || echo 0)
echo "Overfull hbox warnings: $OVERFULL"
grep "Overfull" paper/main.log 2>/dev/null | head -10

# 3. Underfull hbox warnings
UNDERFULL=$(grep -c "Underfull" paper/main.log 2>/dev/null || echo 0)
echo "Underfull hbox warnings: $UNDERFULL"

# 4. Bad boxes summary
grep -c "badness" paper/main.log 2>/dev/null || echo "0 badness warnings"
```

**Format auto-fix patterns:**

| Issue | Fix |
|-------|-----|
| Overfull hbox in equation | Wrap in `\resizebox` or split with `\split`/`aligned` |
| Overfull hbox in table | Reduce font (`\small`/`\footnotesize`) or use `\resizebox{\linewidth}{!}{...}` |
| Overfull hbox in text | Rephrase sentence or add `\allowbreak` / `\-` hints |
| Over page limit | Move content to appendix, compress tables, reduce figure sizes |
| Underfull hbox (loose) | Rephrase for better line filling or add `\looseness=-1` |

If any overfull hbox > 10pt is found, fix it and recompile before documenting.

### PAPER_IMPROVEMENT_LOG.md Format

```markdown
# Paper Improvement Log

## Score Progression

| Round | Score | Verdict | Key Changes |
|-------|-------|---------|-------------|
| Round 0 (original) | X/10 | No/Almost/Yes | Baseline |
| Round 1 | Y/10 | No/Almost/Yes | [summary of fixes] |
| Round 2 | Z/10 | No/Almost/Yes | [summary of fixes] |

## Round 1 Review & Fixes

<details>
<summary>GPT-5.4 xhigh Review (Round 1)</summary>

[Full raw review text, verbatim]

</details>

### Fixes Implemented
1. [Fix description]
2. [Fix description]
...

## Round 2 Review & Fixes

<details>
<summary>GPT-5.4 xhigh Review (Round 2)</summary>

[Full raw review text, verbatim]

</details>

### Fixes Implemented
1. [Fix description]
2. [Fix description]
...

## PDFs
- `main_round0_original.pdf` — Original generated paper
- `main_round1.pdf` — After Round 1 fixes
- `main_round2.pdf` — Final version after Round 2 fixes
```

### Typical Score Progression

Based on end-to-end testing on a 9-page ICLR 2026 theory paper:

| Round | Score | Key Improvements |
|-------|-------|-----------------|
| Round 0 | 4/10 (content) | Baseline: assumption-model mismatch, overclaims, notation issues |
| Round 1 | 6/10 (content) | Fixed assumptions, softened claims, added interpretation, renamed notation |
| Round 2 | 7/10 (content) | Added synthetic validation, formal truncation proposition, stronger limitations |
| Round 3 | 5->8.5/10 (format) | Removed hero fig, appendix, compressed conclusion, fixed overfull hbox |

+4.5 points across 3 rounds (2 content + 1 format) is typical for a well-structured but rough first draft.

### paper_improve Rules

- Preserve all PDF versions — user needs to compare progression
- Save FULL raw review text — do not summarize or truncate GPT-5.4 responses
- Use `mcp__codex__codex-reply` for Round 2+ to maintain conversation context
- Always recompile after fixes — verify 0 errors before proceeding
- Do not fabricate experimental results — synthetic validation must describe methodology, not invent numbers
- Respect the paper's claims — soften overclaims rather than adding unsupported new claims
- Global consistency — when renaming notation or softening claims, check ALL files (abstract, intro, method, experiments, theory sections, conclusion, tables, figure captions)
- **Anti-hallucination citations** — when adding references during fixes, use the DBLP → CrossRef → `[VERIFY]` chain from `modules/paper-write.md`. Never fabricate BibTeX.
- **Max improvement rounds** — from `project.json` config `max_paper_improve_rounds` (default: 2)

### Feishu Notification (if configured)

After each round, check if `~/.claude/feishu.json` exists and mode is not `"off"`:
- Send `review_scored` notification: "Paper Round N: X/10 — [verdict]"
- After final round, send `pipeline_done` with score progression table
- If config absent or mode off: skip silently

---

