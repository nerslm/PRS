# PRS — Thin Harness for Scientific Research

PRS is a state-driven thin harness for long-running scientific research. The main agent (`/prs`) owns the research story — it does almost everything directly, maintaining deep understanding across all stages. `next_action` in `progress.json` drives progress.

## Core Principles

1. **Main agent does the research.** You execute the full pipeline yourself — literature survey, idea evaluation, method refinement, experiment design, result interpretation, paper writing.
2. **next_action is the control surface.** After every action, write what to do next.
3. **Use sub-agents only for context-polluting grunt work.** Bulk web searches, code from a clear spec, experiment deployment, figure generation, LaTeX compilation, citation fetching.
4. **Read context modules for domain guidance.** The module is your reference, not a script.
5. **Sub-agents CANNOT spawn sub-agents.** For parallel search, YOU decompose and spawn.

---

## Autonomy Mode

`project.json` has a single `autonomy` field that controls when to pause for user input:

| Mode | Behavior |
|---|---|
| `full` | Don't pause for user input — auto-select best options at gates. **Every step is still mandatory** (literature survey, Codex review, refinement loops, experiments, review loop, paper improvement). Autonomy controls pausing, NOT which steps to execute. |
| `gates` | Pause only at stage boundaries (idea selection, experiment plan approval, paper outline approval). Auto-proceed within stages. |
| `manual` | Pause after every major action to report results and get user direction. |

---

## Research Process Overview

```
/prs-init "research direction" — venue: [venue]
│   Creates: project.json, progress.json
│
▼ Stage: idea                                    Modules: research.md, refine.md
│   1. Literature survey — decompose directions, spawn parallel search sub-agents, synthesize
│   2. Idea generation — brainstorm (Codex MCP), filter, novelty check, pilot
│   3. Refine proposal — drive each round (Codex MCP reviews, you do anchor check + revisions)
│   ○ Gate: user selects idea (autonomy=gates/manual only)
│   Artifacts: notes/lit_survey.md, IDEA_REPORT.md, refine-logs/FINAL_PROPOSAL.md
│
▼ Stage: experiment                              Module: experiment.md
│   1. Experiment plan — design yourself
│   2. Implementation — delegate code writing if straightforward
│   3. Deployment — delegate SSH/screen/monitoring
│   4. Result collection
│   ○ Gate: user approves experiment plan (autonomy=gates/manual only)
│   Artifacts: refine-logs/EXPERIMENT_PLAN.md, code/, results/
│
▼ Stage: review                                  Module: review.md
│   Drive review loop — understand reviewer feedback to decide what to fix
│   Artifacts: AUTO_REVIEW.md
│
▼ Stage: paper                                   Modules: paper-common.md + paper-{plan,figures,write,compile,improve}.md
│   1. Paper plan — design narrative structure, claims-evidence matrix
│   2. Figures — delegate matplotlib generation
│   3. Write — design narrative yourself, can delegate LaTeX drafting
│   4. Compile — delegate (mechanical)
│   5. Improve — drive review yourself or delegate if fixes are straightforward
│   ○ Gate: user approves paper outline (autonomy=gates/manual only)
│   Artifacts: PAPER_PLAN.md, figures/, paper/main.tex, paper/main.pdf
│
▼ done
```

### Auxiliary Skills (standalone, invoke anytime)

`/paper-poster` `/paper-slides` `/paper-illustration` `/grant-proposal` `/formula-derivation` `/proof-writer` `/dse-loop` `/comm-lit-review` `/arxiv` `/mermaid-diagram` `/feishu-notify` `/pixel-art`

### Input Templates (in `input-templates/`)

`RESEARCH_BRIEF_TEMPLATE.md` `NARRATIVE_REPORT_TEMPLATE.md` `EXPERIMENT_PLAN_TEMPLATE.md` `PAPER_PLAN_TEMPLATE.md`

---

## State Files

**project.json** — config (write-once):
```json
{ "version": 1, "title": "...", "direction": "...", "venue": "neurips", "created": "YYYY-MM-DD",
  "config": { "autonomy": "gates", "reviewer_model": "gpt-5.4",
    "arxiv_download": false, "pilot_max_hours": 2, "max_review_rounds": 4,
    "max_refine_rounds": 5, "max_paper_improve_rounds": 4,
    "score_threshold_refine": 9, "score_threshold_review": 8.5 } }
```

**progress.json** — stage states + next_action + artifacts:
```json
{
  "current_stage": "idea",
  "next_action": {
    "goal": "Run literature survey on the research direction",
    "inputs": [],
    "success_criteria": "notes/lit_survey.md written with landscape map and gaps",
    "do_not_do": "Don't generate ideas yet — survey first"
  },
  "artifacts": {
    "lit_survey": { "path": "notes/lit_survey.md", "status": "missing" },
    "idea_report": { "path": "IDEA_REPORT.md", "status": "missing" },
    "final_proposal": { "path": "refine-logs/FINAL_PROPOSAL.md", "status": "missing" },
    "experiment_plan": { "path": "refine-logs/EXPERIMENT_PLAN.md", "status": "missing" },
    "code": { "path": "code/", "status": "missing" },
    "results": { "path": "results/", "status": "missing" },
    "auto_review": { "path": "AUTO_REVIEW.md", "status": "missing" },
    "paper_plan": { "path": "PAPER_PLAN.md", "status": "missing" },
    "figures": { "path": "figures/latex_includes.tex", "status": "missing" },
    "paper_source": { "path": "paper/main.tex", "status": "missing" },
    "paper_pdf": { "path": "paper/main.pdf", "status": "missing" }
  },
  "stages": {
    "idea": { "status": "in_progress", "summary": "" },
    "experiment": { "status": "not_started", "summary": "" },
    "review": { "status": "not_started", "summary": "" },
    "paper": { "status": "not_started", "summary": "" }
  },
  "history": []
}
```

**session.json** — handoff (overwritten each session):
`accomplished`, `next_actions`, `active_threads`, `critical_context`, `files_modified`, `warnings`.

---

## Stage Transitions

| From | To | Condition |
|---|---|---|
| (init) | idea | project.json created |
| idea | experiment | FINAL_PROPOSAL.md ready |
| experiment | review | Results collected in results/ |
| review | paper | Review score >= threshold OR max rounds |
| paper | done | Improvement rounds complete |

**Gates** exist at: idea→experiment, experiment plan approval, paper outline approval. Gates are **skipped when `autonomy=full`**, the agent auto-selects the best option. In `gates` mode, pause at these transitions. In `manual` mode, pause after every action.

**Verification**: stages advance only in order (idea → experiment → review → paper). Before each transition, verify the condition above is met and all required artifacts for the current stage are `ready`.

---

## Context Modules

| Stage | Module(s) to read |
|---|---|
| idea (survey + brainstorm) | `modules/research.md` |
| idea (refine) | `modules/refine.md` |
| experiment | `modules/experiment.md` |
| review | `modules/review.md` |
| paper plan | `modules/paper-common.md` + `modules/paper-plan.md` |
| paper figures | `modules/paper-common.md` + `modules/paper-figures.md` |
| paper write | `modules/paper-common.md` + `modules/paper-write.md` |
| paper compile | `modules/paper-common.md` + `modules/paper-compile.md` |
| paper improve | `modules/paper-common.md` + `modules/paper-improve.md` |

Module paths relative to `.claude/skills/`. On-demand references:
- `references/venues/{venue}.md` — venue profile
- `references/writing-principles.md` — narrative quality
- `references/citation-discipline.md` — citation verification

---

## Using Sub-Agents

**If it's easy but would pollute your context, hand it to a sub-agent.** Examples: bulk web searches, writing code from a clear spec, SSH experiment deployment, generating matplotlib figures, compiling LaTeX.

When delegating, pass: stage summaries + module absolute path (resolve via Glob) + project root + config. Sub-agent reads module, executes, writes artifacts, reports back. Sub-agents cannot spawn sub-agents.

---

## Session Protocol

**Start**: Read state files → read `next_action` → present status.
**Work**: Execute `next_action` → read relevant module → update artifacts + stage summary → write new `next_action` → check autonomy mode → repeat or pause.
**End**: Update progress.json → write session.json.

---

## ARIS File Conventions

| Stage | Output Files |
|---|---|
| idea | notes/lit_survey.md, IDEA_REPORT.md, refine-logs/FINAL_PROPOSAL.md, refine-logs/round-N-*.md |
| experiment | refine-logs/EXPERIMENT_PLAN.md, EXPERIMENT_TRACKER.md, code/, results/ |
| review | AUTO_REVIEW.md |
| paper | PAPER_PLAN.md, figures/, paper/main.tex, paper/sections/*.tex, paper/references.bib, paper/main.pdf |

---

## Fallback

If a module doesn't cover an edge case, read the original skill in `archive/{skill-name}/SKILL.md`.

---

## Key Rules

- **next_action is the control surface** — always update after completing work
- **Never delete progress** — history is append-only
- **JSON for state, Markdown for artifacts**
- **One action at a time** — complete, update, then proceed
- **Check `autonomy` before pausing** — full=never pause, gates=pause at stage boundaries, manual=pause after every action
- **Loop control is score-only** — refine/review/paper-improve continue until numeric score >= threshold or max rounds exhausted. Ignore verdict text.
- **Large files** — if Write fails, use `Bash(cat << 'EOF' > file)`
- **Anti-hallucination citations** — DBLP → CrossRef → `[VERIFY]` (see `modules/paper-write.md`)
- **You do the research, sub-agents do the chores**
- **Never skip steps** — autonomy controls pausing, not which steps to execute
