# Experiment Context Module — Planning, Implementation, Deployment

This module covers the `experiment_plan`, `implementation`, and `experiments` phases. It turns a refined research proposal into a claim-driven experiment roadmap, implements the code, deploys to GPUs, monitors progress, and collects/analyzes results.

## Inputs

This module expects one or more of:

1. **`refine-logs/FINAL_PROPOSAL.md`** — method description with problem anchor, dominant contribution, and method details
2. **`refine-logs/REFINEMENT_REPORT.md`** — summary of the refinement process
3. **`refine-logs/REVIEW_SUMMARY.md`** — aggregated reviewer feedback
4. **`refine-logs/EXPERIMENT_PLAN.md`** (for implementation/experiments phases) — claim-driven experiment roadmap
5. **`refine-logs/EXPERIMENT_TRACKER.md`** (for implementation/experiments phases) — run-by-run execution table
6. **`IDEA_REPORT.md`** — fallback if refine-logs don't exist

If none exist, derive context from the user's prompt or ask what experiments to implement.

## Outputs

| Phase | Output Files |
|---|---|
| experiment_plan | `refine-logs/EXPERIMENT_PLAN.md`, `refine-logs/EXPERIMENT_TRACKER.md` |
| implementation | `code/` directory with experiment scripts |
| experiments | `results/` directory (JSON/CSV), updated `refine-logs/EXPERIMENT_TRACKER.md`, `refine-logs/EXPERIMENT_RESULTS.md` |

---

## Sub-phase: experiment_plan

### Goal

Turn a stable proposal into a **claim -> evidence -> run order** roadmap. The goal is NOT to generate a giant benchmark wishlist. The goal is to support four things:

1. The method actually solves the anchored problem
2. The dominant contribution is real and focused
3. The method is elegant enough that extra complexity is unnecessary
4. Any frontier-model-era component is genuinely useful, not decorative

### Step 1: Load the Proposal Context

Read existing files if they exist:

- `refine-logs/FINAL_PROPOSAL.md`
- `refine-logs/REVIEW_SUMMARY.md`
- `refine-logs/REFINEMENT_REPORT.md`

Extract:

- **Problem Anchor**
- **Dominant contribution**
- **Optional supporting contribution**
- **Critical reviewer concerns**
- **Data / compute / timeline constraints**
- **Which frontier primitive is central, if any**

### Step 2: Freeze the Paper Claims

Before proposing experiments, write down the claims that must be defended.

- **Primary claim**: the main mechanism-level contribution
- **Supporting claim**: optional, only if it directly strengthens the main paper story
- **Anti-claim to rule out**: e.g. "the gain only comes from more parameters," "the gain only comes from a larger search space," or "the modern component is just decoration"
- **Minimum convincing evidence**: what would make each claim believable to a strong reviewer?

Prefer one dominant claim plus one supporting claim. Do not exceed two primary claims unless the paper truly has multiple inseparable claims.

### Step 3: Build the Experimental Storyline

Design the paper around a compact set of experiment blocks (max 5 core blocks). Default to the following blocks and delete any that are not needed:

1. **Main anchor result** — does the method solve the actual bottleneck?
2. **Novelty isolation** — does the dominant contribution itself matter?
3. **Simplicity / elegance check** — can a bigger or more fragmented version be avoided?
4. **Frontier necessity check** — if an LLM / VLM / Diffusion / RL-era component is central, is it actually the right tool?
5. **Failure analysis or qualitative diagnosis** — what does the method still miss?

For each block, decide whether it belongs in:

- **Main paper** — essential to defend the core claims
- **Appendix** — useful but non-blocking
- **Cut** — interesting, but not worth the paper budget

Prefer one strong baseline family over many weak baselines (max 3 baseline families). If a stronger modern baseline exists, use it instead of padding the list.

### Step 4: Specify Each Experiment Block

For every kept block, fully specify:

- **Claim tested**
- **Why this block exists**
- **Dataset / split / task**
- **Compared systems**: strongest baselines, ablations, and variants only
- **Metrics**: decisive metrics first, secondary metrics second
- **Setup details**: backbone, frozen vs trainable parts, key hyperparameters, training budget, seeds (default 3 seeds when stochastic variance matters)
- **Success criterion**: what outcome would count as convincing evidence?
- **Failure interpretation**: if the result is negative, what does it mean?
- **Table / figure target**: where this result should appear in the paper
- **Priority**: MUST-RUN / NICE-TO-HAVE

Special rules:

- A **simplicity check** should usually compare the final method against either an overbuilt variant or a tempting extra component that the paper intentionally rejects.
- A **frontier necessity check** should usually compare the chosen modern primitive against the strongest plausible simpler or older alternative.
- If the proposal is intentionally non-frontier, say so explicitly and skip the frontier block instead of forcing one.

### Step 5: Build the Execution Order

Use this milestone structure:

1. **Sanity stage** — data pipeline, metric correctness, one quick overfit or toy split
2. **Baseline stage** — reproduce the strongest baseline(s)
3. **Main method stage** — run the final method on the primary setting
4. **Decision stage** — run the decisive ablations for novelty, simplicity, and frontier necessity
5. **Polish stage** — robustness, qualitative figures, appendix extras

For each milestone, estimate:

- Compute cost
- Expected turnaround time
- Stop / go decision gate
- Risk and mitigation

Separate **must-run** from **nice-to-have** experiments.

### Step 6: Write the Outputs

#### Output Formats

**EXPERIMENT_PLAN.md** — use the following template:

```markdown
# Experiment Plan

**Problem**: [problem]
**Method Thesis**: [one-sentence thesis]
**Date**: [today]

## Claim Map
| Claim | Why It Matters | Minimum Convincing Evidence | Linked Blocks |
|-------|-----------------|-----------------------------|---------------|
| C1    | ...             | ...                         | B1, B2        |

## Paper Storyline
- Main paper must prove:
- Appendix can support:
- Experiments intentionally cut:

## Experiment Blocks

### Block 1: [Name]
- Claim tested:
- Why this block exists:
- Dataset / split / task:
- Compared systems:
- Metrics:
- Setup details:
- Success criterion:
- Failure interpretation:
- Table / figure target:
- Priority: MUST-RUN / NICE-TO-HAVE

### Block 2: [Name]
...

## Run Order and Milestones
| Milestone | Goal | Runs | Decision Gate | Cost | Risk |
|-----------|------|------|---------------|------|------|
| M0        | ...  | ...  | ...           | ...  | ...  |

## Compute and Data Budget
- Total estimated GPU-hours:
- Data preparation needs:
- Human evaluation needs:
- Biggest bottleneck:

## Risks and Mitigations
- [Risk]:
- [Mitigation]:

## Final Checklist
- [ ] Main paper tables are covered
- [ ] Novelty is isolated
- [ ] Simplicity is defended
- [ ] Frontier contribution is justified or explicitly not claimed
- [ ] Nice-to-have runs are separated from must-run runs
```

**EXPERIMENT_TRACKER.md** — use the following template:

```markdown
# Experiment Tracker

| Run ID | Milestone | Purpose | System / Variant | Split | Metrics | Priority | Status | Notes |
|--------|-----------|---------|------------------|-------|---------|----------|--------|-------|
| R001   | M0        | sanity  | ...              | ...   | ...     | MUST     | TODO   | ...   |
```

### Planning Rules

- Prefer a compact paper story — design the main table first, then add only the ablations that defend it
- Defend simplicity explicitly — include a deletion study or a stronger-but-bloated variant comparison
- Defend frontier choices explicitly — prove the modern primitive beats the strongest simpler alternative
- Prefer strong baselines over long baseline lists
- Separate must-run from nice-to-have — do not let appendix ideas delay core evidence
- Reuse proposal constraints — do not invent unrealistic budgets or data assumptions
- Do not fabricate results — plan evidence, do not claim evidence

---

## Sub-phase: implementation

### Goal

Take the experiment plan and turn it into working, reviewed experiment code ready for deployment. This bridges planning and execution.

### Step 1: Parse the Experiment Plan

Read `EXPERIMENT_PLAN.md` and extract:

1. **Run order and milestones** — which experiments run first (sanity -> baseline -> main -> ablation -> polish)
2. **For each experiment block:**
   - Dataset / split / task
   - Compared systems and variants
   - Metrics to compute
   - Setup details (backbone, hyperparameters, seeds)
   - Success criterion
   - Priority (MUST-RUN vs NICE-TO-HAVE)
3. **Compute budget** — total estimated GPU-hours
4. **Method details** from `FINAL_PROPOSAL.md` — what exactly to implement

### Step 2: Implement Experiment Code

If a base repo is specified, clone it first and extend it. Otherwise scan the project for existing scripts, model code, data loaders -- reuse as much as possible.

For each milestone (in order: sanity -> baselines -> main -> ablations), implement:
- Training scripts with proper argparse (all hyperparameters configurable)
- Evaluation scripts computing the specified metrics
- Data loading / preprocessing if needed
- Baseline implementations if not already present
- Fixed random seeds for reproducibility
- Results saved to JSON/CSV for later analysis
- Proper logging (wandb if configured in CLAUDE.md)

**Self-review before deploying**: all hyperparameters from plan reflected in argparse? Random seed fixed? Results saved as JSON/CSV? Code matches FINAL_PROPOSAL.md?

### Step 3: Cross-Model Code Review (via Codex MCP)

Before deploying, send the experiment code to the reviewer model for review:

```
mcp__codex__codex:
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    Review the following experiment implementation for correctness.

    ## Experiment Plan:
    [paste key sections from EXPERIMENT_PLAN.md]

    ## Method Description:
    [paste from FINAL_PROPOSAL.md]

    ## Implementation:
    [paste the experiment scripts]

    Check for:
    1. Does the code correctly implement the method described in the proposal?
    2. Are all hyperparameters from the plan reflected in the code?
    3. Are there any logic bugs (wrong loss function, incorrect data split, missing eval)?
    4. Is the evaluation metric computed correctly?
    5. **CRITICAL: Does evaluation use the dataset's actual ground truth labels — NOT another model's output as ground truth?** This is a common and severe bug.
    6. Any potential issues (OOM risk, numerical instability, missing seeds)?

    For each issue found, specify: CRITICAL / MAJOR / MINOR and the exact fix.
```

**On review results:**
- **No CRITICAL issues** — proceed to deployment
- **CRITICAL issues found** — fix them, then re-submit for review (max 2 rounds)
- **Codex MCP unavailable** — skip silently, proceed to deployment (graceful degradation)

### Step 4: Sanity Check

Before deploying the full experiment suite, run the sanity-stage experiment first:

- Training loop runs without errors
- Metrics are computed and saved correctly
- GPU memory usage is within bounds
- Output format matches expectations

If sanity fails, fix the code, re-run. Do not proceed to full deployment with broken code.

### Implementation Rules

- Reuse existing code — scan the project before writing new scripts; extend, don't duplicate
- All hyperparameters must be configurable via argparse; random seeds must be fixed and controllable
- Update `EXPERIMENT_TRACKER.md` to reflect real status after each run completes

---

## Sub-phase: experiments

This sub-phase covers deployment, monitoring, and result collection/analysis.

### Deployment

#### Step 1: Detect Environment

Read the project's `CLAUDE.md` to determine the experiment environment:

- **Local GPU**: Look for local CUDA/MPS setup info
- **Remote server**: Look for SSH alias, conda env, code directory

If no server info is found in `CLAUDE.md`, ask the user.

#### Step 2: Pre-flight GPU Check

Check GPU availability: `nvidia-smi --query-gpu=index,memory.used,memory.total --format=csv,noheader` (prefix with `ssh <server>` for remote). Free GPU = memory.used < 500 MiB. For Mac MPS: `python -c "import torch; print(torch.backends.mps.is_available())"`. **ALWAYS check GPU availability first -- never blindly assign GPUs.**

#### Step 3: Sync Code (Remote Only)

Check `CLAUDE.md` for `code_sync` setting. **rsync (default)**: `rsync -avz --include='*.py' --exclude='*' <local_src>/ <server>:<remote_dst>/` — only sync necessary files, NOT data/checkpoints. **git**: push local changes, pull on server.

#### Step 4: W&B Integration (when `wandb: true` in CLAUDE.md)

**Skip entirely if `wandb` is not set or is `false` in CLAUDE.md.**

If W&B is enabled: check scripts for existing `import wandb` (skip if present). Otherwise add `wandb.init(project=..., name=..., config={...})` + `wandb.log()` calls for: `train/loss`, `train/lr`, `eval/loss`, `eval/ppl`, `eval/accuracy`, `gpu/memory_used`, `speed/samples_per_sec`, plus any custom metrics. Call `wandb.finish()` at end. Verify login: `ssh <server> "wandb status"`.

#### Step 5: Deploy Experiments

**Remote (via SSH + screen)** — each experiment gets a dedicated screen session with GPU binding:
```bash
ssh <server> "screen -dmS <exp_name> bash -c '\
  eval \"\$(<conda_path>/conda shell.bash hook)\" && \
  conda activate <env> && \
  CUDA_VISIBLE_DEVICES=<gpu_id> python <script> <args> 2>&1 | tee <log_file>'"
```

**Local**: `CUDA_VISIBLE_DEVICES=<gpu_id> python <script> <args> 2>&1 | tee <log_file>` (use `run_in_background: true` for long jobs).

#### Step 6: Verify and Report

Verify: `ssh <server> "screen -ls"` (remote) or check process + GPU allocation (local). Report: which GPU, which screen/process, what command, estimated time. Launch parallel on different GPUs following the plan's milestone order. **MAX_PARALLEL_RUNS = 4** — do not exceed 4 concurrent experiments (limited by available GPUs).

#### Feishu Notification (if configured)

After deployment, check if `~/.claude/feishu.json` exists and mode is not `"off"`. If configured, send a `deployment_started` notification with experiment names, servers, and estimated completion time. At completion, send `experiments_done` with result summary. If absent or off, skip silently.

### Monitoring

#### Collecting Status

1. **Check running sessions**: `ssh <server> "screen -ls"`
2. **Capture screen output**: `ssh <server> "screen -S <name> -X hardcopy /tmp/screen_<name>.txt && tail -50 /tmp/screen_<name>.txt"` (if hardcopy fails, check log files from tee)
3. **Check JSON results**: `ssh <server> "ls -lt <results_dir>/*.json 2>/dev/null | head -20"` then fetch and parse
4. **W&B metrics** (when `wandb: true`): use `wandb.Api()` to list runs, pull history (`scan_history`), pull summary. Extract: training loss curve (converging/diverging/plateauing?), eval metrics, LR schedule, GPU memory, run status. Include dashboard link: `https://wandb.ai/<entity>/<project>/runs/<run_id>`

#### Summarize and Interpret

Present results as comparison table (Experiment / Metric / Delta vs Baseline / Status). Compare against correct baselines (same config). Flag unexpected results (negative delta, NaN, divergence). Note if experiments are still running. If results look wrong, check training logs before concluding. Always show raw numbers before interpretation.

### Result Collection and Analysis

1. **Locate results**: find all JSON/CSV files in `figures/`, `results/`, or project-specific output directories. Parse into structured data.
2. **Build comparison table**: organize by independent variables (model, hyperparams, data config) vs dependent variables (primary + secondary metrics). Always compute delta vs baseline.
3. **Statistical analysis**: multiple seeds -> report mean +/- std; parameter sweep -> identify trends (monotonic, U-shaped, plateau); flag outliers.
4. **Generate insights**: for each finding structure as Observation (data + numbers) -> Interpretation (why) -> Implication (for research question) -> Next step (what experiment would test this).
5. **Update documentation**: update `refine-logs/EXPERIMENT_TRACKER.md` Status and Notes. Write `refine-logs/EXPERIMENT_RESULTS.md` with: results by milestone (M0-M3 tables), summary (X/Y must-run completed, main result positive/negative/inconclusive, ready for review_loop YES/NO).

**Analysis output format**: always include (1) raw data table, (2) key findings (numbered, concise), (3) suggested next experiments.

---

## Quality Constraints and Anti-Patterns

### Critical Rules (all sub-phases)

- **CRITICAL — Evaluation must use dataset ground truth.** ALWAYS compare model predictions against the dataset's actual ground truth labels/targets — NEVER use another model's output as ground truth. Triple-check: (1) ground truth comes from the dataset split, not from a baseline/backbone model, (2) evaluation metrics are computed against the same ground truth for all methods, (3) if the task has official eval scripts, use those.
- **Every experiment must defend a claim.** If it does not change a reviewer belief, cut it.
- **Sanity first.** Never deploy a full suite without verifying the sanity stage passes.
- **Follow the plan.** Do not invent experiments not in EXPERIMENT_PLAN.md.
- **Save everything as JSON/CSV.** The review loop needs parseable results, not just terminal output.
- **Budget awareness.** Track GPU-hours against the plan's budget. Warn if approaching the limit.
- **Don't wait forever.** If an experiment exceeds 2x its estimated time, flag it and move on.
- **Always show raw numbers before interpretation.** Structure insights as: Observation -> Interpretation -> Implication -> Next step.

### Anti-Patterns

- Giant benchmark wishlists with no claim linkage
- Padding the baseline list with many weak baselines instead of a few strong ones
- Letting appendix/nice-to-have experiments delay must-run core evidence
- Using another model's output as evaluation ground truth
- Deploying a full experiment suite without a passing sanity stage
- Writing all code from scratch instead of reusing existing scripts
- Saving results only as terminal output instead of JSON/CSV
- Inventing experiments not specified in the plan

## Verification Criteria

### experiment_plan -> implementation
- `refine-logs/EXPERIMENT_PLAN.md` exists and contains claim map, experiment blocks, and run order
- `refine-logs/EXPERIMENT_TRACKER.md` exists with all planned runs listed

### implementation -> experiments
- Code directory has `.py` files implementing the planned experiments
- All hyperparameters from the plan are reflected in argparse
- Random seeds are fixed and controllable
- Results are saved in parseable format (JSON/CSV)
- Code matches `FINAL_PROPOSAL.md`'s method description
- Sanity test has passed

### experiments -> review_loop
- `results/` directory has parseable data files (JSON/CSV)
- `refine-logs/EXPERIMENT_TRACKER.md` is updated with actual statuses and results
- Must-run experiments are completed (or explicitly flagged as blocked with reason)
- Results can be organized into comparison tables with delta vs baseline

### Large File Handling

If the Write tool fails due to file size, immediately retry using Bash (`cat << 'EOF' > file`) to write in chunks. Do NOT ask the user for permission — just do it silently.
