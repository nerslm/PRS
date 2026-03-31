---
name: research-init
description: "Initialize or sync project state. Creates/updates PROJECT.md, detects existing assets, auto-starts the appropriate stage. Trigger with /research-init."
argument-hint: [research-direction]
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, Skill
---

# Research Init

Initialize or sync project for: **$ARGUMENTS**

## Workflow

### Step 1: Detect State

Check if `PROJECT.md` exists:
- **YES** → Go to Step 3 (Sync)
- **NO** → Go to Step 2 (New Project)

### Step 2: New Project

**If $ARGUMENTS provided**: use it as research direction, parse inline params (venue, etc.), go to Step 2b.

**If $ARGUMENTS empty**: ask in one message:

```
1. Research direction (describe in your own words):
2. Target venue:
   - nature-communications / prl / neurips / (other)
```

Wait for response, then go to Step 2b.

**Step 2b**: Show config, ask user to confirm:

```
📋 Config:
- Direction: [direction]
- Venue: [venue]
- Auto proceed: true
- Human checkpoint: true

Change anything? Or Enter to confirm.
```

Wait for response. Then scan directory and generate `PROJECT.md`:

```markdown
# PROJECT.md

## Config
direction: [direction]
venue: [venue]
auto_proceed: [true/false]
human_checkpoint: [true/false]
created: [date]

## Progress
- [date] init | [assets summary or "fresh project"]
```

Then go to Step 4 (Start).

### Step 3: Sync Existing

Read `PROJECT.md`, scan directory for assets, reconcile Progress vs actual files. Fix discrepancies, report status. Then go to Step 4 (Start).

### Step 4: Start

Scan directory for these files to determine current stage:

| File | Stage |
|---|---|
| (nothing) | S1 — start `/research-pipeline` |
| `IDEA_REPORT.md` | S1 done |
| `refine-logs/FINAL_PROPOSAL.md` | S1 refine done |
| `refine-logs/EXPERIMENT_PLAN.md` | S1 plan done |
| `code/*.py` | S2 done |
| `results/pilot/` or `PILOT_RESULTS.md` | S4 done |
| `results/full/` or `FULL_RESULTS.md` | S5 done |
| `AUTO_REVIEW.md` | S5 review done |
| `NARRATIVE_REPORT.md` | S6 ready |
| `paper/main.tex` | S6 started |
| `paper/main.pdf` | S6 done |

**If fresh project (no assets)** → start `/research-pipeline "$ARGUMENTS"` immediately.

**If assets detected** → show status and recommended next step, wait for user confirmation:

```
S1 Idea:     ✓ IDEA_REPORT.md
S2 Code:     ✓ code/
S3 Results:  ✗ not started
...
Recommended: [next stage]. Proceed?
```

Then invoke the appropriate skill:

| Next Stage | Skill |
|---|---|
| S1 | `/research-pipeline "$ARGUMENTS"` |
| S2 | `/research-pipeline "$ARGUMENTS"` (skips to Stage 2) |
| S3 | `/run-experiment` |
| S4 | `/auto-review-loop` |
| S5 | `/paper-writing "NARRATIVE_REPORT.md"` |
| S6 improve | `/auto-paper-improvement-loop "paper/"` |

## Key Rules

- Never delete progress entries — only append.
- Never modify research output files — only read to detect state.
- Conservative detection — empty file = "started" not "done".
- Keep PROJECT.md concise — key-value config, one-line progress entries.
