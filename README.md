# Parallel Goal Workflows

**[中文说明](README.zh-CN.md)**

`parallel-goal-workflows` is a guidance skill for delegated, multi-agent work.
It helps a lead agent hand workflow ownership to an orchestrator, stay out of
task-level execution, and receive an acceptance-ready report instead of every
intermediate detail.

## Install

```bash
npx skills add patrick-fu/parallel-goal-workflows
```

Confirm the install:

```bash
test -f ~/.agents/skills/parallel-goal-workflows/SKILL.md
```

Update later:

```bash
npx skills update
```

## Quick Use

This skill is intentionally high-overhead. Name it explicitly when you want the
workflow boundary:

```text
Use parallel-goal-workflows for this task. The Lead Agent should start an
Orchestrator, wait instead of doing task-level work, and report back only after
the Orchestrator returns an acceptance-ready result.
```

## Minimal Orchestrator Goal Packet

```text
/goal Orchestrate this delegated workflow to completion and return an
acceptance-ready report.

Context:
[User goal, constraints, relevant project rules, and quality bar.]

Deliverable:
[Worker results, independent review result, acceptance or verification result,
repair loop if any, final judgment, remaining risks, and a concise report the
Lead can relay.]

Pause if:
[Credentials, destructive actions, external approval, user judgment, or a
repeated blocker is required.]
```

## What It Does

This skill turns a broad delegated task into an orchestrator-owned workflow:

- the Lead Agent owns the user conversation and final handoff;
- the Orchestrator owns task decomposition, scheduling, review, acceptance, and
  repair routing;
- Worker, Review, Acceptance, Repair, and Synthesis agents each receive focused
  goals;
- every spawned agent uses either native Goal mode, when the host exposes it,
  or a goal-shaped delegation packet with a clear completion condition;
- the Lead waits with callback-style patience instead of polling or taking work
  back;
- worker agents may delegate further when the host environment supports nested
  subagents.

The goal is context, not control. The skill does not prescribe a rigid script.
It gives agents enough ownership boundaries to coordinate well while leaving
room for the workflow owner to adapt.

## When To Use It

Use this skill when a task benefits from deliberate, high-overhead delegation
and you do not want the main conversation to become the coordination workspace.
It is not the default pattern for ordinary coding, research, review, simple
parallel exploration, or generic goal decomposition.

Good fits include:

- parallel code review, codebase audits, or cross-checked research;
- multi-step implementation plans that need independent workers and review;
- long-running command or subagent work where the lead might otherwise poll,
  interrupt, or restart too aggressively;
- review and repair loops where the main context should only receive the final
  decision and evidence;
- nested subagent workflows where a worker may need its own workers.

## Goal Discipline

This skill is goal-first. Every participating agent should start from a goal,
not a vague chore:

- Lead goal: preserve the conversation boundary and wait for the orchestrator's
  acceptance-ready report.
- Orchestrator goal: own decomposition, scheduling, review, acceptance, repair,
  and final reporting.
- Downstream goals: give each Worker, Review, Acceptance, Repair, and Synthesis
  agent one concrete outcome, expected evidence, boundaries, and pause
  conditions.

When the host exposes native Goal mode for the relevant session or thread, use
it. When a runtime does not expose per-subagent Goal mode, put the same goal
packet in the delegation message so the subagent still works from an explicit
completion contract.

## Workflow Shapes

The orchestrator chooses the shape that fits the task. These diagrams are
coarse workflow patterns, not scripts.

### Orchestrated Review

```mermaid
flowchart LR
  User["User"] --> Lead["Lead Agent<br/>conversation boundary"]
  Lead --> Orchestrator["Orchestrator<br/>workflow owner"]
  Orchestrator --> Worker["Worker / research goal"]
  Worker --> Review["Independent review"]
  Review --> Decision{"Good enough?"}
  Decision -- "No" --> Repair["Repair Agent"]
  Repair --> Review
  Decision -- "Yes" --> Acceptance["Acceptance / Verification"]
  Acceptance --> Report["Acceptance-ready report"]
  Report --> Lead
  Lead --> User
```

### Parallel Synthesis

```mermaid
flowchart LR
  O["Orchestrator"] --> A["Worker A goal"]
  O --> B["Worker B goal"]
  O --> C["Worker C goal"]
  A --> S["Synthesis goal"]
  B --> S
  C --> S
  S --> Decision{"Conflict or gap?"}
  Decision -- "Yes" --> Followup["Targeted follow-up goal"]
  Followup --> S
  Decision -- "No" --> Acceptance["Acceptance / report"]
```

### Rolling waves

```mermaid
flowchart LR
  O["Orchestrator"] --> Explore["Wave 1:<br/>broad exploration"]
  Explore --> Decision{"Enough signal?"}
  Decision -- "No" --> Narrow["Next wave:<br/>narrower goals"]
  Narrow --> Decision
  Decision -- "Yes" --> Focus["Focused worker goals"]
  Focus --> Review["Review / acceptance"]
  Review --> Report["Final report"]
```

### Nested delegation

```mermaid
flowchart LR
  O["Orchestrator"] --> W["Worker goal"]
  W --> Decision{"Needs deeper help?"}
  Decision -- "Yes" --> A["Sub-worker A goal"]
  Decision -- "Yes" --> B["Sub-worker B goal"]
  A --> S["Worker synthesis"]
  B --> S
  Decision -- "No" --> Direct["Worker result"]
  S --> Review["Review / acceptance"]
  Direct --> Review
  Review --> Report["Final report"]
```

## Why It Helps

### Keeps the Lead Agent from retaking delegated work

Main agents often struggle to remain in observation mode after delegation. After
spawning a subagent or launching a long-running command, they may start doing
the same work themselves, poll too frequently, stop slow commands, or close and
restart subagents at the first sign of friction.

This skill gives the Lead Agent its own boundary goal: start the orchestrator,
wait with callback-style patience, relay user updates when needed, and report
back without becoming the hidden worker.

### Uses the Orchestrator as a context buffer

In a normal subagent workflow, the Main Agent often still absorbs review,
acceptance, repair decisions, and noisy intermediate findings. That burns the
main context window.

Here, a second-level Orchestrator absorbs that work. The Lead gets the final
report, supporting evidence, and remaining risks, while the messy coordination
stays inside the delegated workflow.

### Preserves flexible orchestration

Dynamic workflow systems often move the plan into code so the runtime can run
large repeatable fan-outs. This skill is deliberately lighter: it keeps the plan
in agent goals and ownership boundaries. Use it when you want a reusable
coordination preference rather than a generated workflow script.

## Requirements

For the full workflow, the host environment should support native Goal mode
where available and multi-level subagents when nested delegation is needed.

- **Codex:** check the [Codex subagents docs](https://developers.openai.com/codex/subagents)
  and [config basics](https://developers.openai.com/codex/config-basic). Codex
  documents Goal mode and says to enable `features.goals` if `/goal` is not
  visible. It also documents `agents.max_depth` as the spawned-agent nesting
  depth and notes that the default `max_depth = 1` prevents deeper nesting. A
  practical starting point is:

  ```toml
  [agents]
  max_threads = 50
  max_depth = 5

  [features]
  multi_agent = true
  goals = true
  ```

- **Claude Code:** use version `2.1.172` or newer for nested subagents. The official
  [Claude Code changelog](https://code.claude.com/docs/en/changelog#2-1-172)
  says v2.1.172 added sub-agents spawning their own sub-agents, up to 5 levels
  deep. Claude Code's `/goal` requires `2.1.139` or newer, but the public
  subagent configuration docs do not document a per-subagent `goal` field. Use
  native `/goal` for Claude sessions that expose it; for named subagents, pass
  the goal packet in the delegation prompt unless your runtime exposes a native
  per-subagent goal control. Check your local version with:

  ```bash
  claude --version
  ```

## Repository Layout

This standalone repository is a single-skill package with the skill body and
references flattened at the repository root:

```text
README.md
README.zh-CN.md
SKILL.md
references/
```

## More Skills

For more reusable agent skills, see
[Awesome Skills](https://github.com/patrick-fu/awesome-skills). It includes
skills for brainstorming, coding-agent delegation, code review, commit
messages, goal contracts, learning coaching, home config sync, and
log-driven debugging.

## Related Reading

- [Codex subagents](https://developers.openai.com/codex/subagents)
- [Codex goals](https://developers.openai.com/codex/use-cases/follow-goals)
- [Codex config basics](https://developers.openai.com/codex/config-basic)
- [Claude Code goals](https://code.claude.com/docs/en/goal)
- [Claude Code subagents](https://code.claude.com/docs/en/sub-agents)
- [Claude Code dynamic workflows](https://code.claude.com/docs/en/workflows)
- [Claude Code changelog](https://code.claude.com/docs/en/changelog#2-1-172)
- [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
