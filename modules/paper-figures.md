# Paper Sub-module

> Also read `modules/paper-common.md` for shared inputs/outputs, quality constraints, and verification criteria.

## Sub-phase: paper_figures

### Scope: What This Skill Can and Cannot Do

| Category | Can auto-generate? | Examples |
|----------|-------------------|----------|
| **Data-driven plots** | Yes | Line plots (training curves), bar charts (method comparison), scatter plots, heatmaps, box/violin plots |
| **Comparison tables** | Yes | LaTeX tables comparing prior bounds, method features, ablation results |
| **Multi-panel figures** | Yes | Subfigure grids combining multiple plots (e.g., 3x3 dataset x method) |
| **Architecture/pipeline diagrams** | No — manual | Model architecture, data flow diagrams, system overviews. At best a rough TikZ skeleton. |
| **Generated image grids** | No — manual | Grids of generated samples (e.g., GAN/diffusion outputs). These come from running your model. |
| **Photographs / screenshots** | No — manual | Real-world images, UI screenshots, qualitative examples |

In practice, this skill handles ~60% of figures (all data plots + tables). The remaining ~40% (hero figure, architecture diagram, qualitative results) need to be created manually and placed in `figures/` before paper_write. The skill will detect these as "existing figures" and preserve them.

### Plotting Environment Setup

Create a shared style configuration script:

```python
# paper_plot_style.py — shared across all figure scripts
import matplotlib.pyplot as plt
import matplotlib
matplotlib.rcParams.update({
    'font.size': 10,                # FONT_SIZE — matches typical conference body text
    'font.family': 'serif',
    'font.serif': ['Times New Roman', 'Times', 'DejaVu Serif'],
    'axes.labelsize': 10,
    'axes.titlesize': 11,
    'xtick.labelsize': 9,
    'ytick.labelsize': 9,
    'legend.fontsize': 9,
    'figure.dpi': 300,              # DPI
    'savefig.dpi': 300,
    'savefig.bbox': 'tight',
    'savefig.pad_inches': 0.05,
    'axes.grid': False,
    'axes.spines.top': False,
    'axes.spines.right': False,
    'text.usetex': False,           # set True if LaTeX is available
    'mathtext.fontset': 'stix',
})

# Color palette
COLORS = plt.cm.tab10.colors  # or Set2, or colorblind-safe

def save_fig(fig, name, fmt='pdf'):
    """Save figure to figures/ with consistent naming."""
    fig.savefig(f'figures/{name}.{fmt}')
    print(f'Saved: figures/{name}.{fmt}')
```

Style options: `publication` (default, clean for print), `poster` (larger fonts), `slide` (bold colors). Color palette options: `tab10`, `Set2`, `colorblind` (deuteranopia-safe). Output format: `pdf` (vector, best for LaTeX), `png` (raster fallback).

### Figure Type Auto-Selection Decision Tree

| Data Pattern | Recommended Type | Size |
|-------------|-----------------|------|
| X=time/steps, Y=metric | Line plot | 0.48\textwidth |
| Methods x 1 metric | Bar chart | 0.48\textwidth |
| Methods x multiple metrics | Grouped bar / radar | 0.95\textwidth |
| Two continuous variables | Scatter plot | 0.48\textwidth |
| Matrix / grid values | Heatmap | 0.48\textwidth |
| Distribution comparison | Box/violin plot | 0.48\textwidth |
| Multi-dataset results | Multi-panel (subfigure) | 0.95\textwidth |
| Prior work comparison | LaTeX table | — |

### Figure Type Reference

| Type | When to Use | Typical Size |
|------|------------|--------------|
| Line plot | Training curves, scaling trends | 0.48\textwidth |
| Bar chart | Method comparison, ablation | 0.48\textwidth |
| Grouped bar | Multi-metric comparison | 0.95\textwidth |
| Scatter plot | Correlation analysis | 0.48\textwidth |
| Heatmap | Attention, confusion matrix | 0.48\textwidth |
| Box/violin | Distribution comparison | 0.48\textwidth |
| Architecture | System overview | 0.95\textwidth |
| Multi-panel | Combined results (subfigures) | 0.95\textwidth |
| Comparison table | Prior bounds vs. ours (theory) | full width |

### Generating Each Figure

For each figure in the plan, create a standalone Python script (one script per figure). Example patterns:

**Line plots** (training curves, scaling):
```python
# gen_fig2_training_curves.py
from paper_plot_style import *
import json
with open('figures/exp_results.json') as f:
    data = json.load(f)
fig, ax = plt.subplots(1, 1, figsize=(5, 3.5))
ax.plot(data['steps'], data['fac_loss'], label='Factorized', color=COLORS[0])
ax.plot(data['steps'], data['crf_loss'], label='CRF-LR', color=COLORS[1])
ax.set_xlabel('Training Steps')
ax.set_ylabel('Cross-Entropy Loss')
ax.legend(frameon=False)
save_fig(fig, 'fig2_training_curves')
```

**Bar charts** (comparison, ablation):
```python
fig, ax = plt.subplots(1, 1, figsize=(5, 3))
methods = ['Baseline', 'Method A', 'Method B', 'Ours']
values = [82.3, 85.1, 86.7, 89.2]
bars = ax.bar(methods, values, color=[COLORS[i] for i in range(len(methods))])
ax.set_ylabel('Accuracy (%)')
for bar, val in zip(bars, values):
    ax.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 0.3,
            f'{val:.1f}', ha='center', va='bottom', fontsize=9)
save_fig(fig, 'fig3_comparison')
```

**Comparison tables** (LaTeX, for theory papers):
```latex
\begin{table}[t]
\centering
\caption{Comparison of estimation error bounds. $n$: sample size, $D$: ambient dim, $d$: latent dim, $K$: subspaces, $n_k$: modes.}
\label{tab:bounds}
\begin{tabular}{lccc}
\toprule
Method & Rate & Depends on $D$? & Multi-modal? \\
\midrule
\citet{MinimaxOkoAS23} & $n^{-s'/D}$ & Yes (curse) & No \\
\citet{ScoreMatchingdistributionrecovery} & $n^{-2/d}$ & No & No \\
\textbf{Ours} & $\sqrt{\sum n_k d_k / n}$ & No & Yes \\
\bottomrule
\end{tabular}
\end{table}
```

**Architecture/pipeline diagrams** (MANUAL — outside this skill's scope):
- These require manual creation using draw.io, Figma, Keynote, or TikZ
- If the figure already exists in `figures/`, preserve it and generate only the LaTeX `\includegraphics` snippet
- Flag as `[MANUAL]` in the figure plan and `latex_includes.tex`

### latex_includes.tex Format

For each figure, output the LaTeX code to include it. Save all snippets to `figures/latex_includes.tex`:

```latex
% === Fig 2: Training Curves ===
\begin{figure}[t]
    \centering
    \includegraphics[width=0.48\textwidth]{figures/fig2_training_curves.pdf}
    \caption{Training curves comparing factorized and CRF-LR denoising.}
    \label{fig:training_curves}
\end{figure}
```

### Figure Quality Review via Codex MCP

Send figure descriptions and captions to the reviewer model for review:

```
mcp__codex__codex:
  model: [reviewer_model from project.json]
  config: {"model_reasoning_effort": "xhigh"}
  prompt: |
    Review these figure/table plans for a [VENUE] submission.

    For each figure:
    1. Is the caption informative and self-contained?
    2. Does the figure type match the data being shown?
    3. Is the comparison fair and clear?
    4. Any missing baselines or ablations?
    5. Would a different visualization be more effective?

    [list all figures with captions and descriptions]
```

Apply feedback before finalizing figures.

### Figure Quality Checklist

Before finishing, verify each figure:

- [ ] Font size readable at printed paper size (not too small)
- [ ] Colors distinguishable in grayscale (print-friendly)
- [ ] **No title inside figures** — titles go only in LaTeX `\caption{}`
- [ ] Legend does not overlap data
- [ ] Axis labels have units where applicable
- [ ] Axis labels are publication-quality (not variable names like `emp_rate`)
- [ ] Figure width fits single column (0.48\textwidth) or full width (0.95\textwidth)
- [ ] PDF output is vector (not rasterized text)
- [ ] No matplotlib default title (remove `plt.title` for publications)
- [ ] Serif font matches paper body text (Times / Computer Modern)
- [ ] Colorblind-accessible (if using colorblind palette)

### paper_figures Rules

- Every figure must be reproducible — save the generation script alongside the output
- Do NOT hardcode data — always read from JSON/CSV files
- Use vector format (PDF) for all plots — PNG only as fallback
- No decorative elements — no background colors, no 3D effects, no chart junk
- Consistent style across all figures — same fonts, colors, line widths
- One script per figure — easy to re-run individual figures when data changes
- No titles inside figures — captions are in LaTeX only
- Comparison tables count as figures — generate them as standalone .tex files

---

