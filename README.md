# PRS — Personal Research System

A thin harness for Claude Code doing scientific research. From idea to submission.

## What Is This

PRS gives Claude Code the structure it needs for long-running research: state tracking, session recovery, domain knowledge modules, and a clear pipeline. The main agent does the research — PRS just keeps it organized.

## Quick Start

```bash
# Initialize — asks 3 questions: direction, venue, autonomy mode
/prs-init "your research direction"

# Continue research from where you left off
/prs

# Jump to paper writing
/prs-paper

# With intent
/prs "refine my method"
/prs "run experiments"
```

## How It Works

```
/prs-init → project.json + progress.json
                │
/prs        → reads state → reads next_action → executes → updates state → repeats
                │
           4 stages, driven by next_action:

           idea → experiment → review → paper → done

           Each stage has context modules with domain constraints.
           The main agent does the thinking. Sub-agents do the chores.
```

## 4 Stages

| Stage | What Happens | Key Artifacts |
|---|---|---|
| **idea** | Literature survey → brainstorm ideas → novelty check → refine proposal | IDEA_REPORT.md, refine-logs/FINAL_PROPOSAL.md |
| **experiment** | Design experiments → implement code → deploy → collect results | EXPERIMENT_PLAN.md, code/, results/ |
| **review** | Autonomous review loop via Codex MCP → implement fixes → re-review | AUTO_REVIEW.md |
| **paper** | Plan → figures → write LaTeX → compile → venue-aware improvement | PAPER_PLAN.md, paper/main.pdf |

## Autonomy Modes

Set during `/prs-init`. One parameter, three choices:

| Mode | Behavior |
|---|---|
| `full` | Runs the entire pipeline without pausing. Idea to paper, hands-off. |
| `gates` | Auto-advances within stages. Pauses at 3 decision points: idea selection, experiment plan, paper outline. |
| `manual` | Pauses after every major action for your review. |

## State Files

All JSON, in the project root:

- **project.json** — direction, venue, autonomy, config (write-once)
- **progress.json** — current stage, `next_action`, artifacts registry, stage summaries, history
- **session.json** — handoff for the next session (what was done, what to do next)

`next_action` is the control surface — it tells the agent exactly what to do next. No hardcoded phase transitions.

## Context Modules

Domain knowledge distilled from the original skills. The main agent reads them for guidance when working on a stage:

| Module | Content |
|---|---|
| `research.md` | Literature search patterns, Codex MCP brainstorm prompts, novelty verification, pilot experiment design |
| `refine.md` | Problem Anchor pattern, 7-dimension scoring rubric, Codex MCP review templates, simplicity/anchor checks |
| `experiment.md` | Claim-driven experiment blocks, SSH+screen deployment, code review, W&B integration |
| `review.md` | Review loop (Codex MCP + LLM fallback), crash recovery, fix prioritization, full-text review via sub-agent |
| `paper-*.md` | Claims-Evidence Matrix, section guidelines, DBLP citation chain, De-AI polish, latexmk recipes, venue templates |

## Sub-Agents

Simple rule: **if it's easy but would pollute the main agent's context, hand it to a sub-agent.**

- Parallel literature search (one agent per sub-direction)
- Code implementation from a clear spec
- Experiment deployment + monitoring
- Figure generation (matplotlib)
- LaTeX compilation
- Review calls to Codex MCP (reads all files, composes full-text prompt, returns review)

Everything else — idea evaluation, method design, experiment planning, result interpretation, paper narrative — the main agent does directly.

## Venue Support

Set once in `project.json`. Affects reviewer persona, scoring criteria, LaTeX template.

Included: `neurips`, `iclr`, `icml`, `cvpr`, `acl`, `aaai`, `nature-communications`, `prl`

Venue profiles in `references/venues/`. LaTeX templates in `templates/` (24 files covering all major venues).

## Auxiliary Skills

Standalone tools, not part of the main pipeline:

`/paper-poster` `/paper-slides` `/paper-illustration` `/grant-proposal` `/formula-derivation` `/proof-writer` `/dse-loop` `/comm-lit-review` `/arxiv` `/mermaid-diagram` `/feishu-notify` `/pixel-art`

## Input Templates

In `input-templates/` — structured templates to help you prepare inputs:

- `RESEARCH_BRIEF_TEMPLATE.md` — describe your research direction
- `EXPERIMENT_PLAN_TEMPLATE.md` — structure your experiment design
- `NARRATIVE_REPORT_TEMPLATE.md` — organize results into a narrative
- `PAPER_PLAN_TEMPLATE.md` — plan your paper structure

## Directory Structure

```
PRS/
├── CLAUDE.md                  # Harness core (207 lines)
├── README.md                  # This file
├── skills/
│   ├── prs-init/              # /prs-init
│   ├── prs/                   # /prs
│   └── prs-paper/             # /prs-paper
├── modules/                   # Context modules (10 files)
├── references/                # Venue profiles, writing principles, citation discipline
├── templates/                 # LaTeX venue templates (24 files)
├── input-templates/           # User input templates (4 files)
├── tools/                     # arxiv_fetch.py
├── auxiliary/                 # 12 standalone skills
└── archive/                   # Original skills (fallback reference)
```

## Migration from Old PRS

If you have a project with `PROJECT.md` or old `progress.json` (with `current_phase` / `auto_proceed` / `human_checkpoint`):

1. Run `/prs-init` in the project directory
2. It detects old format, maps to the new 4-stage model, builds artifacts registry
3. Existing artifact files are auto-detected via canonical + legacy path fallback
