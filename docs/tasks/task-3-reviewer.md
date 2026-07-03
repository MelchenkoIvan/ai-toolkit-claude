# Task 3 — Reviewer agent

> Build in its own conversation. Read `docs/architecture.md` first (full context).
> **Soft dependency on Task 1** — the rubric loads the `coding-principles` skill.
> Buildable before Task 1 if that rubric dimension is stubbed, but principles must
> exist for a full review. No dependency on Task 2.

## Goal

Create the `reviewer` agent: a read-only agent with **its own review rubric** that
inspects a change and writes a review-report artifact. This is deliberately *not*
the built-in `/code-review` — you want a custom rubric and a persisted report.

## Prerequisites

- Read `docs/architecture.md` §4 (pipeline), §8 (contracts + artifacts).
- Use the `create-agent` skill.

## Deliverables

| File | What |
|---|---|
| `plugins/workflows/agents/reviewer.agent.md` | Read-only reviewer, own rubric, writes one report |

## Reviewer agent spec

Frontmatter:
```yaml
---
name: reviewer
description: "Use this agent to review a code change against a project rubric and produce a review report. Read-only over the code — it inspects the diff/files and writes exactly one report artifact, never edits the code. Triggers after an implementation + tests land, or when an orchestrator hands off a change for review."
model: inherit
color: blue
tools: ["Read", "Grep", "Glob", "Write", "Skill"]
---
```
> `Write` is granted **only** so it can emit the report artifact — never to edit code.
> `Skill` is granted so it can load `coding-principles` for its rubric.
> No `Bash`, no `Edit` on source. State this boundary in the body.

Body must define:
- **Rubric** — the custom review dimensions (e.g. correctness, adherence to
  `coding-principles`, readability, test adequacy, security smells). This is the
  reason for a custom agent over `/code-review` — make the rubric explicit.
  The agent loads the `coding-principles` skill via the `Skill` tool at the start
  of every review so the principles dimension matches what the developer was told.
- **Input contract** — change summary + test report from earlier stages, **plus
  the unified diff**. The reviewer has no `Bash` and cannot run `git diff` — the
  orchestrator (or the human dispatching it) computes the diff and passes it in.
  State in the body: if no diff is provided, ask for one rather than reviewing
  whole files and guessing what changed.
- **Output contract / artifact** — review report: verdict (✅ / ⚠️ / ❌), findings
  table (severity, file:line, issue, suggested fix), summary. Written to
  `.pipeline-artifacts/<run-id>/review-report.md` when a `run-id` is provided,
  else a path the caller names.
- **Golden rule** — never modify source; read, judge, report. Cite `file:line` per finding.

## Acceptance checklist

- [ ] `reviewer.agent.md` created.
- [ ] Tools = `Read, Grep, Glob, Write, Skill`; body states `Write` is report-only, never edits code.
- [ ] Diff comes from the caller (no `Bash`); body says to ask for it when missing.
- [ ] Rubric loads `coding-principles` via `Skill` — not a reinvented principles list.
- [ ] Custom rubric spelled out (the reason it isn't `/code-review`).
- [ ] Findings cite `file:line`; masks nothing it shouldn't; verdict line present.
- [ ] Emits one report artifact matching the §8 artifact pattern.
- [ ] Run `marketplace-sync` after.

## Out of scope

- Fixing findings (hand back to developer via the orchestrator).
- Orchestration / retry loop (feature-pipeline, later task).
