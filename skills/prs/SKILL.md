---
name: prs
description: "Universal PRS entry point. Reads state, executes next_action, updates progress. /prs to continue, /prs \"write the paper\" to jump."
argument-hint: [optional-intent]
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, WebSearch, WebFetch, Agent, Skill, mcp__codex__codex, mcp__codex__codex-reply
---

# PRS — Research Orchestrator

**Intent**: $ARGUMENTS

You own the research story. Execute `next_action`, then decide what comes next.

## Step 1: Load State

Read from project root:
1. **project.json** — config (including `autonomy` mode)
2. **progress.json** — `current_stage`, `next_action`, `artifacts`, stage summaries
3. **session.json** — previous session handoff (if exists)

If `project.json` missing → tell user to run `/prs-init` and stop.

## Step 2: Orient

If $ARGUMENTS has an intent, map to stage:
- literature/survey/idea/brainstorm → `idea`
- experiment/code/implement/deploy/run → `experiment`
- review/evaluate → `review`
- paper/write/draft → `paper`

Otherwise use `current_stage` + `next_action` from progress.json.

Present status:
```
PRS: [title] | Venue: [venue] | Stage: [current_stage] | Autonomy: [mode]
Next action: [next_action.goal]
[Key lines from stage summaries]
Last session: [accomplished, or "first session"]
```

## Step 3: Execute next_action

Read the relevant context module (see CLAUDE.md module table) for domain guidance.

- **Grunt work** (search, code, compile, deploy, figures) → delegate to sub-agent
- **Everything else** → do it yourself

## Step 4: After Completion

1. **Update artifacts** — set status to `ready` in progress.json
2. **Update stage summary** — what was accomplished + key results
3. **Verify** — check output files exist
4. **Check loop score** — if in refine/review/paper-improve: score < threshold AND rounds remain → continue the loop, don't advance stage.
5. **Write next `next_action`** — what to do next
6. **Check autonomy mode**:
   - `full` → keep going, loop back to Step 3
   - `gates` → keep going UNLESS at a stage boundary gate — then pause
   - `manual` → pause, report, ask
7. **Append to history**

## Step 5: Session End

1. Update progress.json — stages, artifacts, next_action, history
2. Write session.json — accomplished, next_actions, active threads, critical context, files modified

## Key Rules

- **next_action is the control surface** — always update it
- **You own the research story** — understand results deeply enough to decide what comes next
- **Delegate grunt work, retain judgment**
- **One action at a time** — complete, update, then proceed
- **Verify before advancing stage**
- **Check `autonomy` before pausing** — the ONLY thing that controls whether you pause
- **Loop control is score-only** — refine/review/paper-improve continue until score >= threshold or max rounds. Ignore verdict text.
- **Large files** — if Write fails, use `Bash(cat << 'EOF' > file)`
