---
name: analyse-and-plan
description: >
  Deep codebase analysis → strategy discussion → publishable implementation plan.
  Traces code paths, finds reusable patterns, identifies impact, presents
  options with trade-offs, then publishes the plan as a GitHub issue comment
  or local Markdown file. Does NOT write implementation code. Use when the
  user says "analyse and plan", "create plan for", "research before
  implementing", "draft an implementation plan", or invokes
  `/analyse-and-plan`. Always use this skill when the user wants a plan
  before code is written.
---

# Analyse & Plan

Deep analysis → strategy discussion → AI-ready plan. Detailed enough that an implementing skill (e.g., `/implement-task`, `/solve-issue`) can skip exploration and go straight to delegation. **Produces plans, never code.**

---

## Accepted Inputs

- **GitHub issue URL or number** — e.g., `owner/repo#55`.
- **Free-text feature or bug description** — e.g., "add break duration limits".

---

## Phase 1 — Input Processing

Goal: ensure a clear problem statement exists.

### If a GitHub issue exists

```bash
gh issue view <issue-number> --repo <owner>/<repo> --json title,body,labels,assignees,comments
```

Extract title, body, labels, comments, acceptance criteria.

### If the user provided free text

Decide with the user where to publish the plan:
1. **Create a GitHub issue first**, then attach the plan as a comment.
2. **Local Markdown file** (no GitHub issue) — write to a path the user chooses.

If creating an issue:
```bash
gh issue list --repo <owner>/<repo> --search "<keywords>" --state all --limit 10   # dedup check
gh issue create --repo <owner>/<repo> \
  --title "<title>" \
  --body "<problem statement + acceptance criteria>"
```

If using a local Markdown file: ask the user for the target path (default `./plans/<short-desc>.md`).

### For both paths

If the input is vague or missing acceptance criteria, **ask the user before continuing**:
- Clear description of desired behavior or bug
- Acceptance criteria / expected outcomes
- Constraints (perf, security, compatibility)
- Whether the change touches multiple repos

**Do not proceed to Phase 2 until requirements are clear.**

---

## Phase 2 — Deep Codebase Analysis

The core of the workflow. Build a thorough, code-level understanding before proposing approaches.

### 2a — Convention & architecture scan

Read whichever exist in the repo:
- `CLAUDE.md`
- `.github/copilot-instructions.md`
- `.cursor/rules/*.md`, `.cursorrules`
- `README.md` (setup, build, test commands)
- `.github/instructions/*.instructions.md`

Internalize the stack, layering, naming, error handling, and test conventions. If `CLAUDE.md` is missing, infer from the project's manifest files (`package.json`, `*.csproj`, `pyproject.toml`, etc.) and adjacent code.

### 2b — Code tracing

Use the `Agent` tool with `subagent_type: "Explore"` and `model: "haiku"` (multiple in parallel) to investigate. Batch related questions per agent.

**Generic tracing template** — adapt to the stack:

- **For UI / frontend work:** find the affected screen/page → which hooks or stores does it use → which API client functions → which endpoints → which handlers → which data sources.
- **For API / backend work:** find the route/handler → command/query → business logic → data access → entity/model → storage.
- **For background / async work:** find the trigger (cron, queue, event) → handler → side effects → external calls → completion semantics.
- **For cross-service work:** trace the full data flow across service boundaries. Verify request/response shapes match between caller and callee. Check shared constants are in sync.

For each affected file, record:
- **Path**
- **Current role** (what it does)
- **How it relates to the change** (what it would need to do)

### 2c — Pattern discovery

For each repo:
- **Closest existing feature** — find the nearest analogue (existing CRUD if you're adding CRUD; existing list view if adding a list view) and study its layout, naming, layering.
- **Reusable code** — utilities, hooks, components, helpers, validators, mappers, base classes that can be reused or extended.
- **Constants and enums** — identify any that need extending or new ones to add.

Record the **reference implementation** the new code should mirror.

### 2d — Test landscape analysis

For each repo:
- **Existing tests for related features** — find the closest test file as a template.
- **Test infrastructure** — mocking utilities, base classes, in-memory DB / fixture setup.
- **Test commands** — confirm via `CLAUDE.md` / `package.json` scripts / project files.
- **Coverage gaps** — note untested code that would now need coverage.

### 2e — Impact analysis

Determine the full blast radius:
- **Files to modify** — exact list
- **Files to create** — exact list
- **Downstream consumers** — if changing a handler / DTO / response shape, who consumes it?
- **Shared contracts** — does any cross-service contract need updating?
- **Schema changes** — DB migrations needed?
- **Constants / enums** — shared constants needing update?
- **Breaking changes** — anything that risks breaking existing behavior?

---

## Phase 3 — Implementation Strategy Discussion

Discuss the **approach** with the user before drafting the detailed plan. Trade-offs first, plan second.

### Step 1 — Identify viable approaches

Using Phase 2's analysis, list reasonable ways to solve the problem. Consider variations along:
- **Architecture** — new entity vs extending existing; new endpoint vs modifying existing.
- **Scope** — minimal viable change vs comprehensive refactor.
- **Reuse** — build from scratch vs adapt existing patterns.
- **Cross-repo impact** — single-repo vs multi-repo.
- **Data model** — different ways to model the data.

### Step 2 — Present options to the user

**If only one reasonable approach:** present it directly with brief justification.

**If multiple viable approaches:** present up to 3 with trade-offs:

```
Based on my analysis, here are the implementation strategies:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🅰  <Strategy Name>
   <1-2 sentence description>

   ✅ Advantages:
   - <advantage 1>
   - <advantage 2>

   ❌ Disadvantages:
   - <disadvantage 1>
   - <disadvantage 2>

   📁 Estimated scope: <N files>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🅱  <Strategy Name>
   ...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

💡 My recommendation: <which option and why>
```

### Step 3 — Get the user's decision

Accept:
- A direct choice ("go with A")
- A hybrid ("A but with B's data model")
- A different direction (re-analyse and propose again)

### Step 4 — Lock in the strategy

Record the chosen strategy as the foundation for the detailed plan.

**Do not proceed to Phase 4 until the user confirms a strategy.**

---

## Phase 4 — Plan Synthesis

Combine all analysis into a structured plan. Use this template exactly:

```markdown
## 🗺️ Implementation Plan for #<issue-number>

### 1. Context & Understanding

**Issue:** <title>
**Type:** <bug fix / feature / enhancement>
**Affected repos:** <list>

<1-2 paragraph summary of what needs to happen and why>

**Current behavior:** <what happens now>
**Desired behavior:** <what should happen after>

### 2. Codebase Analysis

#### Architecture of the Affected Area
<layers, data flow, key types>

#### Data Flow Trace
```
<Trigger>
  → <Layer 1> (path/to/file:line)
  → <Layer 2> (path/to/file:line)
  → <Layer 3> (path/to/file:line)
  → <Storage>
```

#### Existing Patterns Found
| Pattern | Reference File | How It Applies |
|---|---|---|
| <name> | `path/to/file` | <how the change should follow it> |

#### Reusable Code Identified
- `path/to/utility` — <what it does, how to reuse>

### 3. Implementation Strategy

**Chosen approach:** <strategy from Phase 3>

**Why this approach:**
- <rationale>

**Alternatives considered:**
| Alternative | Why Not Chosen |
|---|---|
| <name> | <reason> |

**Architecture decisions:**
- <decision 1>

### 4. Per-Repository Task Breakdown

#### Task 1: <Repo Name>

**Base branch:** `<branch>`
**Dependencies:** <none / blocks Task N>

**Files to create:**
| File | Purpose | Pattern to Follow |
|---|---|---|
| `path/to/new-file` | <what it does> | Based on `path/to/reference` |

**Files to modify:**
| File | Lines | Change |
|---|---|---|
| `path/to/file` | L42-L65 | <specific change> |

**Implementation steps:**
1. <step — specific, actionable>
2. <step>

**Context for implementer:**
<code snippets, contracts, conventions the implementing agent needs>

### 5. Dependency Order
<for multi-repo changes — which groups serialize, which run in parallel>

### 6. Testing Strategy

| Test Case | Type | Test File | Template |
|---|---|---|---|
| <happy path> | Unit | `path/to/test` | Based on `path/to/existing-test` |
| <negative> | Unit | `path/to/test` | Based on `path/to/existing-test` |
| <edge case> | Unit | `path/to/test` | — |

**Test data / fixtures:**
- <any mock data needed>

**Existing test utilities to reuse:**
- `path/to/helper` — <what it provides>

### 7. Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| <description> | L/M/H | L/M/H | <how to mitigate> |

### 8. Acceptance Criteria Mapping

| Criterion | Implementation | Verification |
|---|---|---|
| <from issue> | <which change satisfies it> | <which test verifies it> |
```

---

## Phase 5 — User Review

**Before publishing**, present the complete plan to the user.

1. Print the full plan.
2. Ask: "Here is the proposed plan. Publish as-is, or refine anything?"
3. If changes requested → apply, re-present, iterate.
4. **Only proceed to Phase 6 after explicit user approval.**

---

## Phase 6 — Publish

### If publishing to a GitHub issue

```bash
gh issue comment <issue-number> --repo <owner>/<repo> --body "$(cat <<'EOF'
<the approved plan from Phase 4>
EOF
)"
```

### If publishing to a local Markdown file

Write the plan to the user-chosen path (default `./plans/<short-desc>.md`).

### Print summary

```
🔍 Analysis & Plan Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━
Subject: <issue ref or local plan path>
Status: ✅ Plan published

📋 Plan location: <issue URL / file path>
🏗️ Repos affected: <list>
📝 Tasks: <count>
🧪 Tests: <count>
⚠️ Risks: <count>

Next: Run /implement-task or /solve-issue to execute this plan.
```

---

## Phase 7 — Optional Handoff

Ask the user whether to proceed with implementation:

> "The plan is published. Hand this off for implementation now?"

- **YES (bug fix):** invoke `/solve-issue` — it picks up the plan from the issue or file.
- **YES (feature):** invoke `/implement-task` — same.
- **NO:** stop. The user will invoke implementation when ready.

---

## Rules

1. **ALWAYS** confirm requirements are clear before analysing.
2. **NEVER** write implementation code — this skill produces plans, not code.
3. **ALWAYS** use parallel exploration agents for multi-area or multi-repo analysis.
4. **ALWAYS** trace code paths to specific file:line references — vague plans are useless.
5. **ALWAYS** include reference implementations (the "pattern to follow") per task.
6. **ALWAYS** include a testing strategy with specific test cases and template references.
7. **ALWAYS** present the plan to the user before publishing.
8. **NEVER** publish or hand off without user approval.
9. If the issue is vague, **ask for clarification** — do not guess.
10. If analysis reveals more complexity than expected, surface that and adjust scope.
