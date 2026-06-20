# Parallel Goal Workflows

**[English README](README.md)**

`parallel-goal-workflows` 是一个需要手动调用的高开销多 Agent 工作流 Skill。它让主会话保留
Lead 角色，把任务级策略交给一个 Orchestrator，并允许更深层 Agent 处理局部目标，但不再创建新的
全局策略层。

## 安装

```bash
npx skills add patrick-fu/parallel-goal-workflows
```

后续更新：

```bash
npx skills update
```

## 快速使用

显式点名这个 Skill：

```text
请使用 $parallel-goal-workflows 处理这个任务。保持一个全局策略 owner：
Lead -> Orchestrator。Orchestrator 可以为了局部目标使用嵌套 Worker、Reviewer、
Verifier、Repair、Synthesis 或 Helper，但任何子 Agent 都不应该为同一个用户目标再创建
另一个 Orchestrator。
```

## 核心边界

```text
Lead -> Orchestrator -> Worker / Review / Acceptance / Repair / Synthesis / Helper...
```

- Lead 负责用户会话、等待、转发澄清和汇报 Orchestrator 的最终结果。
- Orchestrator 负责任务级策略、review、repair、acceptance 和最终判断。
- 下游 Agent 可以创建更窄的 helper，但不重新运行 lead/orchestrator 工作流。

这个 Skill 要解决的问题不是“层级太多”。真正的问题是 Ultra-Strategy：被委派的 Agent 把整个任务
当成一个新的全局策略问题，于是又派生一个 Orchestrator。修复点是 role packet，不是浅层 depth cap。

## 运行时兼容

- **Claude Code:** `SKILL.md` 里的 `disable-model-invocation: true` 让 Skill 只能手动调用。
  Claude Code v2.1.172 及更新版本支持最多 5 层嵌套 subagent。
- **OpenAI Codex:** `agents/openai.yaml` 里设置
  `policy.allow_implicit_invocation: false`，保留显式 `$parallel-goal-workflows` 调用，同时关闭隐式触发。

## Codex Depth

实用的 Codex 配置：

```toml
[agents]
max_threads = 50
max_depth = 5

[features]
multi_agent = true
goals = true
```

把 depth 当成局部 helper 的容量：

```text
Lead -> Orchestrator -> Worker -> Helper -> Verifier -> Repair
```

不要把 depth 用来创建 `Orchestrator -> Orchestrator` 递归。委派 packet 应该始终写明全局策略 owner
和局部 scope。

更多细节见
[`references/codex-nested-subagents.md`](references/codex-nested-subagents.md)。

## 更多 Skills

更多可复用的 Agent Skills 可以看
[Awesome Skills](https://github.com/patrick-fu/awesome-skills/blob/main/README.zh-CN.md)。
