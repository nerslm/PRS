# Review Context Module — Autonomous Research Review Loop

This module covers the `review_loop` phase. It drives an autonomous cycle of external review, structured parsing, targeted fixes, experiment deployment, and documentation — repeating until the reviewer gives a positive assessment or the maximum number of rounds is reached.

## Inputs

- **Research artifacts**: code, experiment results (in `results/`), figures, analysis scripts, project narrative documents
- **Prior review documents**: any existing `AUTO_REVIEW.md` from earlier runs
- **Project state**: `project.json` (config: `autonomy`, `reviewer_model`, `max_review_rounds`, `score_threshold_review`, `venue`), `progress.json` (phase state for crash recovery), `session.json` (thread IDs, next actions)
- **Venue profile**: `references/venues/{venue_slug}.md` — contains the reviewer persona and evaluation criteria. The venue slug is derived from `project.json`'s `venue` field (lowercased). If the venue file does not exist, fall back to `references/venues/neurips.md`.
- **Memory files and project notes**: any accumulated context from prior phases

## Outputs

- **`AUTO_REVIEW.md`** — cumulative review log (one section per round, plus final summary and method description)
- **Updated `progress.json`** — `stages.review` object updated each round with `round`, `score`, `thread_id`, `status`
- **Updated `session.json`** — thread IDs and critical context for session handoff
- **Code/analysis changes** — fixes implemented during Phase C
- **New experiment results** — from any experiments launched during the loop

---

## Loop Structure (Phase A through E)

The loop runs up to `max_review_rounds` times (from `project.json` config). Each iteration proceeds through five phases:

### Initialization

1. **Check `progress.json`** for crash recovery (see Crash Recovery section below).
2. Read project narrative documents, memory files, and any prior review documents.
3. Read recent experiment results (check output directories, logs).
4. Identify current weaknesses and open TODOs from prior reviews.
5. Initialize round counter = 1 (unless recovered from state).
6. Create/update `AUTO_REVIEW.md` with header and timestamp.

### Phase A: Review

**The review call should be delegated to a sub-agent** to avoid polluting the main agent's context with the full text of all project files. The sub-agent reads everything, composes the prompt, calls Codex, and returns the raw review text.

**Before composing the prompt**, read `references/venues/{venue_slug}.md` and use its **Prompt Fragment** section as the reviewer instructions. If the file does not exist, fall back to `references/venues/neurips.md`.

#### What the reviewer must see (full content, NOT summaries)

The reviewer can only judge what it receives. Send **actual file content**, not your interpretation:

1. **Method proposal** — read and paste `refine-logs/FINAL_PROPOSAL.md` in full
2. **Experiment results** — read `results/*.json` or `results/*.csv`, format as tables with actual numbers
3. **Key code logic** — read and paste core files (model definition, training loop, evaluation script) — the reviewer needs to check if the implementation matches the claims
4. **Experiment plan** — paste `refine-logs/EXPERIMENT_PLAN.md` so reviewer understands what was supposed to be tested
5. **Known weaknesses** — paste any prior review feedback from `AUTO_REVIEW.md`
6. **If paper exists** — concatenate `paper/sections/*.tex` and send the full paper text instead of items 1-4

**Round 1 — Spawn sub-agent for initial review:**

The main agent spawns a sub-agent to handle the file reading + Codex call:

```
Agent(
  description: "Review round N via Codex",
  prompt: "You are a review assistant. Your job:
    1. Read the venue profile at: {absolute_path}/references/venues/{venue_slug}.md
       Extract the Prompt Fragment section.
    2. Read the FULL content of these files from {project_root}:
       - refine-logs/FINAL_PROPOSAL.md (method proposal)
       - refine-logs/EXPERIMENT_PLAN.md (experiment design)
       - results/ directory — all JSON/CSV files (actual experiment data)
       - code/ directory — key .py files (model, train, eval scripts)
       - AUTO_REVIEW.md (prior review feedback, if exists)
       If paper/sections/*.tex exist, concatenate all sections as the full paper text and use that instead.
    3. Compose a single Codex MCP prompt containing ALL of the above content (full text, not summaries).
       Structure it as:
       ---
       [Round {N}/{MAX_ROUNDS} of autonomous review loop]

       ## Research Proposal (full text)
       {paste FINAL_PROPOSAL.md}

       ## Experiment Plan
       {paste EXPERIMENT_PLAN.md}

       ## Code (key files)
       {paste code files}

       ## Experiment Results
       {paste results as formatted tables}

       ## Prior Review Feedback
       {paste AUTO_REVIEW.md last round, or 'First round' if none}

       ## Review Instructions
       {paste venue Prompt Fragment}

       Be brutally honest. If the work is ready, say so clearly.
       ---
    4. Call mcp__codex__codex with config: {'model_reasoning_effort': 'xhigh'} and the composed prompt.
    5. Save the threadId.
    6. Return to me: the FULL raw review text (verbatim, do not summarize) + the threadId.

    Project config: venue={venue}, reviewer_model={reviewer_model}"
)
```

**Round 2+ — Spawn sub-agent for follow-up review:**

```
Agent(
  description: "Review round N follow-up via Codex",
  prompt: "You are a review assistant. Your job:
    1. Read the venue profile at: {absolute_path}/references/venues/{venue_slug}.md
       Extract the Prompt Fragment section (in case thread context was compressed).
    2. Read the updated files from {project_root}:
       - Any changed code files
       - Updated results (new JSON/CSV)
       - refine-logs/FINAL_PROPOSAL.md (if revised)
       If paper exists, concatenate paper/sections/*.tex for the full paper text.
    3. Call mcp__codex__codex-reply with:
       - threadId: {saved_thread_id}
       - config: {'model_reasoning_effort': 'xhigh'}
       - prompt containing:
         [Round N update]
         Since last review, we have:
         {list of changes with actual content, not summaries}
         Updated results: {paste actual data tables}

         ## Review Instructions (reminder)
         {paste venue Prompt Fragment}

         Please re-score and re-assess. Same format: Score, Verdict, Remaining Weaknesses, Minimum Fixes.
    4. Return: FULL raw review text (verbatim) + threadId."
)
```

ALWAYS use `config: {"model_reasoning_effort": "xhigh"}`. The sub-agent saves and returns the `threadId` so it can be passed to subsequent round sub-agents.

#### Main agent's role after receiving the review

The sub-agent returns the raw review text (~2K tokens). The main agent:
1. Reads the review carefully
2. If it needs to verify a specific point (e.g., "Theorem 3 has a gap"), reads that specific file
3. Makes judgment calls: what to fix, what to push back on, whether the score is fair
4. Proceeds to Phase B (parse) and Phase C (implement fixes)

### Phase B: Parse Assessment

**CRITICAL: Save the FULL raw response** from the external reviewer verbatim (store in a variable for Phase E). Do NOT discard or summarize — the raw text is the primary record.

Then extract structured fields:

- **Score** (numeric 1-10)
- **Verdict** ("ready" / "almost" / "not ready")
- **Action items** (ranked list of fixes)

**Loop control**: Continue until score >= `score_threshold_review` OR `max_review_rounds` exhausted. Only the numeric score decides — ignore verdict text.

### Human Checkpoint (if enabled)

**Skip this step if `autonomy` is `full` in `project.json`.** Only pause when `autonomy` is `gates` or `manual`.

When pausing, present the review results and wait for user input:

```
Round N/MAX_ROUNDS review complete.

Score: X/10 — [verdict]
Top weaknesses:
1. [weakness 1]
2. [weakness 2]
3. [weakness 3]

Suggested fixes:
1. [fix 1]
2. [fix 2]
3. [fix 3]

Options:
- Reply "go" or "continue" → implement all suggested fixes
- Reply with custom instructions → implement your modifications instead
- Reply "skip 2" → skip fix #2, implement the rest
- Reply "stop" → end the loop, document current state
```

Wait for the user's response. Parse their input:
- **Approval** ("go", "continue", "ok", "proceed"): proceed to Phase C with all suggested fixes
- **Custom instructions** (any other text): treat as additional/replacement guidance for Phase C. Merge with reviewer suggestions where appropriate
- **Skip specific fixes** ("skip 1,3"): remove those fixes from the action list
- **Stop** ("stop", "enough", "done"): terminate the loop, jump to Termination

### Feishu Notification (if configured)

After parsing the score, check if `~/.claude/feishu.json` exists and mode is not `"off"`:
- Send a `review_scored` notification: "Round N: X/10 — [verdict]" with top 3 weaknesses
- If **interactive** mode and verdict is "almost": send as checkpoint, wait for user reply on whether to continue or stop
- If config absent or mode off: skip entirely (no-op)

### Phase C: Implement Fixes (if not stopping)

**Core principle: take every reviewer concern seriously and address it fully.** Do not skip, minimize, or half-address reviewer feedback. If the reviewer says "add ablation X", actually run ablation X — don't just add a sentence explaining why it's unnecessary. If the reviewer says "the proof has a gap", fix the proof — don't just add a disclaimer.

For each action item (highest priority first):

1. **CRITICAL fixes first**: These are blockers. Drop everything else and fix them. If it requires new experiments, design and run them. If it requires rewriting theory, rewrite it.
2. **MAJOR fixes**: Implement fully. If the reviewer asks for a new experiment or analysis, actually do it — don't substitute with a weaker alternative unless the requested experiment is truly infeasible.
3. **MINOR fixes**: Address all of them. They're cheap and show the reviewer you care.
4. **New experiments**: If the reviewer requests experiments, deploy them (GPU via SSH + screen/tmux). Wait for results before the next review round — don't re-submit without the evidence the reviewer asked for.
5. **Analysis/figures**: Update tables, regenerate figures with new data, add requested visualizations.
6. **Documentation**: Record every fix in the round's section of AUTO_REVIEW.md with: what the reviewer said → what you did → evidence of the fix.

**What NOT to do**:
- Don't dismiss reviewer concerns with "we believe this is already addressed" unless you can point to specific evidence
- Don't skip requested experiments because they're "too expensive" — if truly infeasible, explain WHY with compute estimates and propose the closest feasible alternative
- Don't reframe reviewer concerns as non-issues — address the substance, not just the wording
- Don't re-submit to the reviewer until ALL action items from the current round are addressed

### Phase D: Wait for Results

If experiments were launched:
- Monitor remote sessions for completion
- Collect results from output files and logs

If an experiment takes > 30 minutes, launch it and continue with other fixes while waiting.

### Phase E: Document Round

Append to `AUTO_REVIEW.md` (see AUTO_REVIEW.md Format section below for exact template).

**Update `progress.json`** — write the `stages.review` object with current `round`, `score`, `thread_id`, and `status`.

Increment round counter and return to Phase A.

### Termination

When loop ends (positive assessment or max rounds):

1. Update `progress.json` `stages.review` with `"status": "done"`
2. Write final summary to `AUTO_REVIEW.md`
3. Update project notes with conclusions
4. **Write method/pipeline description** to `AUTO_REVIEW.md` under a `## Method Description` section — a concise 1-2 paragraph description of the final method, its architecture, and data flow. This serves as input for paper figure generation (so it can generate architecture diagrams automatically).
5. If stopped at max rounds without positive assessment:
   - List remaining blockers
   - Estimate effort needed for each
   - Suggest whether to continue manually or pivot
6. **Feishu notification** (if configured): Send `pipeline_done` with final score progression table

---

## Crash Recovery

Long-running loops may hit the context window limit, triggering automatic compaction. To survive this, state is persisted in `progress.json`'s `stages.review` object after each round.

### State Fields in progress.json

The `review` stage object contains these fields for crash recovery:

```json
{
  "stages": {
    "review": {
      "status": "in_progress",
      "round": 2,
      "score": 5.0,
      "thread_id": "019cd392-...",
      "last_verdict": "not ready",
      "pending_experiments": ["screen_name_1"],
      "timestamp": "2026-03-13T21:00:00",
      "summary": ""
    }
  }
}
```

**Write this state at the end of every Phase E** (after documenting the round). Overwrite each time — only the latest state matters.

**On completion** (positive assessment or max rounds), set `"status": "done"` so future invocations don't accidentally resume a finished loop.

### Recovery Logic on Initialization

1. Read `progress.json` and inspect the `stages.review` object:
   - If `status` is `"not_started"`: **fresh start** (normal case)
   - If `status` is `"done"`: **fresh start** (previous loop finished normally)
   - If `status` is `"in_progress"` AND `timestamp` is older than 24 hours: **fresh start** (stale state from a killed/abandoned run — reset the phase and start over)
   - If `status` is `"in_progress"` AND `timestamp` is within 24 hours: **resume**
     - Recover `round`, `thread_id`, `score`, `pending_experiments` from the stage object
     - Read `AUTO_REVIEW.md` to restore full context of prior rounds
     - If `pending_experiments` is non-empty, check if they have completed (e.g., check screen sessions)
     - Resume from the next round (round = saved round + 1)
     - Log: "Recovered from context compaction. Resuming at Round N."

---

## Review Prompt Templates

### Initial Review (Round 1) — Codex MCP

```
mcp__codex__codex:
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [Round N/MAX_ROUNDS of autonomous review loop]

    [Full research context: claims, methods, results, known weaknesses]
    [Changes since last round, if any]

    [INSERT the Prompt Fragment from the venue profile file here]

    Be brutally honest. If the work is ready, say so clearly.
```

### Continuation Review (Round 2+) — Codex MCP Reply

```
mcp__codex__codex-reply:
  threadId: [saved from round 1]
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    [Round N update]

    Since your last review, we have:
    1. [Action 1]: [result]
    2. [Action 2]: [result]
    3. [Action 3]: [result]

    Updated results table:
    [paste metrics]

    Please re-score and re-assess. Are the remaining concerns addressed?
    Same format: Score, Verdict, Remaining Weaknesses, Minimum Fixes.
```

---

## Generic LLM Fallback

When Codex MCP is unavailable, use any OpenAI-compatible API endpoint. This allows review via DeepSeek, MiniMax, Kimi, ZhiPu, SiliconFlow, or any other compatible provider.

### Supported Providers

| Provider | LLM_BASE_URL | LLM_MODEL |
|----------|--------------|-----------|
| **OpenAI** | `https://api.openai.com/v1` | `gpt-4o`, `o3` |
| **DeepSeek** | `https://api.deepseek.com/v1` | `deepseek-chat`, `deepseek-reasoner` |
| **MiniMax** | `https://api.minimax.chat/v1` | `MiniMax-M2.5` |
| **Kimi (Moonshot)** | `https://api.moonshot.cn/v1` | `moonshot-v1-8k`, `moonshot-v1-32k` |
| **ZhiPu (GLM)** | `https://open.bigmodel.cn/api/paas/v4` | `glm-4`, `glm-4-plus` |
| **SiliconFlow** | `https://api.siliconflow.cn/v1` | `Qwen/Qwen2.5-72B-Instruct` |

### MCP Tool Method (Preferred)

```
mcp__llm-chat__chat:
  system: "You are a senior ML reviewer (NeurIPS/ICML level)."
  prompt: |
    [Round N/MAX_ROUNDS of autonomous review loop]

    [Full research context: claims, methods, results, known weaknesses]
    [Changes since last round, if any]

    1. Score this work 1-10 for a top venue
    2. List remaining critical weaknesses (ranked by severity)
    3. For each weakness, specify the MINIMUM fix
    4. State clearly: is this READY for submission? Yes/No/Almost

    Be brutally honest. If the work is ready, say so clearly.
```

### Round 2+ via Generic LLM

```
mcp__llm-chat__chat:
  system: "You are a senior ML reviewer (NeurIPS/ICML level)."
  prompt: |
    [Round N/MAX_ROUNDS of autonomous review loop]

    ## Previous Review Summary (Round N-1)
    - Previous Score: X/10
    - Previous Verdict: [ready/almost/not ready]
    - Previous Key Weaknesses: [list]

    ## Changes Since Last Review
    1. [Action 1]: [result]
    2. [Action 2]: [result]

    ## Updated Results
    [paste updated metrics/tables]

    Please re-score and re-assess:
    1. Score this work 1-10 for a top venue
    2. List remaining critical weaknesses (ranked by severity)
    3. For each weakness, specify the MINIMUM fix
    4. State clearly: is this READY for submission? Yes/No/Almost

    Be brutally honest. If the work is ready, say so clearly.
```

### curl Fallback (when no MCP tool is available)

```bash
curl -s "${LLM_BASE_URL}/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${LLM_API_KEY}" \
  -d '{
    "model": "${LLM_MODEL}",
    "messages": [
      {"role": "system", "content": "You are a senior ML reviewer (NeurIPS/ICML level)."},
      {"role": "user", "content": "[Full review prompt]"}
    ],
    "max_tokens": 4096
  }'
```

Prefer MCP tool over curl when available. Use curl only as a last resort.

---

## Human Checkpoint Protocol

Controlled by `autonomy` in `project.json` config. When `gates` or `manual`, the loop pauses after Phase B of each round. When `full`, the loop runs without pausing.

### Presentation Format

```
Round N/MAX_ROUNDS review complete.

Score: X/10 — [verdict]
Top weaknesses:
1. [weakness 1]
2. [weakness 2]
3. [weakness 3]

Suggested fixes:
1. [fix 1]
2. [fix 2]
3. [fix 3]

Options:
- Reply "go" or "continue" → implement all suggested fixes
- Reply with custom instructions → implement your modifications instead
- Reply "skip 2" → skip fix #2, implement the rest
- Reply "stop" → end the loop, document current state
```

### Input Parsing

| User Input | Action |
|---|---|
| "go", "continue", "ok", "proceed" | Proceed to Phase C with all suggested fixes |
| Any other text | Treat as custom guidance; merge with reviewer suggestions |
| "skip 1,3" | Remove those numbered fixes from the action list |
| "stop", "enough", "done" | Terminate the loop, jump to Termination |

When `autonomy` is `full`, skip this step entirely — the loop runs fully autonomously.

---

## AUTO_REVIEW.md Format

Each round appends the following block:

```markdown
## Round N (timestamp)

### Assessment (Summary)
- Score: X/10
- Verdict: [ready/almost/not ready]
- Key criticisms: [bullet list]

### Reviewer Raw Response

<details>
<summary>Click to expand full reviewer response</summary>

[Paste the COMPLETE raw response from the external reviewer here — verbatim, unedited.
This is the authoritative record. Do NOT truncate or paraphrase.]

</details>

### Actions Taken
- [what was implemented/changed]

### Results
- [experiment outcomes, if any]

### Status
- [continuing to round N+1 / stopping]
```

### Final Summary (on Termination)

After the loop ends, append:

```markdown
## Final Summary

- Rounds completed: N
- Score progression: [round 1 score] → [round 2 score] → ...
- Final verdict: [ready/almost/not ready]
- Remaining blockers: [if any]

## Method Description

[1-2 paragraph description of the final method, architecture, and data flow.
Serves as input for paper figure generation.]
```

---

## Fix Prioritization Rules

When implementing fixes from the reviewer's action items, follow this priority order:

1. **Address ALL reviewer concerns** — do not skip or minimize any feedback
2. **CRITICAL/MAJOR fixes must be fully resolved** before re-submitting for review
3. **If the reviewer asks for experiments, run them** — don't substitute with weaker alternatives. If truly infeasible, explain with compute estimates and propose the closest feasible alternative
4. **Always implement metric additions** — cheap, high impact
5. **If an experiment takes > 30 minutes**, launch it asynchronously (screen/tmux) and continue with other fixes while waiting — but DO wait for results before the next review round
6. Process items in reviewer-ranked severity order within each priority tier
7. **Only flag for manual follow-up** if the fix genuinely requires resources you don't have (specific hardware, proprietary data, human annotation)

---

## Quality Constraints

- **Anti-hallucination citations**: When adding references during fixes, NEVER fabricate BibTeX. Use the DBLP -> CrossRef -> `[VERIFY]` chain: (1) `curl -s "https://dblp.org/search/publ/api?q=TITLE&format=json"` -> get key -> `curl -s "https://dblp.org/rec/{key}.bib"`, (2) if not found, `curl -sLH "Accept: application/x-bibtex" "https://doi.org/{doi}"`, (3) if both fail, mark with `% [VERIFY]`. Do NOT generate BibTeX from memory.
- **Honesty**: Include negative results and failed experiments. Do NOT hide weaknesses to game a positive score.
- **Implement before re-reviewing**: Fix issues BEFORE sending the next review round. Do not promise to fix things later.
- **Document EVERYTHING**: The review log (`AUTO_REVIEW.md`) should be self-contained and reproducible.
- **Update project notes after each round**, not just at the end.
- **Large file handling**: If the Write tool fails due to file size, immediately retry using Bash (`cat << 'EOF' > file`) to write in chunks. Do NOT ask the user for permission — just do it silently.

---

## Verification Criteria

Before transitioning out of review stage:

- `AUTO_REVIEW.md` exists and contains at least one complete round (Assessment + Raw Response + Actions + Results + Status)
- `progress.json` `stages.review` has `"status": "done"` with final `score` and `round` recorded
- Either the score >= `score_threshold_review`, OR `max_review_rounds` have been completed
- If max rounds reached without passing: remaining blockers are documented with effort estimates
- Method description section is written in `AUTO_REVIEW.md` for downstream paper figure generation
