---
name: implement-task
description: >
  End-to-end feature implementation pipeline for a GitHub issue. Creates a git
  worktree, explores the codebase, plans, delegates implementation to the
  bundled `developer` agent, runs review via the bundled `reviewer` agent, and
  opens a pull request. Use when the user says "implement task", "implement
  issue #N", "implement <feature>", invokes `/implement-task`, or asks to
  implement a GitHub issue end-to-end. Always use this skill for any
  end-to-end feature implementation request — never just patch files inline.
---

# Implement Task

End-to-end feature implementation: prepare → worktree → explore → plan → implement → review → PR. The skill orchestrates the pipeline and delegates the heavy lifting to two bundled sub-agents (`developer` and `reviewer`) so the main conversation stays oriented around planning and review, not raw editing.

---

## Accepted Inputs

- **GitHub issue URL or number** — e.g., `owner/repo#55` or `https://github.com/owner/repo/issues/55`.
- **Free-text feature description** — e.g., "add a review system for completed tasks". A GitHub issue will be created in Phase 1.

Features are always tracked by a GitHub issue. If none exists, one is created in Phase 1.

---

## Phase 1 — Task Preparation

Goal: ensure a tracked GitHub issue exists with clear acceptance criteria.

### If the user provided free text (no issue yet)

1. Determine the target repo. Default = current directory's repo (`gh repo view --json nameWithOwner -q .nameWithOwner`). If the change clearly belongs to a different repo, ask the user.

2. Check for duplicates:
   ```bash
   gh issue list --repo <owner>/<repo> --search "<title keywords>" --state all --limit 10
   ```
   If a likely duplicate is found, surface it to the user and ask whether to link to it or create a new one.

3. Create the issue:
   ```bash
   gh issue create --repo <owner>/<repo> \
     --title "<title>" \
     --body "<structured body — feature description, acceptance criteria, constraints>" \
     --label "enhancement"
   ```

### If a GitHub issue already exists

Read the issue and its comments:
```bash
gh issue view <issue-number> --repo <owner>/<repo> --json title,body,labels,assignees,comments
```

Extract: title, body, labels, acceptance criteria, affected areas.

### For both paths

If the issue is vague or missing acceptance criteria, **ask the user before continuing**. Specifically check for:
- Clear description of the desired behavior
- Acceptance criteria / expected outcomes
- Any constraints (perf, security, compatibility)
- Whether the feature touches multiple repos

**Do not proceed to Phase 2 until requirements are clear.**

---

## Phase 2 — Worktree Setup

Goal: create a dedicated git worktree so the main checkout stays untouched and parallel runs don't collide.

### Step 1 — Choose paths

Detect repo root and parent (worktrees go in a sibling directory):
```bash
REPO_ROOT=$(git rev-parse --show-toplevel)
REPO_NAME=$(basename "$REPO_ROOT")
WORKTREES_ROOT="$(dirname "$REPO_ROOT")/worktrees"
SHORT_DESC="<kebab-case, max 4 words, derived from issue title>"
WORK_TYPE="feat"
WORKTREE_PATH="$WORKTREES_ROOT/$REPO_NAME/$WORK_TYPE/<issue-number>-$SHORT_DESC"
BRANCH="$WORK_TYPE/<issue-number>-$SHORT_DESC"
```

### Step 2 — Detect base branch

```bash
BASE_BRANCH=$(gh repo view --json defaultBranchRef -q .defaultBranchRef.name)
```

If the project's `CLAUDE.md` documents a different convention (e.g., PRs target `development`), prefer that. Verify the branch exists:
```bash
git show-ref --verify --quiet "refs/remotes/origin/$BASE_BRANCH" || \
  BASE_BRANCH=$(gh repo view --json defaultBranchRef -q .defaultBranchRef.name)
```

### Step 3 — Create the worktree

```bash
mkdir -p "$WORKTREES_ROOT/$REPO_NAME/$WORK_TYPE"
cd "$REPO_ROOT"
git fetch origin "$BASE_BRANCH"
git worktree add "$WORKTREE_PATH" -b "$BRANCH" "origin/$BASE_BRANCH"
```

### Step 4 — Install dependencies

Detect the package manager and run install:
- `package.json` present → `npm install` (or `pnpm install`, `yarn install` if their lockfile is present)
- `*.csproj` / `*.sln` present → `dotnet restore`
- `pyproject.toml` / `requirements.txt` → `pip install -r requirements.txt` or the project's documented setup
- `Cargo.toml` → `cargo build` (downloads deps)
- `go.mod` → `go mod download`

Run inside the worktree:
```bash
cd "$WORKTREE_PATH" && <install-command>
```

If `CLAUDE.md` documents a different setup procedure, follow that.

### Error handling

- **Branch already exists with a worktree** → ask the user: reuse or remove + re-create.
- **Branch exists without a worktree** → `git worktree add "$WORKTREE_PATH" "$BRANCH"` (no `-b`).
- **Path already exists, not a valid worktree** → ask the user before removing.

**Do not proceed to Phase 3 until the worktree is created and dependencies installed.**

---

## Phase 3 — Codebase Exploration

Goal: build enough context to plan accurately.

### Step 1 — Read repo conventions

Read whichever exist in the worktree:
- `CLAUDE.md`
- `.github/copilot-instructions.md`
- `.cursor/rules/*.md`, `.cursorrules`
- `README.md` (setup, commands)
- `.github/instructions/*.instructions.md`

Internalize stack, layering, naming, test conventions.

### Step 2 — Explore in parallel

Use the `Agent` tool with `subagent_type: "Explore"` and `model: "haiku"` (multiple in parallel) to investigate:
- Existing patterns near the feature area
- Reusable code (utilities, hooks, components, validators, mappers)
- Integration points (API contracts, shared constants, data models)
- Test patterns and test placement

Batch related questions per agent. 1–3 agents in parallel is the sweet spot.

### Step 3 — Identify scope

After exploration, list:
- Files to create / modify
- Whether shared constants or contracts need updating
- Cross-cutting concerns (auth, validation, logging) that apply

---

## Phase 4 — Implementation Plan

Goal: produce a structured plan posted to the GitHub issue before any code is written.

### Step 1 — Draft the plan

```
## Implementation Plan

### Feature
<what is being built and why>

### Architecture Decisions
<key design choices, new entities/endpoints/screens, why this approach>

### Tasks
- **Files:** list of files to create or modify
- **Changes:** what to do in each
- **Tests:** what tests to write
- **Dependencies:** ordering, if any

### Risk Assessment
<potential side effects, areas that might break>
```

For multi-repo features (rare for general use, but supported): split tasks per repo and identify dependency order between repos. Tasks within the same group run in parallel; groups are serialized.

### Step 2 — Post the plan to the issue

```bash
gh issue comment <issue-number> --repo <owner>/<repo> --body "$(cat <<'EOF'
## 🗺️ Implementation Plan

<paste the structured plan>
EOF
)"
```

---

## Phase 5 — Delegate Implementation

Goal: hand the plan to the `developer` agent and let it implement, build, test, commit, and push.

### Step 1 — Build the task package

For each task (most features = one task), assemble:

```
**Repository:** <name>
**Worktree path:** <absolute path>
**Work type:** feat
**Issue number:** <N>
**Base branch:** <base>
**Branch (already created):** <branch>

**Task:** <detailed description>

**Affected files:**
- `path/to/file1` — <create/modify what>
- `path/to/file2` — <create/modify what>

**Implementation steps:**
1. <step>
2. <step>

**Context:**
<related code snippets, API contracts, conventions to respect>
```

Include enough context that the developer agent can work without re-exploring.

### Step 2 — Invoke the developer agent

The agent file is bundled at `<plugin-root>/agents/developer.agent.md`. Use the `Read` tool to read it, then invoke the `Agent` tool with the content prepended to the prompt as system context:

```
1. Use the `Read` tool to read `<plugin-root>/agents/developer.agent.md`.
2. Invoke the `Agent` tool with:
     subagent_type: "general-purpose"
     model: "sonnet"
     prompt: <agent file content> + "\n\n---\n\n" + <the task package from Step 1>
```

For independent multi-repo tasks: spawn multiple `Agent` calls in the same response (parallel, optionally `run_in_background: true`). For dependent tasks: serialize.

### Step 3 — Collect results

Each developer agent reports: branch name, files changed, test results, commit SHA, push status.

If any agent reports a build/test failure:
1. Analyze the error.
2. Either re-invoke the developer with corrected instructions OR revise the plan and re-delegate.
3. Do not proceed to Phase 6 until builds and tests are green.

---

## Phase 6 — Code Review

Goal: catch correctness, duplication, security, and convention issues before opening the PR.

### Step 1 — Get the diff

```bash
cd "$WORKTREE_PATH"
git diff "origin/$BASE_BRANCH..HEAD"
```

### Step 2 — Invoke the reviewer agent

Agent bundled at `<plugin-root>/agents/reviewer.agent.md`.

```
1. Use the `Read` tool to read `<plugin-root>/agents/reviewer.agent.md`.
2. Invoke the `Agent` tool with:
     subagent_type: "general-purpose"
     model: "sonnet"
     prompt: <agent file content>
             + "\n\n---\n\nReview the following diff. Feature: <brief>. Diff: <paste>. Follow your full review protocol."
```

For multi-repo: one reviewer call per repo (parallel).

### Step 3 — Process findings

If findings exist:
1. Group by severity. Critical / High must be fixed before PR.
2. Build fix instructions per finding.
3. Re-invoke the developer agent with the fix package.
4. Re-run the review.

**Maximum 3 review rounds.** If issues persist after 3, list unresolved findings in the PR body.

### Step 4 — Post review summary on the issue

```bash
gh issue comment <issue-number> --repo <owner>/<repo> --body "$(cat <<'EOF'
## 🔍 Code Review

**Rounds:** <N>/3
**Verdict:** ✅ Approved / ⚠️ Approved with notes / 🔴 Unresolved findings

### Notes
<observations>
EOF
)"
```

---

## Phase 7 — PR Creation

Goal: open a pull request linked to the issue.

If tests failed in any repo, **do not create a PR** — report and stop.

### Step 1 — Verify the worktree is clean and pushed

```bash
cd "$WORKTREE_PATH"
git status                                    # should be clean
git log --oneline "origin/$BASE_BRANCH..HEAD" # confirm commits exist
git push -u origin "$BRANCH"                  # idempotent
```

### Step 2 — Open the PR

```bash
gh pr create \
  --repo "<owner>/<repo>" \
  --base "$BASE_BRANCH" \
  --head "$BRANCH" \
  --title "feat: <brief description> (#<issue-number>)" \
  --label "enhancement" \
  --body "$(cat <<'EOF'
## Summary

Closes #<issue-number>

### Feature Description
<brief — derived from implementation summary>

### Implementation Details
- `path/to/file1` — <what was done>
- `path/to/file2` — <what was done>

### Testing
- Build: <result>
- Tests: <result>

### AI Review
- Rounds: <N>
- Verdict: <approved / approved with notes / unresolved>
EOF
)"
```

### Step 3 — Verify

```bash
gh pr view <pr-number> --repo <owner>/<repo>
gh pr checks <pr-number> --repo <owner>/<repo>   # optional CI status
```

### Step 4 — Multi-repo PRs (if applicable)

For multi-repo features:
- Open one PR per repo.
- Use `Closes #<issue-number>` only on the primary PR; use `Related to #<issue-number>` on the others (avoids duplicate auto-close).
- Edit each PR body to cross-reference the others:
  ```
  > **Related PRs:**
  > - <owner>/<repo-a>#<pr>
  > - <owner>/<repo-b>#<pr>
  ```

---

## Phase 8 — Convention Maintenance (Optional)

If the implementation introduced or changed any of:
- API endpoints, request/response shapes
- Domain entities, enums, shared constants
- Project structure (new top-level dirs, renamed key files)
- Testing patterns or build commands
- Auth schemes

…then `CLAUDE.md` (or the repo's equivalent convention doc) is now slightly stale. Read the diff, decide if an update is needed, and if yes, update `CLAUDE.md` in a follow-up commit on the same branch:

```bash
cd "$WORKTREE_PATH"
# edit CLAUDE.md
git add CLAUDE.md
git commit -m "$(cat <<'EOF'
docs: update CLAUDE.md to reflect <change>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
git push
```

If nothing changed at the convention level, skip this phase.

---

## Output Summary

Print a final summary:

```
🚀 Feature Implementation Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Issue: <owner>/<repo>#<N> — "<Title>"
Status: ✅ Implemented

📋 Plan: Posted as issue comment
💻 Implementation:
   - Branch: <branch>
   - Files: <count>
   - Build/Tests: ✅
🔍 Review: <verdict> (<N> rounds)
🔀 Pull Request: <owner>/<repo>#<pr> — <PR URL>

🌳 Worktree (clean up after merge):
   <worktree-path>
   To remove: cd <repo-root> && git worktree remove <worktree-path>
```

---

## Rules

1. **ALWAYS** ensure a GitHub issue exists before starting.
2. **ALWAYS** create a worktree in Phase 2 — never edit the main checkout directly.
3. **ALWAYS** install dependencies in the worktree before delegating.
4. Run all phases in order — exploration → planning → implementation → review are mandatory.
5. **ALWAYS** open the PR automatically in Phase 7 — do not ask the user to run a separate skill.
6. If requirements are unclear, **ask** before implementing.
7. Keep the GitHub issue updated with progress comments (plan in Phase 4, review in Phase 6).
8. Every commit includes the `Co-Authored-By: Claude <noreply@anthropic.com>` trailer.
9. Use `feat/` branch prefix and `feat:` commit prefix for features.
10. Worktree path: `<repo-parent>/worktrees/<repo-name>/feat/<issue-number>-<short-desc>`.
11. Never `git push --force`. Use `git push --force-with-lease` after amends.
12. Never hardcode `main` as the base branch — always detect it.
