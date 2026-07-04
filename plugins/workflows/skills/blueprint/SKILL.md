---
name: blueprint
description: >
  Use when turning requirements — a PRD, spec, design notes, tag descriptions,
  or free text — into a structured backlog of epics/features/tasks before any
  implementation, and when re-running to refine an existing backlog against
  changed requirements. Triggers when the user says "break down this PRD",
  "plan the work", "create tasks from these requirements", "groom the backlog",
  or invokes `/blueprint`. Use it even when the user only hands over a document
  and asks "what should we build?" — decomposition into a backlog is the job.
---

# Blueprint

Turn fuzzy requirements into two deliverables the implementation pipeline can
consume directly: a **structured backlog** under `backlog/` and an
**`ARCHITECTURE.md`**, both written before any code.

**Core principle:** every task file *is* the `developer` agent's input contract.
A capable model already writes good task *content* unprompted — Given/When/Then
acceptance criteria, dependencies, traceability, an architecture doc. What it
does *not* produce unprompted is a **stable, machine-readable shape**: YAML
frontmatter, a `status` lifecycle, and a folder/ID scheme that survives re-runs.
That shape is what this skill adds. So the skill is a **contract to fill in**,
not a pile of rules — match the template exactly and the pipeline handoff works.

## When to use

- The user hands over a PRD, spec, design, notes, or tag descriptions and wants
  it turned into tasks/features/epics.
- A backlog already exists and requirements changed — refine it, don't rebuild.
- Someone asks "what should we build?" / "plan this out" over a document.

**Not for:** implementing, testing, or reviewing code (that's the `developer` /
`tester` / `reviewer` agents downstream); or a single obvious task — just write
that task, don't scaffold a backlog around it.

## Workflow

1. **Capture intent.** Extract scope, goals, constraints, and explicit content
   from the input. Ask only for genuine gaps; don't invent requirements the
   source is silent on — flag those as "to gather" instead.
2. **Discovery (opt-in `--scan`).** Default off. With `--scan`, dispatch the
   read-only `Explore` agent to map what already exists, so you don't create
   tasks for built features and can fill each task's `paths`.
3. **Read the existing backlog first** if `backlog/` is present (see Grooming).
4. **Decompose** to adaptive depth (below), one task per independently
   shippable unit of work.
5. **Write each task** to the contract below — this is the step that earns the
   skill; get the frontmatter exactly right.
6. **Write `ARCHITECTURE.md`** (below).
7. **Write/refresh `BACKLOG.md`** roll-up and report.

## Output layout — adaptive depth

Written to `backlog/` at the repo root:

```
backlog/
├── ARCHITECTURE.md          ← current → target, components, decisions, mermaid
├── BACKLOG.md               ← index + status roll-up
└── features/
    └── 01-<feature-slug>/
        ├── _feature.md      ← goal, scope, source
        └── tasks/
            └── 01-<task-slug>.md
```

- **Tiny** input (a handful of tasks) → flat `backlog/tasks/` only.
- **Normal** → `features/NN-<slug>/tasks/NN-<slug>.md` (shown above).
- **Large** (many features across themes) → add `epics/NN-<slug>/` on top.

## Task file — the contract (fill this in exactly)

```markdown
---
id: F01-T02              # feature-scoped: F<feature#>-T<task#>. Stable — never renumber on re-run.
title: Validate login form fields
status: todo             # todo | in-progress | done
priority: high           # high | medium | low
estimate: M              # S | M | L
depends_on: [F01-T01]    # task ids this blocks on; [] if none
paths: [src/auth/]       # populated only when --scan ran; feeds developer.paths
source: "PRD §2.3"       # traceability back to the requirement
---

## Description
What & why, 2–4 sentences.

## Acceptance Criteria
- **Given** <context> **When** <action> **Then** <observable outcome>
- **Given** … **When** … **Then** …

## Notes
- Out of scope: <what this task deliberately excludes>
```

`id`, `depends_on`, `paths`, and the Given/When/Then criteria map 1:1 onto what
the `developer` and `tester` agents already read — no translation layer.

**IDs are feature-scoped and stable:** `F01-T02` = feature 01, task 02.
Inserting a task never renumbers existing ones; a task keeps its id for life so
re-runs diff cleanly.

**Acceptance criteria are Given/When/Then**, one scenario per behavior path
(happy path, each error path, each boundary). Keep them observable and testable —
the `tester` agent turns them straight into cases.

## Architecture document

`backlog/ARCHITECTURE.md`: problem & goal → current state → target state →
components → output/data model → key decisions → risks. Use **mermaid** for
architecture and flows. Link each feature back to the section it realizes.

## Grooming — refining an existing backlog

When `backlog/` already exists, **read it before writing anything**. Then:

- Diff the new/changed requirements against what's there; **add or update only
  what changed.**
- **Never rewrite a task whose `status: done`** — it records shipped work; a
  changed requirement becomes a *new* task, not an edit to a done one.
- Keep every existing `id` stable; append new ids rather than renumbering.
- Refresh the `BACKLOG.md` roll-up counts.

Grooming means the skill is safe to run repeatedly — each run refines, never
regenerates.

## Report

End with a short summary the user can act on:

- where the backlog was written (`backlog/`)
- counts: epics / features / tasks created vs. updated, and the status roll-up
- whether `--scan` ran (and so whether `paths` are populated)
- the path to `ARCHITECTURE.md`
