# Task 1 — Developer agent + shared stack skills

> Build in its own conversation. Read `docs/architecture.md` first (full context).
> This task is **foundational** — tasks 2 and 3 depend on the skills built here.

## Goal

Create the `developer` agent (router + implementer) and the shared stack skills it
loads. After this task, someone can hand the developer a task and it implements it
in the right stack.

## Prerequisites

- Read `docs/architecture.md` §2 (skill vs agent), §3 (real skills vs files),
  §4 (pipeline), §5 (routing), §6 (principles).
- Use the `create-agent` skill for the agent file.

## Deliverables

| File | What |
|---|---|
| `plugins/workflows/skills/coding-principles/SKILL.md` | Universal DRY / KISS / YAGNI / naming / error-handling rules |
| `plugins/workflows/skills/react-dev/SKILL.md` | Thin React/TS skill (triggers + index) |
| `plugins/workflows/skills/react-dev/references/components.md` | React conventions |
| `plugins/workflows/skills/react-dev/references/testing.md` | Jest + RTL conventions (used by task 2) |
| `plugins/workflows/skills/dotnet-dev/SKILL.md` | Thin .NET/C# skill (triggers + index) |
| `plugins/workflows/skills/dotnet-dev/references/patterns.md` | .NET conventions |
| `plugins/workflows/skills/dotnet-dev/references/testing.md` | xUnit + Moq conventions (used by task 2) |
| `plugins/workflows/agents/developer.agent.md` | The router + implementer agent |

> Stack skills are **thin**: SKILL.md triggers + points to `references/`. Depth lives
> in the reference files. See `architecture.md` §3.

## Developer agent spec

Frontmatter:
```yaml
---
name: developer
description: "Use this agent to implement a coding task in a specific stack. Detects React vs .NET from repo signals and task text, loads the matching stack skill(s), implements, and returns a change summary. Triggers when an orchestrator hands off an implementation task or a developer asks to build a feature in this codebase."
model: inherit
color: green
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob", "Skill"]
---
```
`Skill` in tools is load-bearing — without it the agent can't pull stack skills.

Body must define:
- **Routing** — inspect task + repo (`package.json` → react-dev, `.csproj` → dotnet-dev).
  Load the *relevant set* (both, if full-stack). Always load `coding-principles`.
- **Input contract** — JSON-shaped task (see `architecture.md` §8): `task`, optional
  `stack`, `paths`, `constraints`. Falls back to repo detection if `stack` absent.
- **Re-entry** — accept the retry input shape from §8 (`attempt`, `previous_summary`,
  `test_failures`, `review_findings`) and fix rather than re-implement blind.
- **Output contract / artifact** — change summary: verdict, stack, files changed,
  verify result, follow-ups (see §8 example). Written to
  `.pipeline-artifacts/<run-id>/` when a `run-id` is provided in the input.
- **Verify ownership** — developer verifies *build + pre-existing tests only*.
  New tests for the change belong to the tester agent — never claim test
  coverage this agent didn't create.
- **Golden rule** — do exactly what the task describes; never touch unrelated code.

## Acceptance checklist

- [ ] All 8 files created at the paths above.
- [ ] Stack SKILL.md files are thin and reference their `references/` files.
- [ ] `react-dev` / `dotnet-dev` descriptions auto-trigger on their stack.
- [ ] developer agent has `Skill` in tools and a routing section that loads the relevant set.
- [ ] Input + output contracts present and match `architecture.md` §8 (including re-entry input).
- [ ] Verify-ownership boundary stated (build + pre-existing tests only; new tests = tester's).
- [ ] Each new skill ships an `evals/evals.json` per repo convention (see `git-ship/evals/`, `create-agent/evals/`).
- [ ] `plugins/workflows/.claude-plugin/plugin.json` exposes agents (`"agents": "./agents/"`) — add if missing.
- [ ] Run `marketplace-sync` skill after (README / manifest / CLAUDE.md tree).

## Out of scope

- tester agent (task 2), reviewer agent (task 3), feature-pipeline orchestrator (later task).
