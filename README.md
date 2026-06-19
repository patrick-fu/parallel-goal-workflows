# Parallel Goal Workflows

**[中文说明](README.zh-CN.md)**

`parallel-goal-workflows` is an agent skill for goal-driven multi-agent work. It
guides a lead agent to start an orchestrator, hold a conversation-level boundary
goal, wait with callback-style patience, and report back while the orchestrator
coordinates workers, review, acceptance, and repair.

## Install

```bash
npx skills add patrick-fu/parallel-goal-workflows
```

To update later:

```bash
npx skills update
```

## What It Helps With

- delegated workflows where the lead should not become a hidden worker
- fan-out / fan-in agent work with independent review
- orchestrator-owned acceptance and repair loops
- nested subagent workflows when the host environment supports them
- Codex configuration guidance for nested subagents

## Workflow Shape

```mermaid
flowchart LR
  User["User"] --> Lead["Lead Agent: boundary goal"]
  Lead --> Orchestrator["Orchestrator: workflow owner"]
  Orchestrator --> Workers["Worker / Research / Evidence"]
  Orchestrator --> Review["Independent Review"]
  Review --> Decision{"Review passes?"}
  Decision -- "No" --> Repair["Repair Agent"]
  Repair --> Review
  Decision -- "Yes" --> Acceptance["Acceptance / Verification"]
  Acceptance --> Report["Acceptance-ready report"]
  Report --> Lead
  Lead --> User
```

## Review And Repair Loop

```mermaid
sequenceDiagram
  participant User
  participant Lead
  participant Orchestrator
  participant Worker
  participant Review
  participant Acceptance
  User->>Lead: Ask for delegated work
  Lead->>Orchestrator: Delegate goal and wait
  Orchestrator->>Worker: Produce result and evidence
  Worker-->>Orchestrator: Result
  Orchestrator->>Review: Independent review
  alt Needs repair
    Review-->>Orchestrator: Findings
    Orchestrator->>Worker: Narrow repair goal
    Worker-->>Orchestrator: Repaired result
    Orchestrator->>Review: Re-check
  else Passes review
    Review-->>Orchestrator: Pass
  end
  Orchestrator->>Acceptance: Verify against user goal
  Acceptance-->>Orchestrator: Acceptance signal
  Orchestrator-->>Lead: Acceptance-ready report
  Lead-->>User: Plain handoff
```

## Included Skill

- `parallel-goal-workflows`

## Notes

The skill is intentionally guidance-first. It provides context and ownership
patterns rather than a rigid script for how every agent must behave.
