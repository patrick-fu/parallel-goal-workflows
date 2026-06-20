# Parallel Goal Workflows

**[中文说明](README.zh-CN.md)**

`parallel-goal-workflows` is a manual guidance skill for high-overhead
multi-agent work. It keeps the main conversation as the Lead, hands task
strategy to one Orchestrator, and lets deeper agents handle local goals without
creating another global strategy layer.

## Install

```bash
npx skills add patrick-fu/parallel-goal-workflows
```

Update later:

```bash
npx skills update
```

## Quick Use

Name the skill explicitly:

```text
Use $parallel-goal-workflows for this task. Keep one global strategy owner:
Lead -> Orchestrator. The Orchestrator may use nested workers, reviewers,
verifiers, repair agents, synthesis agents, or helpers for local goals, but no
child should create another Orchestrator for the same user goal.
```

## Core Boundary

```text
Lead -> Orchestrator -> Worker / Review / Acceptance / Repair / Synthesis / Helper...
```

- The Lead owns the user conversation, waits, relays clarifications, and reports
  the Orchestrator's final result.
- The Orchestrator owns task-level strategy, review, repair, acceptance, and
  final judgment.
- Downstream agents may create narrower helpers, but they do not re-run the
  lead/orchestrator workflow.

The problem this skill prevents is not "too many levels." The problem is
Ultra-Strategy: a delegated agent treats the whole task as a fresh strategy
problem and spawns another global Orchestrator. The fix is a role packet, not a
shallow depth cap.

## Runtime Compatibility

- **Claude Code:** `disable-model-invocation: true` in `SKILL.md` makes the
  skill manual. Claude Code v2.1.172 and newer support nested subagents up to 5
  levels deep.
- **OpenAI Codex:** `agents/openai.yaml` sets
  `policy.allow_implicit_invocation: false`, so explicit `$parallel-goal-workflows`
  still works while implicit invocation is disabled.

## Codex Depth

A practical Codex configuration is:

```toml
[agents]
max_threads = 50
max_depth = 5

[features]
multi_agent = true
goals = true
```

Use depth as capacity for local helpers:

```text
Lead -> Orchestrator -> Worker -> Helper -> Verifier -> Repair
```

Do not use depth to create `Orchestrator -> Orchestrator` recursion. The
delegation packet should always state the global strategy owner and local scope.

For more detail, see
[`references/codex-nested-subagents.md`](references/codex-nested-subagents.md).

## More Skills

For more reusable agent skills, see
[Awesome Skills](https://github.com/patrick-fu/awesome-skills).
