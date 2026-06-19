# Parallel Goal Workflows

**[English README](README.md)**

`parallel-goal-workflows` 是一个面向多 Agent 委托工作的指导型 Skill。它帮助
Lead Agent 把工作流所有权交给 Orchestrator，自己只等待而不抢回任务，并最终接收一份
可验收报告，而不是吞下所有中间细节。

## 安装

```bash
npx skills add patrick-fu/parallel-goal-workflows
```

后续更新：

```bash
npx skills update
```

## 快速使用

这个 Skill 是有意设计的高开销工作流。需要 Lead / Orchestrator 边界时，建议显式点名：

```text
请使用 parallel-goal-workflows 处理这个任务。Lead Agent 应该启动 Orchestrator，
自己只等待和汇报，不做任务级执行；等 Orchestrator 返回可验收结果后再向我汇报。
```

## 它能做什么

这个 Skill 会把一个宽泛的委托任务变成 Orchestrator 负责的工作流：

- Lead Agent 只负责用户会话和最终汇报；
- Orchestrator 负责拆解任务、调度、review、验收、repair 路由和任务级判断；
- Worker、Review、Acceptance、Repair、Synthesis 等 Agent 分别拿到聚焦的 goal；
- 每一个被派生出来的 Agent 都尽量使用原生 Goal mode；没有 per-subagent goal 能力时，
  使用 goal-shaped delegation packet；
- Lead 以接近 callback 的节奏等待，而不是频繁轮询、打断或重启被委派的工作；
- 在宿主环境支持嵌套 subagent 时，下游 Agent 可以按需继续向下委派。

核心思路是 context, not control。这个 Skill 给 Agent 清晰的职责边界和完成信号，但不把
工作流写死成脚本。

## 什么时候使用

当任务值得引入一个明确的编排层，并且你不希望主会话变成协调工作区时，可以使用这个 Skill。

典型场景包括：

- 并行 code review、代码库审计、交叉验证式 research；
- 需要独立 worker 和 review 的多步骤实现计划；
- 长时间命令或 Sub Agent 工作，Lead 容易频繁轮询、打断或重启；
- review 和 repair loop 很重要，但主上下文只应该接收最终判断和证据；
- worker 可能还需要继续派生下级 worker 的嵌套 subagent workflow。

它不是普通编码、调研、review、简单并行探索或泛泛 goal decomposition 的默认模式。

## Goal-First 协调

每个参与工作的 Agent 都应该从 goal 开始，而不是接收一个模糊的差事：

- Lead goal：守住会话边界，等待 Orchestrator 返回可验收报告。
- Orchestrator goal：负责拆解、调度、review、验收、repair 和最终报告。
- Downstream goals：给每个 Worker、Review、Acceptance、Repair、Synthesis Agent 一个
  具体结果、证据要求、边界和暂停条件。

当宿主环境能给对应 session 或 thread 设置原生 Goal mode 时，就使用原生能力；当运行时
没有暴露 per-subagent Goal mode 时，就把同一份 goal packet 放进委派消息里。

## 工作流形态

Orchestrator 会根据任务选择合适的形态。下面这些是示例，不是脚本。

### 编排式评审

```mermaid
flowchart LR
  User["用户"] --> Lead["Lead Agent<br/>会话边界"]
  Lead --> Orchestrator["Orchestrator<br/>工作流 owner"]
  Orchestrator --> Worker["Worker goal"]
  Worker --> Review["独立 Review"]
  Review --> Decision{"足够好吗？"}
  Decision -- "否" --> Repair["Repair goal"]
  Repair --> Review
  Decision -- "是" --> Acceptance["Acceptance / Verification"]
  Acceptance --> Report["可验收报告"]
  Report --> Lead
  Lead --> User
```

### 并行综合

```mermaid
flowchart LR
  O["Orchestrator"] --> A["Worker A goal"]
  O --> B["Worker B goal"]
  O --> C["Worker C goal"]
  A --> S["Synthesis goal"]
  B --> S
  C --> S
  S --> Decision{"有冲突或缺口？"}
  Decision -- "是" --> Followup["定向补充 goal"]
  Followup --> S
  Decision -- "否" --> Acceptance["验收 / 报告"]
```

### 嵌套委派

```mermaid
flowchart LR
  O["Orchestrator"] --> W["Worker goal"]
  W --> Decision{"需要更深层帮助？"}
  Decision -- "是" --> A["Sub-worker A goal"]
  Decision -- "是" --> B["Sub-worker B goal"]
  A --> S["Worker synthesis"]
  B --> S
  Decision -- "否" --> Direct["Worker result"]
  S --> Review["Review / Acceptance"]
  Direct --> Review
  Review --> Report["最终报告"]
```

## 为什么有用

**抑制 Lead Agent 抢占工作。** Main/Lead Agent 完成委派后，经常很难保持观察状态。
这个 Skill 会给 Lead 一个自己的边界 goal：启动 Orchestrator、等待、转发补充信息，
最后汇报结果，但不变成隐藏 worker。

**隔离协调噪声。** Review 发现、repair loop、acceptance 检查和中间分歧都留在被编排的
工作流里。Lead 只接收最终报告、关键证据和剩余风险。

**保持灵活。** 这个 Skill 不是 workflow 脚本。它把协调约束放在 agent goal 和职责边界里，
让 Orchestrator 按任务选择合适的形态。

## 使用要求

要完整发挥工作流效果，宿主环境应该在可用时支持原生 Goal mode，并在需要嵌套委派时
支持多层级 Sub Agent。

- **Codex:** 参考 [Codex subagents 文档](https://developers.openai.com/codex/subagents)
  和 [config basics](https://developers.openai.com/codex/config-basic)。Codex 文档说明了 Goal mode；
  如果 `/goal` 不出现在 slash command 列表里，可以启用 `features.goals`。Codex 文档也说明
  `agents.max_depth` 控制 spawned agent 的嵌套深度，并且默认 `max_depth = 1` 会阻止更深层级的嵌套。
  一个实用的起始配置是：

  ```toml
  [agents]
  max_threads = 50
  max_depth = 5

  [features]
  multi_agent = true
  goals = true
  ```

- **Claude Code:** 如果需要嵌套 Sub Agent，请使用 `2.1.172` 或更新版本。官方
  [Claude Code changelog](https://code.claude.com/docs/en/changelog#2-1-172)
  明确写明 v2.1.172 开始支持 sub-agents 再 spawn 自己的 sub-agents，最多 5 层。
  Claude Code 的 `/goal` 需要 `2.1.139` 或更新版本，但公开的 Sub Agent 配置文档没有记录
  per-subagent `goal` 字段。对暴露原生 `/goal` 的 Claude session 使用原生能力；对 named
  subagent，在运行时没有暴露 per-subagent goal 控制时，把 goal packet 写进委派 prompt。

  ```bash
  claude --version
  ```

## 更多 Skills

更多可复用的 Agent Skills 可以看
[Awesome Skills](https://github.com/patrick-fu/awesome-skills/blob/main/README.zh-CN.md)。
