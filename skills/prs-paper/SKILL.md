---
name: prs-paper
description: "Paper writing shortcut. Routes to the next paper activity based on artifact state. Use when user says \"写论文\", \"write paper\", \"paper continue\"."
argument-hint: [optional-paper-intent]
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, WebSearch, WebFetch, Agent, Skill, mcp__codex__codex, mcp__codex__codex-reply
---

# PRS Paper — Paper Writing Orchestrator

**Intent**: $ARGUMENTS

## Step 1: Load State

Read `project.json`, `progress.json`, `session.json`.
If `project.json` missing → tell user to run `/prs-init`.

## Step 2: Determine Paper Activity

If `current_stage` is not `paper`:
- Check prerequisites: auto_review or results must exist
- Warn if missing, ask user if they want to proceed anyway
- Set `current_stage` to `paper`

Determine next paper activity from artifacts registry:

| Missing Artifact | next_action.goal |
|---|---|
| paper_plan | "Create paper plan with Claims-Evidence Matrix" |
| figures | "Generate paper figures from results" |
| paper_source | "Write paper sections in LaTeX" |
| paper_pdf | "Compile paper PDF" |
| (pdf exists, no improvement log) | "Run paper improvement loop" |

If $ARGUMENTS specifies intent (e.g., "rewrite intro", "fix figures"), override `next_action.goal`.

## Step 3: Present Status

```
PRS Paper: [title] | Venue: [venue]
Next: [next_action.goal]

Artifacts:
  paper_plan:   [ready/missing]
  figures:      [ready/missing]
  paper_source: [ready/missing]
  paper_pdf:    [ready/missing]
```

## Step 4: Execute

Same as `/prs` Step 3 — read the relevant paper module, do it yourself or delegate grunt work (figures, compile), update artifacts and next_action.

After completion, advance to next paper activity or report done. Check `autonomy` mode before pausing.

## Key Rules

- Grunt work (figures, compile) → sub-agent. Narrative design, writing → yourself.
- Anti-hallucination: DBLP → CrossRef → [VERIFY]
- Venue compliance: check venue profile for format requirements
- Check `autonomy` before pausing — same rules as `/prs`
