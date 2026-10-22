# thin-orchestrator

[English](README.md) | 简体中文

**贵的模型动脑，便宜的模型动手。**

一个 [Claude Code](https://claude.com/claude-code) 技能 + 子代理，把较大的编码任务拆给两档模型：

- **主会话**（Opus 或 Fable）：负责设计、写任务说明（brief）、审查结果，决定是否验收。
- **`flash-builder` 子代理**（Sonnet）：负责探索仓库、写代码、跑测试、调试，最后交回一份完成报告。

主会话的 token 只花在最值钱的地方：架构、验收标准、实质风险和最终裁决。
冗长的"改代码 → 跑测试 → 修错误"循环交给便宜的模型去跑。

## 工作流程

```
 你 ──▶ 主会话（Opus/Fable）
           │ 1. 判断规模——值不值得委派？
           │ 2. 设计 + 写一份精确的 brief
           │ 3. 派发「一个」flash-builder，然后等待（不轮询）
           ▼
         flash-builder（Sonnet）
           │ 探索 → 实现 → 测试 → 修复 → … → 报告
           ▼
         主会话
           │ 4. 一次批量审查：规格符合度 + 质量/安全
           │ 5. 验收；或把所有问题一次性发回「同一个」worker（默认最多一轮返工）
           ▼
         最终回复：做了什么、实际通过了什么、还剩什么
```

一个正常阶段就是：**一次规划 → 一次派发 → 一次等待 → 一次审查 → 一次回复。**

## 什么时候委派，什么时候不委派

委派有固定开销：worker 的全新上下文（约 5 万 token）、写 brief、审查整个 patch。
所以只有**两条都满足**才委派：

1. 改动涉及 **3 个以上文件**，或要新建一个完整的模块/页面；
2. 实现过程需要**反复**"读代码 → 跑测试 → 改"，而不是一次写完。

否则主会话直接做。一次本地实测：一个约 500 行、2 个文件、一次写完的任务，委派的总成本是直接做的 **1.9 倍**，
token 用量是 **2.9 倍**。只有实现循环本身很长的任务，委派才省钱。

## 核心规则

- **一个工作区只有一个 writer。** 只有明确标注为独立的任务，才允许第二个并行 writer，且必须放在 git worktree 里。
- **等待时放手。** 不轮询、不问进度、不做重叠修改。
- **返工发给同一个 worker**（用 `SendMessage`）。同一个任务包不要另开新 worker，那样要重新付上下文开销，
  还会丢掉它已经建立的理解。
- **做完不等于验收。** 只有主会话能验收。对照记录的 baseline 审查实际 diff，包括未跟踪的新文件。
- **验证先便宜后贵：** 读 worker 的证据 → 跑它给的命令 → 读关键代码。主会话不自己写参考实现或对拍脚本。
- **不找替身。** 没有 `flash-builder` 时，规划完就停下，不退回用 `general-purpose`（它会继承贵的模型）。
- **不自动做 Git 或生产操作。** 没有明确授权，不 commit、不合并、不 push、不部署、不跑迁移。
- **高风险例外。** 鉴权、支付、密钥、多租户、破坏性迁移、生产行为，主会话会加大审查力度，并说明理由。

## 安装

### 作为插件安装（推荐）

在 Claude Code 里执行：

```
/plugin marketplace add donnie0915/thin-orchestrator
/plugin install thin-orchestrator@thin-orchestrator
```

以插件方式安装时，子代理名字可能带命名空间（比如 `thin-orchestrator:flash-builder`），技能两种名字都能识别。

### 手动安装

```sh
git clone https://github.com/donnie0915/thin-orchestrator.git
mkdir -p ~/.claude/skills ~/.claude/agents
cp -r thin-orchestrator/skills/thin-orchestrator ~/.claude/skills/
cp thin-orchestrator/agents/flash-builder.md ~/.claude/agents/
```

重启 Claude Code（或开一个新会话），让技能和子代理生效。

## 使用

直接描述一个较大的任务即可。多文件功能、重构、迁移、"按这个计划实现"之类的请求会自动触发，例如：

> 规划并实现订单页的 CSV 导出功能，带测试。

也可以用 `/thin-orchestrator` 显式调用。不想委派时，直接说"不要委派"，所有工作就留在主会话里。

想换 worker 的模型，改 `agents/flash-builder.md` 里的 `model` 字段（`sonnet`、`haiku` 等）。

## 目录结构

```
.claude-plugin/
  plugin.json            插件清单
  marketplace.json       让本仓库本身就是一个插件市场
agents/
  flash-builder.md       实现 worker（model: sonnet）
skills/thin-orchestrator/
  SKILL.md               工作流定义
  references/            planning.md、execution.md、review.md
  templates/             spec、task-brief、task-report、checkpoint
```

工作产物默认放在项目的 `docs/agent-work/<feature>/` 下；项目已有约定则沿用。

## 致谢

灵感来自 [Astra Flash Orchestrator / Codex on Crack](https://github.com/ethanplusai/codex-on-crack)（MIT）
的"规划者/实现者"分工，这里针对 Claude Code 原生子代理做了改造，并按成本做了调优。

## 许可证

[MIT](LICENSE)
