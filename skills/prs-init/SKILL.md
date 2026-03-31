---
name: prs-init
description: "Initialize or sync a PRS research project. Creates project.json + progress.json with artifact registry and next_action. Detects existing work."
argument-hint: [research-direction]
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob
---

# PRS Init

Initialize or sync research project for: **$ARGUMENTS**

## Step 1: Detect State

Check if `project.json` exists in the current directory:
- **YES** → Step 3 (Sync Existing)
- **NO** → Step 2 (New Project)

## Step 2: New Project

Parse $ARGUMENTS for any explicitly provided info (direction, venue, mode). Then **ask the user for anything not provided**. Three questions, asked in order — do NOT assume defaults for these:

### Question 1: Research Direction

If $ARGUMENTS contains a clear direction → use it, confirm with user.
If $ARGUMENTS is a project folder path → scan the folder, read key files, summarize what you find, then confirm the direction with the user.
If $ARGUMENTS is empty or unclear → ask:
```
What is your research direction? (describe the problem, domain, and constraints)
```

### Question 2: Target Venue

If user already specified (e.g., `— venue: prl`) → use it.
Otherwise **always ask**:
```
What is the target venue?
  - neurips / iclr / icml / cvpr / acl / aaai (ML conferences)
  - nature-communications (interdisciplinary)
  - prl (physics)
  - other: [specify]
```

### Question 3: Automation Mode

If user already specified (e.g., "fully automatic", "I want to approve each step") → use it.
Otherwise **always ask**:
```
How do you want to drive the research?

  A. Full auto — runs the entire pipeline without pausing, from idea to paper
  B. Gates — auto-advances within stages, pauses at key decision points
     (idea selection, experiment plan approval, paper outline approval)
  C. Manual — pauses after every major action for your review and direction
```

### After all 3 answers

Show the complete config for confirmation:
```
Config:
  Direction: [direction]
  Venue: [venue]
  Autonomy: [full / gates / manual]

Change anything? Or confirm to proceed.
```

Then create **project.json**:
```json
{ "version": 1, "title": "[short title]", "direction": "[direction]", "venue": "[venue]",
  "created": "[today]", "config": { "autonomy": "[full/gates/manual]",
    "reviewer_model": "gpt-5.4", "arxiv_download": false, "pilot_max_hours": 2,
    "max_review_rounds": 4, "max_refine_rounds": 5, "max_paper_improve_rounds": 4,
    "score_threshold_refine": 9, "score_threshold_review": 8.5 } }
```

Tip: input templates in `input-templates/` can help structure your research brief.

Then go to Step 4.

## Step 3: Sync Existing

Read `project.json` and `progress.json`. Scan directory for artifacts. Reconcile artifacts registry against actual files. Fix discrepancies. Then go to Step 5.

If migrating from old PRS (has `PROJECT.md` or old `progress.json` with `current_phase`): read old state, map phases to stages, build new format progress.json.

## Step 4: Build Artifacts Registry and Infer Stage

Scan project directory. For each known artifact, check canonical path first, then legacy fallback:

| Artifact | Canonical Path | Legacy Fallback |
|---|---|---|
| lit_survey | notes/lit_survey.md | LITERATURE_REVIEW.md |
| idea_report | IDEA_REPORT.md | notes/IDEA_REPORT.md |
| final_proposal | refine-logs/FINAL_PROPOSAL.md | logs/FINAL_PROPOSAL.md |
| experiment_plan | refine-logs/EXPERIMENT_PLAN.md | logs/EXPERIMENT_PLAN.md |
| experiment_tracker | EXPERIMENT_TRACKER.md | refine-logs/EXPERIMENT_TRACKER.md |
| code | code/ | — |
| results | results/ | — |
| auto_review | AUTO_REVIEW.md | logs/AUTO_REVIEW.md |
| paper_plan | PAPER_PLAN.md | paper/PAPER_PLAN.md |
| figures | figures/latex_includes.tex | paper/figures/latex_includes.tex |
| paper_source | paper/main.tex | — |
| paper_pdf | paper/main.pdf | — |

Set each artifact's `status` to `ready` (exists with content) or `missing`.

**Infer current stage**:

| Condition | Stage |
|---|---|
| No idea/proposal/results/paper artifacts | idea |
| Idea artifacts exist, no experiment plan or results | idea (may be nearly done) |
| Experiment plan or code exists, results missing | experiment |
| Results exist, no review or review incomplete | review |
| Paper artifacts exist | paper |
| Final PDF + improvement log exist | done |

**Write initial next_action** based on what's missing:

| Stage | Missing artifact | next_action.goal |
|---|---|---|
| idea | lit_survey | "Run literature survey on the research direction" |
| idea | idea_report | "Generate and rank candidate research ideas" |
| idea | final_proposal | "Refine selected idea into a concrete proposal" |
| experiment | experiment_plan | "Design claim-driven experiment plan" |
| experiment | code | "Implement experiment code from plan" |
| experiment | results | "Deploy and run experiments, collect results" |
| review | auto_review | "Run autonomous review loop" |
| paper | paper_plan | "Create paper plan with Claims-Evidence Matrix" |
| paper | figures | "Generate paper figures from results" |
| paper | paper_source | "Write paper sections in LaTeX" |
| paper | paper_pdf | "Compile paper PDF" |
| paper | (pdf exists) | "Run paper improvement loop" |

**Initialize stage summaries**: for stages with completed artifacts, read the first ~20 lines of key artifacts to build brief summaries.

Create **progress.json** with: `current_stage`, `next_action`, `artifacts`, `stages`, empty `history`.

## Step 5: Show Status

```
PRS Project: [title]
Venue: [venue] | Created: [date]

Stage Status:
  idea:       [done / in_progress / not_started]
  experiment: [done / in_progress / not_started]
  review:     [done / in_progress / not_started]
  paper:      [done / in_progress / not_started]

Current stage: [current_stage]
Next action: [next_action.goal]

Artifacts:
  [artifact_id]: [ready/missing] ([path])
  ...

Recommended: Run /prs to work on "[next_action.goal]".
```

## Key Rules

- Never delete progress — only append to history
- Never modify research output files during detection — only read
- Conservative: empty file = "in_progress" not "ready"
- Always write next_action — the next session depends on it
- Legacy compatibility: check canonical path first, then fallback. Writers always use canonical.
