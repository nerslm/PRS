# PRS — 个人科研系统

基于 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 的状态驱动科研框架。从 idea 到投稿，一个 agent 掌控全局。

[English](README.md) | 中文

> PRS 是 [ARIS](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep) 的精简重构版。相同的跨模型审稿架构，但围绕一个有状态的主 agent（`/prs`）重新组织，取代了 31 个独立 skill。

## 这是什么

PRS 为 Claude Code 提供长期科研所需的结构：状态追踪（`progress.json`）、会话恢复（`session.json`）、领域知识模块、以及清晰的 4 阶段流水线。主 agent 做研究，PRS 负责组织。

**与 ARIS 的区别：** ARIS 有 31 个独立斜杠命令，PRS 只有 3 个入口（`/prs-init`、`/prs`、`/prs-paper`），一个 `next_action` 驱动所有进展。更少的学习成本，相同的能力。

## 快速开始

### 前置条件

1. **[Claude Code](https://docs.anthropic.com/en/docs/claude-code)** 已安装
2. **[Codex CLI](https://github.com/openai/codex)** — 作为 MCP 服务器提供跨模型审稿（GPT-5.4 审阅 Claude 的输出）
3. **Node.js** ≥ 18（Codex CLI 依赖）
4. **Python 3**（用于 `tools/arxiv_fetch.py` 和实验脚本）

### 第 1 步：安装 PRS

```bash
git clone https://github.com/nerslm/PRS.git
cd PRS

# 安装 skills 到 Claude Code
cp -r skills/* ~/.claude/skills/

# 安装辅助 skills（可选的独立工具）
cp -r auxiliary/* ~/.claude/skills/
```

### 第 2 步：配置 Codex MCP（审稿必需）

所有审稿、精炼、改进循环都依赖 Codex MCP —— Claude Code 写，GPT-5.4 审。

```bash
# 安装 Codex CLI
npm install -g @openai/codex

# 初始配置 —— 提示选模型时选 gpt-5.4
codex setup

# 注册为 Claude Code 的 MCP 服务器
claude mcp add codex -s user -- codex mcp-server
```

> **重要：** 审稿模型在 `~/.codex/config.toml` 里设置，不在 PRS 文件中。确认其中写的是 `model = "gpt-5.4"`（推荐）。其他可选：`gpt-5.3-codex`、`gpt-5.2-codex`、`o3`。

### 第 3 步：验证

```bash
claude
> /prs-init "你的研究方向"
```

如果成功创建 `project.json` 和 `progress.json`，就可以用了。

### 第 4 步（可选）：LaTeX 环境

仅 **paper** 阶段（写论文 + 编译 PDF）需要：

```bash
# Ubuntu/Debian
sudo apt install texlive-full latexmk poppler-utils

# macOS
brew install --cask mactex
brew install poppler

# 验证
latexmk --version && pdftotext -v
```

> 如果只用 idea 发现 + 实验 + 审稿，可以跳过。

## 使用方法

```bash
# 初始化新研究项目
/prs-init "你的研究方向"                          # 交互式：会问会议、自主模式
/prs-init "你的方向" — venue: neurips               # 跳过提示

# 继续研究（读取状态，从上次中断的地方继续）
/prs

# 带意图（引导下一步行动）
/prs "精炼我的方法"
/prs "跑实验"
/prs "开始写论文"

# 直接跳到写论文阶段
/prs-paper
```

## 工作原理

```
/prs-init → project.json + progress.json
                │
/prs        → 读状态 → 读 next_action → 执行 → 更新状态 → 重复
                │
           4 个阶段，由 next_action 驱动：

           idea → experiment → review → paper → done

           每个阶段有对应的上下文模块提供领域约束。
           主 agent 做思考。子 agent 做杂活。
```

## 4 个阶段

| 阶段 | 做什么 | 关键产物 |
|---|---|---|
| **idea** | 文献调研 → 头脑风暴 (Codex MCP) → 查新 → 精炼方案 | `IDEA_REPORT.md`, `refine-logs/FINAL_PROPOSAL.md` |
| **experiment** | 设计实验 → 实现代码 → 部署到 GPU → 收集结果 | `EXPERIMENT_PLAN.md`, `code/`, `results/` |
| **review** | 自动审稿循环 (Codex MCP) → 实施修复 → 重审（最多 4 轮） | `AUTO_REVIEW.md` |
| **paper** | 规划大纲 → 生成图表 → 写 LaTeX → 编译 PDF → 会议感知改进 | `PAPER_PLAN.md`, `paper/main.pdf` |

## 自主模式

`/prs-init` 时设置。一个参数，三种选择：

| 模式 | 行为 |
|---|---|
| `full` | 全程不停。从 idea 到论文，完全无人值守。 |
| `gates` | 阶段内自动推进。在 3 个决策点暂停：idea 选择、实验计划、论文大纲。 |
| `manual` | 每个主要动作后暂停，等你审阅。 |

---

## 详细配置

### Codex MCP（必需）

核心依赖。PRS 使用 **跨模型协作**：Claude Code 执行研究，GPT-5.4（通过 Codex MCP）担任审稿人。避免单模型自我博弈的盲区。

```bash
npm install -g @openai/codex
codex setup                              # 选 gpt-5.4
claude mcp add codex -s user -- codex mcp-server
```

MCP 服务器暴露两个工具：
- `mcp__codex__codex` — 发送审稿/头脑风暴 prompt 给 GPT-5.4
- `mcp__codex__codex-reply` — 获取响应

### 替代模型组合

没有 OpenAI API？PRS 继承了 ARIS 的灵活模型架构，可以替换审稿模型：

| 方案 | 执行者 | 审稿者 | 指南 |
|---|---|---|---|
| **默认** | Claude Code | GPT-5.4 (Codex MCP) | 本 README |
| **llm-chat** | Claude Code | 任意 OpenAI 兼容 API | 见 [模型混搭指南](docs/LLM_API_MIX_MATCH_GUIDE.md) + [`mcp-servers/llm-chat/`](mcp-servers/llm-chat/) |
| **MiniMax** | Claude Code | MiniMax-M2.5 | 见 [MiniMax 指南](docs/MINIMAX_MCP_GUIDE.md) + [`mcp-servers/minimax-chat/`](mcp-servers/minimax-chat/) |
| **Codex→Claude** | Codex CLI | Claude Code | 见 [Codex+Claude 审稿指南](docs/CODEX_CLAUDE_REVIEW_GUIDE_CN.md) + [`mcp-servers/claude-review/`](mcp-servers/claude-review/) |

PRS 在 `mcp-servers/` 内置了 4 个 MCP 服务器，无需依赖外部仓库。

### LaTeX（论文阶段用）

```bash
# Ubuntu/Debian
sudo apt install texlive-full latexmk poppler-utils

# macOS
brew install --cask mactex && brew install poppler
```

PRS 包含 **24 个 LaTeX 会议模板**（`templates/` 目录）：NeurIPS、ICLR、ICML、CVPR、ACL、AAAI、ACM MM、Nature Communications、PRL。

### Gemini API（AI 作图，可选）

`/paper-illustration` 辅助 skill 使用 Google Gemini 生成论文级架构图。

```bash
# 从 https://aistudio.google.com/apikey 获取
export GEMINI_API_KEY="your-key-here"
```

> **免费替代：** 用 `/mermaid-diagram` 代替，无需 API key。

### GPU 服务器（自动实验，可选）

在项目的 `CLAUDE.md` 中添加服务器信息：

```markdown
## Remote Server

- SSH: `ssh my-gpu-server`（密钥认证，免密码）
- GPU: 4x A100
- Conda env: `research`
- Activate: `eval "$(/opt/conda/bin/conda shell.bash hook)" && conda activate research`
- Code directory: `/home/user/experiments/`
```

**没有服务器？** 审稿和写作功能照常工作，只有实验部署会跳过。

### 飞书通知（可选）

实验完成或审稿打分时收手机通知。**默认关闭，未配置时零影响。**

```json
// ~/.claude/feishu.json
{
  "mode": "push",
  "webhook_url": "https://open.feishu.cn/open-apis/bot/v2/hook/YOUR_WEBHOOK_ID"
}
```

双向交互模式需部署 [feishu-claude-code](https://github.com/joewongjc/feishu-claude-code) 桥接，或使用 PRS 自带的 [`mcp-servers/feishu-bridge/`](mcp-servers/feishu-bridge/)。

### Zotero 集成（可选）

```bash
uv tool install zotero-mcp-server
claude mcp add zotero -s user -- zotero-mcp -e ZOTERO_LOCAL=true
```

### Obsidian 集成（可选）

```bash
claude mcp add obsidian-vault -s user -- npx @bitbonsai/mcpvault@latest /path/to/your/vault
```

---

## 无人值守运行配置

添加到 `.claude/settings.local.json`，免除权限提示：

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

## 项目配置

所有配置在 `project.json` 中（`/prs-init` 创建）：

| 参数 | 默认值 | 说明 |
|---|---|---|
| `autonomy` | `gates` | `full` / `gates` / `manual` — 控制何时暂停 |
| `reviewer_model` | `gpt-5.4` | Codex MCP 使用的审稿模型 |
| `arxiv_download` | `false` | 文献调研时下载 arXiv PDF |
| `pilot_max_hours` | `2` | 每个 pilot 实验最大时长 |
| `max_review_rounds` | `4` | 审稿循环最大轮数 |
| `max_refine_rounds` | `5` | idea 精炼最大轮数 |
| `max_paper_improve_rounds` | `4` | 论文改进最大轮数 |
| `score_threshold_refine` | `9` | 精炼分数 >= 此值时停止 |
| `score_threshold_review` | `8.5` | 审稿分数 >= 此值时停止 |

---

## 依赖总结

| 依赖 | 必需？ | 用途 | 安装 |
|---|---|---|---|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | **是** | 执行主体 | `npm install -g @anthropic-ai/claude-code` |
| [Codex CLI](https://github.com/openai/codex) | **是** | 跨模型审稿 (GPT-5.4) | `npm install -g @openai/codex` + `codex setup` |
| Node.js >= 18 | **是** | Codex CLI 依赖 | 系统包管理器 |
| Python 3 | **是** | arxiv_fetch.py, 实验脚本 | 系统 / conda |
| LaTeX (texlive) | 仅论文阶段 | 编译 PDF | `apt install texlive-full latexmk poppler-utils` |
| Gemini API Key | 可选 | 仅 `/paper-illustration` | [aistudio.google.com/apikey](https://aistudio.google.com/apikey) |
| SSH + screen/tmux | 可选 | 远程实验部署 | 系统包 |
| W&B | 可选 | 实验追踪 | `pip install wandb` |
| Zotero MCP | 可选 | 搜索论文库 | [zotero-mcp](https://github.com/54yyyu/zotero-mcp) |
| Obsidian MCP | 可选 | 搜索研究笔记 | [mcpvault](https://github.com/bitbonsai/mcpvault) |
| 飞书 webhook | 可选 | 手机通知 | 飞书群机器人 |

## 与 ARIS 的关系

PRS 基于 [ARIS](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep) 构建。主要区别：

- **单一有状态 agent** — `/prs` + `next_action`，而非 31 个独立命令
- **状态驱动** — `project.json` + `progress.json` + `session.json` 提供完整会话恢复
- **上下文模块** — 领域知识抽取为独立 `.md` 文件
- **相同审稿架构** — 仍使用 Codex MCP 进行 Claude x GPT-5.4 跨模型协作
- **完整 ARIS 存档** — 原始 skills 保留在 `archive/` 作为参考

## License

MIT
