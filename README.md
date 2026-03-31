# PRS — Personal Research System

A state-driven thin harness for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) doing long-running scientific research. From idea to submission, one agent owns the story.

[English](README.md) | [中文](README_CN.md)

> PRS is a restructured, streamlined fork of [ARIS](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep). Same cross-model review architecture, but reorganized around a single stateful agent (`/prs`) instead of many independent skills.

## What Is This

PRS gives Claude Code the structure it needs for long-running research: state tracking (`progress.json`), session recovery (`session.json`), domain knowledge modules, and a clear 4-stage pipeline. The main agent does the research — PRS just keeps it organized.

**Key difference from ARIS:** Instead of 31 independent slash commands, PRS has 3 entry points (`/prs-init`, `/prs`, `/prs-paper`) and a single `next_action` that drives all progress. Less to learn, same capabilities.

## Quick Start

### Prerequisites

1. **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** installed and working
2. **[Codex CLI](https://github.com/openai/codex)** — used as MCP server for cross-model review (GPT-5.4 reviews what Claude writes)
3. **Node.js** ≥ 18 (for Codex CLI)
4. **Python 3** (for `tools/arxiv_fetch.py` and experiment scripts)

### Step 1: Install PRS

```bash
git clone https://github.com/nerslm/PRS.git
cd PRS

# Install skills to Claude Code
cp -r skills/* ~/.claude/skills/

# Install auxiliary skills (optional standalone tools)
cp -r auxiliary/* ~/.claude/skills/
```

### Step 2: Set Up Codex MCP (Required for Review)

All review, refinement, and improvement loops use Codex MCP — Claude Code writes, GPT-5.4 reviews.

```bash
# Install Codex CLI
npm install -g @openai/codex

# Initial setup — select gpt-5.4 when prompted
codex setup

# Register as MCP server in Claude Code
claude mcp add codex -s user -- codex mcp-server
```

> **Important:** The reviewer model is set in `~/.codex/config.toml`, not in PRS files. Make sure it says `model = "gpt-5.4"` (recommended). Other options: `gpt-5.3-codex`, `gpt-5.2-codex`, `o3`.

### Step 3: Verify

```bash
claude
> /prs-init "your research direction"
```

If it creates `project.json` and `progress.json`, you're good to go.

### Step 4 (Optional): LaTeX Environment

Required only for the **paper** stage (writing + compiling PDF):

```bash
# Ubuntu/Debian
sudo apt install texlive-full latexmk poppler-utils

# macOS
brew install --cask mactex
brew install poppler

# Verify
latexmk --version && pdftotext -v
```

> If you only need idea discovery + experiments + review, skip this.

## Usage

```bash
# Initialize a new research project
/prs-init "your research direction"              # interactive: asks venue, autonomy mode
/prs-init "your direction" — venue: neurips       # skip prompts

# Continue research (reads state, picks up where you left off)
/prs

# With intent (guides the next action)
/prs "refine my method"
/prs "run experiments"
/prs "start writing the paper"

# Jump directly to paper writing
/prs-paper
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
| **idea** | Literature survey → brainstorm ideas (Codex MCP) → novelty check → refine proposal | `IDEA_REPORT.md`, `refine-logs/FINAL_PROPOSAL.md` |
| **experiment** | Design experiments → implement code → deploy to GPU → collect results | `EXPERIMENT_PLAN.md`, `code/`, `results/` |
| **review** | Autonomous review loop via Codex MCP → implement fixes → re-review (up to 4 rounds) | `AUTO_REVIEW.md` |
| **paper** | Plan outline → generate figures → write LaTeX → compile PDF → venue-aware improvement | `PAPER_PLAN.md`, `paper/main.pdf` |

## Autonomy Modes

Set during `/prs-init`. One parameter, three choices:

| Mode | Behavior |
|---|---|
| `full` | Runs the entire pipeline without pausing. Idea to paper, hands-off. |
| `gates` | Auto-advances within stages. Pauses at 3 decision points: idea selection, experiment plan, paper outline. |
| `manual` | Pauses after every major action for your review. |

---

## Setup Details

### Codex MCP (Required)

This is the core dependency. PRS uses **cross-model collaboration**: Claude Code executes the research, GPT-5.4 (via Codex MCP) acts as critical reviewer. This avoids single-model self-play blind spots.

```bash
npm install -g @openai/codex
codex setup                              # set model to gpt-5.4
claude mcp add codex -s user -- codex mcp-server
```

The MCP server exposes two tools used throughout PRS:
- `mcp__codex__codex` — send a review/brainstorm prompt to GPT-5.4
- `mcp__codex__codex-reply` — retrieve the response

### Alternative Model Combinations

Don't have OpenAI API access? PRS inherits ARIS's flexible model architecture. You can swap in other models as reviewer:

| Setup | Executor | Reviewer | Guide |
|---|---|---|---|
| **Default** | Claude Code | GPT-5.4 (Codex MCP) | This README |
| **llm-chat** | Claude Code | Any OpenAI-compatible API | See [LLM Mix & Match Guide](docs/LLM_API_MIX_MATCH_GUIDE.md) + [`mcp-servers/llm-chat/`](mcp-servers/llm-chat/) |
| **MiniMax** | Claude Code | MiniMax-M2.5 | See [MiniMax Guide](docs/MINIMAX_MCP_GUIDE.md) + [`mcp-servers/minimax-chat/`](mcp-servers/minimax-chat/) |
| **Codex→Claude** | Codex CLI | Claude Code | See [Codex+Claude Review Guide](docs/CODEX_CLAUDE_REVIEW_GUIDE.md) + [`mcp-servers/claude-review/`](mcp-servers/claude-review/) |

PRS includes 4 MCP servers in `mcp-servers/` for alternative setups — no external repos needed.

### LaTeX (For Paper Stage)

Required tools: `latexmk`, `pdflatex`/`xelatex`, `bibtex`, `pdftotext`.

```bash
# Ubuntu/Debian
sudo apt install texlive-full latexmk poppler-utils

# macOS
brew install --cask mactex && brew install poppler

# Verify
latexmk --version && pdftotext -v
```

PRS includes **24 LaTeX venue templates** in `templates/`:
- NeurIPS, ICLR, ICML, CVPR, ACL, AAAI, ACM MM
- Nature Communications, PRL
- Each with proper style files and BibTeX configs

### Gemini API (For AI Illustrations, Optional)

The `/paper-illustration` auxiliary skill uses Google Gemini to generate publication-quality architecture diagrams. Only needed if you want AI-generated method figures.

```bash
# Get key from https://aistudio.google.com/apikey
export GEMINI_API_KEY="your-key-here"

# Add to shell profile for persistence
echo 'export GEMINI_API_KEY="your-key-here"' >> ~/.bashrc
```

> **Free alternative:** Use `/mermaid-diagram` instead — generates diagrams without any API key.

### GPU Server (For Auto-Experiments, Optional)

When the review loop says "run an ablation", Claude Code can automatically deploy experiments to your GPU server. Add server info to your project's `CLAUDE.md`:

```markdown
## Remote Server

- SSH: `ssh my-gpu-server` (key-based auth, no password)
- GPU: 4x A100
- Conda env: `research` (Python 3.10 + PyTorch)
- Activate: `eval "$(/opt/conda/bin/conda shell.bash hook)" && conda activate research`
- Code directory: `/home/user/experiments/`
- Use `screen` for background jobs
```

If you're already on the GPU machine:

```markdown
## GPU Environment

- This machine has direct GPU access (no SSH needed)
- GPU: 4x A100 80GB
- Activate: `conda activate research`
- Code directory: `/home/user/experiments/`
```

**No server?** Review and writing skills still work. Only experiment deployment is skipped.

### Feishu/Lark Notifications (Optional)

Get mobile notifications when experiments finish or reviews score. **Off by default — zero impact when unconfigured.**

**Push only (5 min):** Create a Feishu group bot → copy webhook URL → save to `~/.claude/feishu.json`:

```json
{
  "mode": "push",
  "webhook_url": "https://open.feishu.cn/open-apis/bot/v2/hook/YOUR_WEBHOOK_ID"
}
```

**Interactive:** Additionally deploy [feishu-claude-code](https://github.com/joewongjc/feishu-claude-code) bridge for bidirectional chat (approve/reject from phone). See [ARIS Feishu setup guide](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep#-feishulark-integration-optional) for details.

### Zotero Integration (Optional)

Search your paper library, read annotations, export BibTeX during literature survey.

```bash
uv tool install zotero-mcp-server
claude mcp add zotero -s user -- zotero-mcp -e ZOTERO_LOCAL=true
```

See [zotero-mcp](https://github.com/54yyyu/zotero-mcp) for Web API setup.

### Obsidian Integration (Optional)

Search your research notes vault during literature survey.

```bash
claude mcp add obsidian-vault -s user -- npx @bitbonsai/mcpvault@latest /path/to/your/vault
```

See [mcpvault](https://github.com/bitbonsai/mcpvault) for details.

---

## Auto-Allow for Overnight Runs

To run without clicking permission prompts, add to `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "mcp__codex__codex",
      "mcp__codex__codex-reply",
      "Write",
      "Edit",
      "Skill(prs)",
      "Skill(prs-paper)"
    ]
  }
}
```

---

## Configuration

All config lives in `project.json` (created by `/prs-init`):

```json
{
  "version": 1,
  "title": "...",
  "direction": "...",
  "venue": "neurips",
  "created": "2026-03-31",
  "config": {
    "autonomy": "gates",
    "reviewer_model": "gpt-5.4",
    "arxiv_download": false,
    "pilot_max_hours": 2,
    "max_review_rounds": 4,
    "max_refine_rounds": 5,
    "max_paper_improve_rounds": 4,
    "score_threshold_refine": 9,
    "score_threshold_review": 8.5
  }
}
```

| Parameter | Default | Description |
|---|---|---|
| `autonomy` | `gates` | `full` / `gates` / `manual` — controls when to pause |
| `reviewer_model` | `gpt-5.4` | Model used via Codex MCP for review |
| `arxiv_download` | `false` | Download arXiv PDFs during literature survey |
| `pilot_max_hours` | `2` | Max hours per pilot experiment |
| `max_review_rounds` | `4` | Review loop iterations before stopping |
| `max_refine_rounds` | `5` | Idea refinement iterations |
| `max_paper_improve_rounds` | `4` | Paper improvement iterations |
| `score_threshold_refine` | `9` | Refinement stops when score >= this |
| `score_threshold_review` | `8.5` | Review loop stops when score >= this |

---

## Venue Support

Set once in `project.json`. Affects reviewer persona, scoring criteria, LaTeX template.

Included: `neurips`, `iclr`, `icml`, `cvpr`, `acl`, `aaai`, `acm`, `nature-communications`, `prl`

Venue profiles in `references/venues/`. Templates in `templates/` (24 files).

## Context Modules

Domain knowledge distilled into focused modules. The main agent reads them when working on each stage:

| Module | Content |
|---|---|
| `research.md` | Literature search, Codex brainstorm prompts, novelty verification, pilot experiment design |
| `refine.md` | Problem Anchor pattern, 7-dimension scoring, Codex review templates |
| `experiment.md` | Claim-driven experiments, SSH+screen deployment, code review, W&B integration |
| `review.md` | Review loop (Codex MCP + LLM fallback), crash recovery, fix prioritization |
| `paper-*.md` | Claims-Evidence Matrix, section guidelines, DBLP citations, De-AI polish, latexmk, venue templates |

## Auxiliary Skills

Standalone tools, not part of the main pipeline. Invoke anytime:

| Skill | Description | Requires |
|---|---|---|
| `/paper-poster` | Conference poster (tcbposter → A0/A1 PDF) | LaTeX |
| `/paper-slides` | Conference slides (beamer → PDF + PPTX) | LaTeX |
| `/paper-illustration` | AI architecture diagrams via Gemini | `GEMINI_API_KEY` |
| `/grant-proposal` | Grant proposal drafting (NSF/NSFC/ERC/KAKENHI/...) | Codex MCP |
| `/formula-derivation` | Formula development and verification | — |
| `/proof-writer` | Rigorous theorem/lemma proof drafting | — |
| `/dse-loop` | Design space exploration (gem5, Yosys, etc.) | — |
| `/comm-lit-review` | Communications domain literature review | — |
| `/arxiv` | Search/download arXiv papers | — |
| `/mermaid-diagram` | Mermaid diagrams (20+ types, no API key) | — |
| `/feishu-notify` | Feishu/Lark notifications | Feishu webhook |
| `/pixel-art` | Pixel art SVG for docs/slides | — |

## Input Templates

In `input-templates/` — structured templates for preparing inputs:

- `RESEARCH_BRIEF_TEMPLATE.md` — describe your research direction
- `EXPERIMENT_PLAN_TEMPLATE.md` — structure your experiment design
- `NARRATIVE_REPORT_TEMPLATE.md` — organize results for paper writing
- `PAPER_PLAN_TEMPLATE.md` — plan your paper structure

---

## Directory Structure

```
PRS/
├── CLAUDE.md                  # Harness core — the main agent reads this
├── README.md                  # This file
├── skills/
│   ├── prs-init/              # /prs-init — initialize a project
│   ├── prs/                   # /prs — main research loop
│   └── prs-paper/             # /prs-paper — jump to paper stage
├── modules/                   # Context modules (10 files)
│   ├── research.md            # Idea stage: survey + brainstorm
│   ├── refine.md              # Idea stage: proposal refinement
│   ├── experiment.md          # Experiment stage
│   ├── review.md              # Review stage
│   ├── paper-common.md        # Paper stage: shared
│   ├── paper-plan.md          # Paper stage: outline
│   ├── paper-figures.md       # Paper stage: figures
│   ├── paper-write.md         # Paper stage: LaTeX
│   ├── paper-compile.md       # Paper stage: build PDF
│   └── paper-improve.md       # Paper stage: improvement loop
├── references/                # Venue profiles, writing principles, citation discipline
├── templates/                 # LaTeX venue templates (24 files)
├── input-templates/           # User input templates (4 files)
├── tools/                     # arxiv_fetch.py (Python, stdlib only)
├── mcp-servers/               # Built-in MCP servers
│   ├── llm-chat/              # Generic OpenAI-compatible reviewer
│   ├── minimax-chat/          # MiniMax reviewer
│   ├── claude-review/         # Codex executor + Claude reviewer bridge
│   └── feishu-bridge/         # Feishu bidirectional bridge
├── docs/                      # Setup guides for alternative models
├── auxiliary/                 # 12 standalone skills
└── archive/                   # Original ARIS skills (fallback reference)
```

## Dependencies Summary

| Dependency | Required? | What For | Install |
|---|---|---|---|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | **Yes** | Execution backbone | `npm install -g @anthropic-ai/claude-code` |
| [Codex CLI](https://github.com/openai/codex) | **Yes** | Cross-model review (GPT-5.4) | `npm install -g @openai/codex` + `codex setup` |
| Node.js ≥ 18 | **Yes** | For Codex CLI | System package manager |
| Python 3 | **Yes** | arxiv_fetch.py, experiment scripts | System / conda |
| LaTeX (texlive) | Paper stage only | Compile PDF | `apt install texlive-full latexmk poppler-utils` |
| Gemini API Key | Optional | `/paper-illustration` only | [aistudio.google.com/apikey](https://aistudio.google.com/apikey) |
| SSH + screen/tmux | Optional | Remote experiment deployment | System packages |
| W&B | Optional | Experiment tracking | `pip install wandb` |
| Zotero MCP | Optional | Search your paper library | [zotero-mcp](https://github.com/54yyyu/zotero-mcp) |
| Obsidian MCP | Optional | Search your research notes | [mcpvault](https://github.com/bitbonsai/mcpvault) |
| Feishu webhook | Optional | Mobile notifications | Feishu group bot |

## Relationship to ARIS

PRS is built on top of [ARIS](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep) (Auto-claude-code-research-in-sleep). The key differences:

- **Single stateful agent** — PRS uses `/prs` with `next_action` in `progress.json` instead of 31 independent slash commands
- **State-driven** — `project.json` + `progress.json` + `session.json` provide full session recovery
- **Context modules** — domain knowledge extracted into focused `.md` files instead of embedded in skill prompts
- **Same review architecture** — still uses Codex MCP for cross-model Claude × GPT-5.4 collaboration
- **Full ARIS archive** — original skills preserved in `archive/` as fallback reference

## License

MIT

## Acknowledgements

- [ARIS](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep) — the original autonomous research system this project builds on
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — Anthropic's CLI for Claude
- [Codex CLI](https://github.com/openai/codex) — OpenAI's CLI, used as MCP server for cross-model review
- [zotero-mcp](https://github.com/54yyyu/zotero-mcp) — Zotero MCP server
- [mcpvault](https://github.com/bitbonsai/mcpvault) — Obsidian vault MCP server
- [feishu-claude-code](https://github.com/joewongjc/feishu-claude-code) — Feishu ↔ Claude Code bridge
- [PaperBanana](https://github.com/dwzhu-pku/PaperBanana) — Academic illustration framework (used by `/paper-illustration`)
