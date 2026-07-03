# Task 2 — Tester agent

> Build in its own conversation. Read `docs/architecture.md` first (full context).
> **Depends on Task 1** — reuses the stack skills and their `references/testing.md`.

## Goal

Create the `tester` agent: a stack-aware agent that writes and runs unit tests for
whatever stack a change touches, then returns a pass/fail report. Same router
pattern as `developer` — it does **not** hardcode frameworks.

## Prerequisites

- Task 1 done: `react-dev`, `dotnet-dev` skills exist, each with `references/testing.md`.
- Read `docs/architecture.md` §7 (testing) and §8 (contracts).
- Use the `create-agent` skill.

## Deliverables

| File | What |
|---|---|
| `plugins/workflows/agents/tester.agent.md` | Stack-aware test writer + runner |

## Tester agent spec

Frontmatter:
```yaml
---
name: tester
description: "Use this agent to write and run unit tests for a code change. Detects the stack (React → Jest/RTL, .NET → xUnit/Moq), loads the matching stack skill for its testing conventions, writes tests, runs them, and returns a pass/fail report with coverage. Triggers after an implementation lands or when an orchestrator hands off a change to be tested."
model: inherit
color: yellow
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob", "Skill"]
---
```

Body must define:
- **Routing** — same as developer: detect stack, load matching stack skill(s). The
  skill's `references/testing.md` supplies the framework (Jest/RTL vs xUnit/Moq).
  Load the *relevant set* — full-stack change → write both suites.
- **Input contract** — the developer's output (files changed, stack) is the tester's
  input. Accepts an explicit `stack` hint or detects it.
- **Process** — write tests in the right framework + file convention
  (`*.test.tsx` / `*Tests.cs`), then run them (`npm test` / `dotnet test`).
- **Output contract / artifact** — test report: pass/fail, counts, coverage,
  failing-test details. Written to `.pipeline-artifacts/<run-id>/` when a
  `run-id` is provided in the input.
- **Verify ownership** — tester owns *new tests for the change*; the developer
  already verified build + pre-existing tests — don't re-verify the build.
- **Golden rule** — test the spec/behavior, not the implementation's bugs; a fresh
  isolated context is the point.

## Acceptance checklist

- [ ] `tester.agent.md` created with `Skill` in tools.
- [ ] Routing section loads stack skill(s); framework comes from `references/testing.md`, not hardcoded.
- [ ] Handles full-stack (loads both, writes both suites).
- [ ] Input contract = developer's output; output contract = test report artifact.
- [ ] Does not duplicate framework knowledge that lives in the stack skills.
- [ ] Run `marketplace-sync` after.

## Out of scope

- Editing the stack skills' testing conventions (owned by Task 1).
- Orchestration / retry loop (feature-pipeline, later task).
