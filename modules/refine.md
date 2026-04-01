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

### Step 1.1: Scan Grounding Material

Check `papers/` and `literature/` first. Read only relevant parts to answer:
- What mechanism do current methods use?
- Where exactly do they fail for this problem?
- Which recent LLM / VLM / Diffusion / RL era techniques are actually relevant here?
- What training objectives, representations, or interfaces are reusable?
- What details distinguish a real method from a renamed high-level idea?

If local material is insufficient, search recent top-venue/arXiv work online. Focus on **method sections, training setup, and failure modes**, not just abstracts.

### Step 1.2: Identify the Technical Gap

Make the gap operational:
1. **Current pipeline failure point**: where does the baseline break?
2. **Why naive fixes are insufficient**: larger context, more data, prompting, memory bank, or stacking more modules.
3. **Smallest adequate intervention**: the least additional mechanism that could plausibly fix the bottleneck.
4. **Frontier-native alternative**: a more current route using foundation-model-era primitives that better matches the bottleneck?
5. **Core technical claim**: exact mechanism claim that could survive top-venue scrutiny.
6. **Required evidence**: minimum proof needed to defend that claim.

### Step 1.3: Choose the Sharpest Route

Compare two candidate routes if both are plausible:
- **Route A: Elegant minimal route** — smallest mechanism targeting the bottleneck.
- **Route B: Frontier-native route** — uses LLM / VLM / Diffusion / RL / distillation / inference-time scaling *only if* it gives a cleaner or stronger story.

Decide by: which is more likely to become a strong paper, which has cleaner novelty, which avoids contribution sprawl. If both are weak, rethink the framing.

### Step 1.4: Concretize the Method

The proposal must answer "how would we actually build this?" Cover:

1. **One-sentence method thesis**: the single strongest mechanism claim.
2. **Contribution focus**: one dominant + at most one supporting contribution.
3. **Complexity budget**: what is frozen/reused, what is new, what tempting additions are excluded.
4. **System graph**: modules, data flow, inputs, outputs.
5. **Representation design**: latent, embedding, plan token, reward signal, memory state, alignment space.
6. **Training recipe**: data source, supervision, pseudo-labeling, negatives, curriculum, losses, weighting, stagewise vs joint.
7. **Inference path**: how trained components are used at test time.
8. **Why the mechanism stays small**: why a larger stack is unnecessary.
9. **Exact role of any frontier primitive**: whether it acts as planner, teacher, critic, reward model, generator prior, search controller, or distillation source.
10. **Failure handling**: what could go wrong + fallback/diagnostic.
11. **Novelty and elegance argument**: why more than naming a module, why the paper looks focused.

If the method is only "add a module" or "use a planner," it is **not concrete enough**.

### Step 1.5: Design Minimal Claim-Driven Validation

For each core claim, define the **smallest strong experiment**:
- the claim being tested
- the necessary baseline or ablation
- the decisive metric
- the expected directional outcome

Rules:
- Ensure one block directly supports the Problem Anchor
- If complexity risk exists, include one simplification/deletion check
- If a frontier primitive is central, include one necessity check
- Default to 1-3 core experiment blocks (full roadmap is for experiment_plan phase)

Soft caps: MAX_PRIMARY_CLAIMS = 2, MAX_NEW_TRAINABLE_COMPONENTS = 2, MAX_CORE_EXPERIMENTS = 3.

### Step 1.6: Write the Initial Proposal

Save to `refine-logs/round-0-initial-proposal.md` using the full proposal template:

```markdown
# Research Proposal: [Title]

## Problem Anchor
- Bottom-line problem:
- Must-solve bottleneck:
- Non-goals:
- Constraints:
- Success condition:

## Technical Gap
[Why current methods fail, why naive bigger systems are not enough, and what mechanism is missing]

## Method Thesis
- One-sentence thesis:
- Why this is the smallest adequate intervention:
- Why this route is timely in the foundation-model era:

## Contribution Focus
- Dominant contribution:
- Optional supporting contribution:
- Explicit non-contributions:

## Proposed Method
### Complexity Budget
- Frozen / reused backbone:
- New trainable components:
- Tempting additions intentionally not used:

### System Overview
[Step-by-step pipeline or ASCII graph]

### Core Mechanism
- Input / output:
- Architecture or policy:
- Training signal / loss:
- Why this is the main novelty:

### Optional Supporting Component
- Only include if truly necessary:
- Input / output:
- Training signal / loss:
- Why it does not create contribution sprawl:

### Modern Primitive Usage
- Which LLM / VLM / Diffusion / RL-era primitive is used:
- Exact role in the pipeline:
- Why it is more natural than an old-school alternative:

### Integration into Base Generator / Downstream Pipeline
[Where the new method attaches, what is frozen, what is trainable, inference order]

### Training Plan
[Stagewise or joint training, losses, data construction, pseudo-labels, schedules]

### Failure Modes and Diagnostics
- [Failure mode]:
- [How to detect]:
- [Fallback or mitigation]:

### Novelty and Elegance Argument
[Closest work, exact difference, why this is a focused mechanism-level contribution rather than a module pile-up]

## Claim-Driven Validation Sketch
### Claim 1: [Main claim]
- Minimal experiment:
- Baselines / ablations:
- Metric:
- Expected evidence:

### Claim 2: [Optional]
- Minimal experiment:
- Baselines / ablations:
- Metric:
- Expected evidence:

## Experiment Handoff Inputs
- Must-prove claims:
- Must-run ablations:
- Critical datasets / metrics:
- Highest-risk assumptions:

## Compute & Timeline Estimate
- Estimated GPU-hours:
- Data / annotation cost:
- Timeline:
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

    1. **Problem Fidelity** (20%): Does the proposal still attack the original problem, or has it drifted?

    2. **Completeness** (20%): Has the approach considered all necessary angles? Missing important methods, baselines, theoretical perspectives, or edge cases? Is the scope appropriate for the research goal?

    3. **Logical Soundness** (20%): Is the reasoning chain airtight? Are assumptions justified? Any logical gaps between problem, approach, and expected evidence?

    4. **Feasibility** (15%): Is the proposal concrete enough to implement? Can it actually produce results with the stated resources and timeline?

    5. **Significance** (15%): Is this worth doing? Would the results (positive or negative) advance understanding in the field?

    6. **First Principles** (10%): Is the approach grounded in fundamental understanding of the problem, or just following conventions / trends / surface-level patterns?

    **OVERALL SCORE** (1-10): Use this weighting: Problem Fidelity 20%, Completeness 20%, Logical Soundness 20%, Feasibility 15%, Significance 15%, First Principles 10%.

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

| Round | Problem Fidelity | Completeness | Logical Soundness | Feasibility | Significance | First Principles | Overall | Verdict |
|-------|------------------|--------------|-------------------|-------------|--------------|------------------|---------|---------|
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

Ensure it contains the complete score evolution table using all 7 dimensions.

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
