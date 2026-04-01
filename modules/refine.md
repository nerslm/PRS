# Refine Context Module — Problem-Anchored Method Refinement

This module covers the `refine` phase. It is loaded by `/prs` when `current_stage` is `idea` and literature survey + idea generation are done.

The goal: turn a vague direction into a **problem → focused method → minimal validation** document that is concrete enough to implement, elegant enough to feel paper-worthy, and current enough to resonate in the foundation-model era.

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
2. **The smallest adequate mechanism wins.** Prefer the minimal intervention that directly fixes the bottleneck.
3. **One paper, one dominant contribution.** Prefer one sharp thesis plus at most one supporting contribution.
4. **Modern leverage is a prior, not a decoration.** When LLM / VLM / Diffusion / RL / distillation / inference-time scaling naturally fit the bottleneck, use them concretely. Do not bolt them on as buzzwords.

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
    This is an early-stage, method-first research proposal.

    Your job is NOT to reward extra modules, contribution sprawl, or a giant benchmark checklist.
    Your job IS to stress-test whether the proposed method:
    (1) still solves the original anchored problem,
    (2) is concrete enough to implement,
    (3) presents a focused, elegant contribution,
    (4) uses foundation-model-era techniques appropriately when they are the natural fit.

    Review principles:
    - Prefer the smallest adequate mechanism over a larger system.
    - Penalize parallel contributions that make the paper feel unfocused.
    - If a modern LLM / VLM / Diffusion / RL route would clearly produce a better paper, say so concretely.
    - If already modern enough, do NOT force trendy components.
    - Do not ask for extra experiments unless needed to prove core claims.
    - Read the Problem Anchor first. If your fix would change the problem, call it drift.

    === PROPOSAL ===
    [Paste FULL proposal]
    === END PROPOSAL ===

    Score these 7 dimensions from 1-10:

    1. **Problem Fidelity** (15%): Does the method still attack the original bottleneck, or has it drifted into solving something easier or different?

    2. **Method Specificity** (25%): Are the interfaces, representations, losses, training stages, and inference path concrete enough that an engineer could start implementing?

    3. **Contribution Quality** (25%): Is there one dominant mechanism-level contribution with real novelty, good parsimony, and no obvious contribution sprawl?

    4. **Frontier Leverage** (15%): Does the proposal use current foundation-model-era primitives appropriately when they are the right tool, instead of defaulting to old-school module stacking?

    5. **Feasibility** (10%): Can this method be trained and integrated with the stated resources and data assumptions?

    6. **Validation Focus** (5%): Are the proposed experiments minimal but sufficient to validate the core claims? Is there unnecessary experimental bloat?

    7. **Venue Readiness** (5%): If executed well, would the contribution feel sharp and timely enough for [VENUE]?

    **OVERALL SCORE** (1-10): Weighted toward Problem Fidelity, Method Specificity, Contribution Quality, and Frontier Leverage.
    Use this weighting: Problem Fidelity 15%, Method Specificity 25%, Contribution Quality 25%, Frontier Leverage 15%, Feasibility 10%, Validation Focus 5%, Venue Readiness 5%.

    For each dimension scoring < 7, provide:
    - The specific weakness
    - A concrete fix at the method level (interface / loss / training recipe / integration point / deletion of unnecessary parts)
    - Priority: CRITICAL / IMPORTANT / MINOR

    Then add:
    - **Simplification Opportunities**: 1-3 concrete ways to delete, merge, or reuse components while preserving the main claim. Write "NONE" if already tight.
    - **Modernization Opportunities**: 1-3 concrete ways to replace old-school pieces with more natural foundation-model-era primitives if genuinely better. Write "NONE" if already modern enough.
    - **Drift Warning**: "NONE" if the proposal still solves the anchored problem; otherwise explain the drift clearly.
    - **Verdict**: READY / REVISE / RETHINK

    Verdict rule:
    - READY: overall score >= 9, no meaningful drift, one focused dominant contribution, and no obvious complexity bloat remains
    - REVISE: the direction is promising but not yet at READY bar
    - RETHINK: the core mechanism or framing is still fundamentally off
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
    First, check whether the original Problem Anchor is still preserved.
    Second, judge whether the method is now more concrete, more focused, and more current.

    Key changes:
    1. [Method change 1]
    2. [Method change 2]
    3. [Simplification / modernization / pushback if any]

    === REVISED PROPOSAL ===
    [Paste the FULL revised proposal]
    === END REVISED PROPOSAL ===

    Please:
    - Re-score the same 7 dimensions and overall
    - State whether the Problem Anchor is preserved or drifted
    - State whether the dominant contribution is now sharper or still too broad
    - State whether the method is simpler or still overbuilt
    - State whether the frontier leverage is now appropriate or still old-school / forced
    - Focus new critiques on missing mechanism, weak training signal, weak integration point, pseudo-novelty, or unnecessary complexity
    - Use the same verdict rule: READY only if overall score >= 9 and no blocking issue remains

    Same output format: 7 scores, overall score, verdict, drift warning, simplification opportunities, modernization opportunities, remaining action items.
```

---

## Phase 3: Parse Feedback and Revise

### Anchor Check (before every revision)

1. Copy Problem Anchor verbatim
2. What is the original bottleneck?
3. Does the current method still solve it?
4. Which reviewer suggestions would cause drift?

### Simplicity Check (before every revision)

1. What is the dominant contribution now?
2. What components can be removed, merged, or kept frozen?
3. Which reviewer suggestions add unnecessary complexity?
4. If a frontier primitive is central, is its role still crisp?

### Revision Strategy

- **Valid feedback**: sharpen mechanism, simplify, modernize if paper improves.
- **Debatable feedback**: revise but explain reasoning with evidence.
- **Wrong/drifting/over-complicating**: push back with evidence from Problem Anchor.

Bias toward: sharper central contribution, fewer moving parts, cleaner backbone reuse, natural frontier leverage, leaner claim-driven experiments.

**Do NOT add multiple parallel contributions to chase score.** If reviewer requests another module, first ask whether the same gain can come from a better interface, distillation signal, reward model, or inference policy.

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

| Round | Main Reviewer Concerns | What This Round Simplified / Modernized | Solved? | Remaining Risk |
|-------|-------------------------|------------------------------------------|---------|----------------|
| 1     | [top issues from review] | [main method changes]                    | [yes / partial / no] | [if any] |
| 2     | ...                     | ...                                      | ...     | ...            |

## Overall Evolution
- [How the method became more concrete]
- [How the dominant contribution became more focused]
- [How unnecessary complexity was removed]
- [How modern technical leverage improved or stayed intentionally minimal]
- [How drift was avoided or corrected]

## Final Status
- Anchor status: [preserved / corrected / unresolved]
- Focus status: [tight / slightly broad / still diffuse]
- Modernity status: [appropriately frontier-aware / intentionally conservative / still old-school]
- Strongest parts of final method:
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

| Round | Problem Fidelity | Method Specificity | Contribution Quality | Frontier Leverage | Feasibility | Validation Focus | Venue Readiness | Overall | Verdict |
|-------|------------------|--------------------|----------------------|-------------------|-------------|------------------|-----------------|---------|---------|
| 1     | ...              | ...                | ...                  | ...               | ...         | ...              | ...             | ...     | ...     |

## Round-by-Round Review Record

| Round | Main Reviewer Concerns | What Was Changed | Result |
|-------|-------------------------|------------------|--------|
| 1     | [top issues]            | [main fixes]     | [resolved / partial / unresolved] |
| 2     | ...                     | ...              | ...    |

## Final Proposal Snapshot
- Canonical clean version lives in `refine-logs/FINAL_PROPOSAL.md`
- Summarize the final thesis in 3-5 bullets here

## Method Evolution Highlights
1. [Most important simplification or focusing move]
2. [Most important mechanism upgrade]
3. [Most important modernization or justification for staying simple]

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

## Simplicity Check
- Dominant contribution after revision:
- Components removed or merged:
- Reviewer suggestions rejected as unnecessary complexity:
- Why the remaining mechanism is still the smallest adequate route:

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

Focus status:
- [tight / slightly broad / still diffuse]

Modernity status:
- [appropriately frontier-aware / intentionally conservative / still old-school]

Key method upgrades:
- [method change 1]
- [method change 2]

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
- **One paper, one dominant contribution.** Avoid multiple parallel contributions.
- **The smallest adequate mechanism wins.** Bigger is not automatically better.
- **Prefer reuse over invention.** Start from strong existing backbones.
- **Modern techniques are a prior, not a decoration.**
- **Minimal experiments.** Only enough to prove core claims in this phase.
- **Review the mechanism, not the parts count.** A long module list is not novelty.
- **Pushback is encouraged.** Argue back with evidence when feedback causes drift or complexity.
- **ALWAYS use `config: {"model_reasoning_effort": "xhigh"}`** for all Codex review calls.
- **Save threadId** and use `mcp__codex__codex-reply` for later rounds.
- **Do not fabricate results.** Only describe expected evidence.
- **Be specific about compute and data assumptions.**
- **Document everything.** Save every raw review, anchor check, simplicity check, and method change.

---

## Verification Criteria

- refine → experiment_plan: `refine-logs/FINAL_PROPOSAL.md` exists with substantive content, score history shows progression, Problem Anchor is preserved in final version.
