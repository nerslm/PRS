# Refine Context Module — Problem-Anchored Method Refinement

This module covers the `refine` phase. It is loaded by `/prs` when `current_stage` is `idea` and literature survey + idea generation are done.

The goal: turn a vague direction into a **problem → concrete approach → validation plan** document that is complete enough to implement, logically sound, and grounded in first principles.

---

## Inputs

- `IDEA_REPORT.md` — selected idea from idea_generation phase
- `project.json` — venue, reviewer_model, max_refine_rounds, score_threshold_refine
- Venue profile: `references/venues/{venue}.md` (Refine Persona section)
- Local papers in `papers/` or `literature/` (for grounding material)

## Outputs

- `refine-logs/round-0-initial-proposal.md` — initial proposal
- `refine-logs/round-N-review.md` — raw review for each round
- `refine-logs/round-N-refinement.md` — full revised proposal for each round
- `refine-logs/score-history.md` — score evolution table
- `refine-logs/REVIEW_SUMMARY.md` — round-by-round resolution log
- `refine-logs/FINAL_PROPOSAL.md` — clean final version
- `refine-logs/REFINEMENT_REPORT.md` — full report with raw reviews

---

## Four Governing Principles

1. **Do not lose the original problem.** Freeze an immutable **Problem Anchor** and reuse it verbatim in every round.
2. **The right scope is whatever the problem demands.** A method paper needs focused novelty; a benchmark needs comprehensive coverage; a theory paper needs rigor. Match scope to research goal, not a template.
3. **Every element must serve the research goal.** Nothing unjustified, nothing missing that's needed.
4. **Ground in first principles.** The approach should follow from fundamental understanding of the problem, not from trends or conventions.

---

## Phase 0: Freeze the Problem Anchor

Before proposing anything, extract the user's immutable bottom-line problem. This anchor must be copied verbatim into every proposal and every refinement round.

Write:
- **Bottom-line problem**: What technical problem must be solved?
- **Must-solve bottleneck**: What specific weakness in current methods is unacceptable?
- **Non-goals**: What is explicitly *not* the goal of this project?
- **Constraints**: Compute, data, time, tooling, venue, deployment limits.
- **Success condition**: What evidence would make the user say "yes, this method addresses the actual problem"?

If later reviewer feedback would change the problem being solved, mark that as **drift** and push back or adapt carefully.

---

## Phase 1: Build the Initial Proposal

### Step 1.1: Understand the Research Landscape

Read existing grounding material (`papers/`, `literature/`, `IDEA_REPORT.md`). Answer:
- What is the current state of knowledge on this problem?
- Where are the gaps — what remains unsolved, untested, or poorly understood?
- What approaches have been tried, and what are their limitations?
- What resources (data, compute, tools, existing code) are available?

If local material is insufficient, search recent literature online. Focus on understanding the **structure of the problem**, not just cataloging prior work.

### Step 1.2: Sharpen the Research Question

Make the question precise and falsifiable:
1. **The gap**: What specific knowledge or capability is missing?
2. **Why it persists**: Why haven't existing approaches closed this gap? What is the structural reason, not just "nobody tried"?
3. **The hypothesis**: What do you believe to be true that, if validated, would close the gap?
4. **Required evidence**: What would constitute convincing support — and what would falsify it?

### Step 1.3: Design the Approach

The approach must answer "how will we answer this research question?" Think from first principles — what does the problem structure demand?

1. **One-sentence thesis**: The single strongest claim this research will make.
2. **Approach overview**: How the question will be answered — algorithm, study design, theoretical framework, evaluation methodology, or any combination.
3. **Key design decisions**: For each major choice, state what was chosen, what alternatives exist, and why this choice follows from the problem (not from convention or trend).
4. **Scope**: What is in scope, what is explicitly out, and why these boundaries are the right ones for this research goal.
5. **Expected contributions**: What the field gains if this succeeds — new knowledge, new tools, new understanding, or new evidence.
6. **Comparison to alternatives**: Why this approach over the obvious alternatives.
7. **Failure modes**: What could go wrong, how to detect it, and what the fallback is.

### Step 1.4: Plan Validation

For each core claim, define how it will be validated:
- The claim being tested
- The evidence needed (experiment, proof, analysis, comparison, coverage)
- The success criterion — what outcome would be convincing
- What a negative result would mean — is it still publishable?

The validation plan should match the research type: experiments for empirical claims, proofs for theoretical claims, comprehensive coverage for benchmark claims, statistical rigor for observational claims.

### Step 1.5: Write the Initial Proposal

Save to `refine-logs/round-0-initial-proposal.md` using the proposal template below. The template is intentionally abstract — adapt the sections to fit your research type.

```markdown
# Research Proposal: [Title]

## Problem Anchor
- Research question:
- Core gap in current knowledge:
- Why this matters (significance):
- Non-goals:
- Constraints (resources, time, data, venue):
- Success condition:

## Research Context
[Current state of knowledge. Key prior work and their limitations. Why the gap persists — the structural reason, not just "nobody tried." What we can build on.]

## Thesis
- One-sentence thesis:
- First-principles argument: why this approach follows from the structure of the problem, not from convention or trend.

## Contributions
- Primary contribution:
- Supporting contributions (if any):
- Explicit non-goals:

## Approach
### Overview
[High-level description of how the research question will be answered. This could be a method, an evaluation framework, a theoretical analysis, an empirical study, or a combination.]

### Key Design Decisions
[For each major decision: what was chosen, what alternatives exist, why this choice follows from the problem structure. Ground every decision in first principles.]

### Scope and Boundaries
- In scope:
- Out of scope:
- Why these boundaries serve the research goal:

## Validation Plan
### Claim 1: [claim]
- Evidence needed:
- Validation approach:
- Success criterion:
- What a negative result means:

### Claim 2: [if applicable]
- Evidence needed:
- Validation approach:
- Success criterion:
- What a negative result means:

## Feasibility
- Required resources (compute, data, tools):
- Timeline estimate:
- Highest-risk assumptions:
- Fallback if key assumption fails:

## Experiment Handoff
- Claims that must be validated:
- Critical datasets / benchmarks / metrics:
- Key baselines or comparisons:
- Highest-risk assumptions to test first:
```

---

## Phase 2: External Method Review (Iterative)

### Review Prompt Template (Round 1)

**Before composing**: read `references/venues/{venue}.md` and use its **Refine Persona** section. Fall back to `references/venues/neurips.md` if not found.

```
mcp__codex__codex:
  model: [reviewer_model from project.json]
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [INSERT Refine Persona from venue profile]

    You are reviewing an early-stage research proposal. Your goal is to help improve it — not to impose a particular paper format.

    Evaluate whether the proposal:
    (1) still solves the original anchored problem,
    (2) has considered all necessary angles — no blind spots in approach, theory, or methodology,
    (3) is logically sound with no gaps or unjustified assumptions,
    (4) is feasible — can actually produce concrete results with the stated resources,
    (5) is grounded in first principles, not conventions or trends.

    Review principles:
    - Start from the Problem Anchor. Every suggestion must serve the original research goal.
    - The right scope is whatever the problem demands — don't push to cut if comprehensive coverage is the point, don't push to expand if tight focus is the point.
    - Check for logical holes: unjustified assumptions, missing edge cases, reasoning gaps, theory-practice mismatches.
    - If something important is missing (methods, baselines, perspectives, edge cases), say what and why.
    - If something doesn't serve the research goal, explain why it should be removed.
    - If your suggestion would change the problem being solved, flag it as drift.

    === PROPOSAL ===
    [Paste FULL proposal]
    === END PROPOSAL ===

    Score these 6 dimensions from 1-10:

    1. **Significance** (30%): Is this worth doing? Would the results (positive or negative) advance understanding in the field?

    2. **Feasibility** (20%): Is the proposal concrete enough to implement? Can it actually produce results with the stated resources and timeline?

    3. **First Principles** (20%): Is the approach grounded in fundamental understanding of the problem, or just following conventions / trends / surface-level patterns?

    4. **Problem Fidelity** (10%): Does the proposal still attack the original problem, or has it drifted?

    5. **Completeness** (10%): Has the approach considered all necessary angles? Missing important methods, baselines, theoretical perspectives, or edge cases? Is the scope appropriate for the research goal?

    6. **Logical Soundness** (10%): Is the reasoning chain airtight? Are assumptions justified? Any logical gaps between problem, approach, and expected evidence?

    **OVERALL SCORE** (1-10): Use this weighting: Significance 30%, Feasibility 20%, First Principles 20%, Problem Fidelity 10%, Completeness 10%, Logical Soundness 10%.

    For each dimension scoring < 7, provide:
    - The specific weakness
    - A concrete, actionable fix
    - Priority: CRITICAL / IMPORTANT / MINOR

    Then add:
    - **Missing Elements**: What the proposal should add — methods, perspectives, analyses, or edge cases it hasn't considered. Write "NONE" if complete.
    - **Unnecessary Elements**: What could be removed because it doesn't serve the research goal. Write "NONE" if everything is justified.
    - **Drift Warning**: "NONE" if the proposal still solves the anchored problem; otherwise explain the drift.
    - **Verdict**: READY / REVISE / RETHINK

    Verdict rule:
    - READY: overall score >= 9, no meaningful drift, logically sound, scope appropriate for the goal
    - REVISE: the direction is promising but has gaps or weaknesses to address
    - RETHINK: the core approach or framing is fundamentally off
```

**CRITICAL**: Save `threadId` from this call. Save FULL raw response verbatim.

Save to `refine-logs/round-1-review.md`.

### Re-evaluation Prompt (Round 2+)

```
mcp__codex__codex-reply:
  threadId: [saved]
  model: [reviewer_model]
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [Round N re-evaluation]

    I revised the proposal based on your feedback.

    Key changes:
    1. [Change 1]
    2. [Change 2]
    3. [Pushback on specific feedback, if any]

    === REVISED PROPOSAL ===
    [Paste the FULL revised proposal]
    === END REVISED PROPOSAL ===

    Please:
    - Re-score the same 6 dimensions and overall
    - State whether the Problem Anchor is preserved or drifted
    - State whether completeness has improved — are former blind spots now addressed?
    - State whether logical gaps have been closed
    - State whether feasibility concerns are resolved
    - Focus new critiques on remaining logical holes, missing perspectives, unjustified assumptions, or first-principles violations
    - Use the same verdict rule: READY only if overall score >= 9 and no blocking issue remains

    Same output format: 6 scores, overall score, verdict, drift warning, missing elements, unnecessary elements, remaining action items.
```

---

## Phase 3: Parse Feedback and Revise

### Anchor Check (before every revision)

1. Copy Problem Anchor verbatim
2. What is the original bottleneck?
3. Does the current method still solve it?
4. Which reviewer suggestions would cause drift?

### Scope & Logic Check (before every revision)

1. Is the current scope appropriate for the research goal?
2. Is anything important missing that should be added?
3. Is anything included that doesn't serve the goal?
4. Are all assumptions justified from first principles?

### Revision Strategy

- **Valid feedback**: address the gap, fix the logic, improve completeness.
- **Debatable feedback**: revise but explain reasoning with evidence.
- **Wrong/drifting**: push back with evidence from Problem Anchor.

Bias toward: logical completeness, first-principles grounding, appropriate scope for the problem.

**Do NOT change scope to chase score.** If reviewer suggests adding or cutting, first ask whether the change serves the original research goal.

Save to `refine-logs/round-N-refinement.md` — MUST contain the full revised proposal, not just diffs.

### Stop Conditions

**Loop control**: Continue until score >= `score_threshold_refine` OR `max_refine_rounds` exhausted. Only the numeric score decides — ignore verdict text.

---

## Phase 5: Final Report

### 5.1: Write `refine-logs/REVIEW_SUMMARY.md`

High-level round-by-round review record:

```markdown
# Review Summary

**Problem**: [user's problem]
**Initial Approach**: [user's vague approach]
**Date**: [today]
**Rounds**: N / MAX_ROUNDS
**Final Score**: X / 10
**Final Verdict**: [READY / REVISE / RETHINK]

## Problem Anchor
[Verbatim anchor used across all rounds]

## Round-by-Round Resolution Log

| Round | Main Reviewer Concerns | What This Round Changed | Solved? | Remaining Risk |
|-------|-------------------------|------------------------------------------|---------|----------------|
| 1     | [top issues from review] | [main method changes]                    | [yes / partial / no] | [if any] |
| 2     | ...                     | ...                                      | ...     | ...            |

## Overall Evolution
- [How completeness improved — blind spots addressed]
- [How logical soundness improved — gaps closed]
- [How scope was adjusted to match the research goal]
- [How drift was avoided or corrected]

## Final Status
- Anchor status: [preserved / corrected / unresolved]
- Completeness: [comprehensive / has gaps / major blind spots]
- Logical soundness: [airtight / minor gaps / significant holes]
- Strongest parts of final proposal:
- Remaining weaknesses:
```

### 5.2: Write `refine-logs/FINAL_PROPOSAL.md`

Clean final version — only the proposal itself, no review chatter or round history:

```markdown
# Research Proposal: [Title]

[Paste the final refined proposal only]
```

If the final verdict is not READY, still write the best current version here.

### 5.3: Write `refine-logs/REFINEMENT_REPORT.md`

Full report with raw reviews:

```markdown
# Refinement Report

**Problem**: [user's problem]
**Initial Approach**: [user's vague approach]
**Date**: [today]
**Rounds**: N / MAX_ROUNDS
**Final Score**: X / 10
**Final Verdict**: [READY / REVISE / RETHINK]

## Problem Anchor
[Verbatim anchor used across all rounds]

## Output Files
- Review summary: `refine-logs/REVIEW_SUMMARY.md`
- Final proposal: `refine-logs/FINAL_PROPOSAL.md`

## Score Evolution

| Round | Significance | Feasibility | First Principles | Problem Fidelity | Completeness | Logical Soundness | Overall | Verdict |
|-------|--------------|-------------|------------------|------------------|--------------|-------------------|---------|---------|
| 1     | ...              | ...                | ...                  | ...               | ...         | ...              | ...             | ...     | ...     |

## Round-by-Round Review Record

| Round | Main Reviewer Concerns | What Was Changed | Result |
|-------|-------------------------|------------------|--------|
| 1     | [top issues]            | [main fixes]     | [resolved / partial / unresolved] |
| 2     | ...                     | ...              | ...    |

## Final Proposal Snapshot
- Canonical clean version lives in `refine-logs/FINAL_PROPOSAL.md`
- Summarize the final thesis in 3-5 bullets here

## Proposal Evolution Highlights
1. [Most important completeness improvement]
2. [Most important logical gap closed]
3. [Most important scope adjustment]

## Pushback / Drift Log
| Round | Reviewer Said | Author Response | Outcome |
|-------|---------------|-----------------|---------|
| 1     | [criticism]   | [pushback + anchor / evidence] | [accepted / rejected] |

## Remaining Weaknesses
[Honest unresolved issues]

## Raw Reviewer Responses

<details>
<summary>Round 1 Review</summary>

[Full verbatim response from reviewer]

</details>

...

## Next Steps
- If READY: proceed to experiment_plan phase
- If REVISE: manually address remaining mechanism weaknesses, then re-run refine phase
- If RETHINK: revisit the core mechanism, possibly restart from idea_generation
```

### 5.4: Finalize `refine-logs/score-history.md`

Ensure it contains the complete score evolution table using all 6 dimensions.

### 5.5: `refine-logs/round-N-refinement.md` Template

Every round-N-refinement.md must use this template (full anchored proposal, not just diffs):

```markdown
# Round N Refinement

## Problem Anchor
[Copy verbatim from round 0]

## Anchor Check
- Original bottleneck:
- Why the revised method still addresses it:
- Reviewer suggestions rejected as drift:

## Scope & Logic Check
- Is scope appropriate for the research goal after revision?
- What was added to address blind spots?
- What was removed as unjustified?
- Are all assumptions grounded in first principles?

## Changes Made

### 1. [Method section changed]
- Reviewer said:
- Action:
- Reasoning:
- Impact on core method:

### 2. [Novelty / modernity / feasibility / validation change]
- Reviewer said:
- Action:
- Reasoning:
- Impact on core method:

## Revised Proposal
[Full updated proposal from Problem Anchor through Claim-Driven Validation Sketch]
```

### 5.6: Present Summary to User

```
Refinement complete after N rounds.

Final score: X/10 (Verdict: READY / REVISE / RETHINK)

Anchor status:
- [preserved / drift corrected / unresolved concern]

Completeness:
- [comprehensive / gaps addressed / remaining blind spots]

Logical soundness:
- [airtight / minor gaps / significant holes]

Key improvements:
- [improvement 1]
- [improvement 2]

Remaining concerns:
- [if any]

Review summary: refine-logs/REVIEW_SUMMARY.md
Full report: refine-logs/REFINEMENT_REPORT.md
Final proposal: refine-logs/FINAL_PROPOSAL.md
```

Update `progress.json`: set refine phase to done with final score, round count, and verdict.

---

## Quality Constraints

- **Anchor first, every round.** Always carry forward the same Problem Anchor.
- **Scope matches the goal.** Don't cut if coverage is the point; don't expand if focus is the point.
- **Every element earns its place.** Nothing unjustified, nothing critical missing.
- **First principles over conventions.** The approach should follow from understanding, not from "everyone does it this way."
- **Check your logic.** Assumptions justified? Reasoning chain complete? Edge cases considered?
- **Pushback is encouraged.** Argue back with evidence when feedback causes drift or changes the research goal.
- **ALWAYS use `config: {"model_reasoning_effort": "xhigh"}`** for all Codex review calls.
- **Save threadId** and use `mcp__codex__codex-reply` for later rounds.
- **Do not fabricate results.** Only describe expected evidence.
- **Be specific about compute and data assumptions.**
- **Document everything.** Save every raw review, anchor check, simplicity check, and method change.

---

## Verification Criteria

- refine → experiment_plan: `refine-logs/FINAL_PROPOSAL.md` exists with substantive content, score history shows progression, Problem Anchor is preserved in final version.
