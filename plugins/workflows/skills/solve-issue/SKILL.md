---
name: solve-issue
description: >
  End-to-end bug fix pipeline for a GitHub issue. Reproduces, finds root cause,
  creates a worktree, implements the fix via the bundled `developer` agent,
  reviews via the bundled `reviewer` agent, and opens a pull request linked
  with `Fixes #N`. Use when the user says "solve issue #N", "fix bug #N",
  "investigate this bug", "fix <description>", or invokes `/solve-issue`.
  Always use this skill for any end-to-end bug-fix request — never just patch
  files inline.
---

# Solve Issue

End-to-end bug-fix pipeline: prepare → worktree → reproduce → root-cause → plan → implement → review → PR. The skill orchestrates and delegates the heavy lifting to bundled `developer` and `reviewer` sub-agents.

---

## Accepted Inputs

- **GitHub issue URL or number** — e.g., `owner/repo#42`.
- **Free-text bug description** — e.g., "the assignee picker doesn't update". A GitHub issue will be created in Phase 1.
- **Structured bug analysis** — e.g., output from another skill that already has the diagnosis.

---

## Phase 1 — Issue Preparation

Goal: ensure a tracked GitHub issue exists with reproduction steps.

### If the user provided free text (no issue yet)

1. Determine the target repo (default = current dir's repo via `gh repo view --json nameWithOwner -q .nameWithOwner`).

2. Check for duplicates:
   ```bash
   gh issue list --repo <owner>/<repo> --search "<keywords>" --state all --limit 10
   ```
   If a likely duplicate exists, surface it and ask whether to link or create new.

3. Create the issue:
   ```bash
   gh issue create --repo <owner>/<repo> \
     --title "<title>" \
     --body "<reproduction steps + expected vs actual + impact>" \
     --label "bug"
   ```

### If a GitHub issue already exists

```bash
gh issue view <issue-number> --repo <owner>/<repo> --json title,body,labels,assignees,comments
```

Extract title, body, labels, comments, repro steps, error logs, environment info.

### For both paths

If the issue lacks a clear repro or expected behavior, **ask the user** before continuing.

**Do not proceed to Phase 2 until you can describe the bug in one sentence and have at least a hypothesis about where to look.**

---

## Phase 2 — Worktree Setup

Goal: create a dedicated git worktree.

### Step 1 — Choose paths

```bash
REPO_ROOT=$(git rev-parse --show-toplevel)
REPO_NAME=$(basename "$REPO_ROOT")
WORKTREES_ROOT="$(dirname "$REPO_ROOT")/worktrees"
SHORT_DESC="<kebab-case, max 4 words>"
WORK_TYPE="fix"
WORKTREE_PATH="$WORKTREES_ROOT/$REPO_NAME/$WORK_TYPE/<issue-number>-$SHORT_DESC"
BRANCH="$WORK_TYPE/<issue-number>-$SHORT_DESC"
```

### Step 2 — Detect base branch

```bash
BASE_BRANCH=$(gh repo view --json defaultBranchRef -q .defaultBranchRef.name)
```

If `CLAUDE.md` documents a different convention (e.g., PRs target `development`), prefer that.

### Step 3 — Create the worktree

```bash
mkdir -p "$WORKTREES_ROOT/$REPO_NAME/$WORK_TYPE"
cd "$REPO_ROOT"
git fetch origin "$BASE_BRANCH"
git worktree add "$WORKTREE_PATH" -b "$BRANCH" "origin/$BASE_BRANCH"
```

### Step 4 — Install dependencies

Detect from manifest files in the worktree:
- `package.json` → `npm install` (or pnpm/yarn if their lockfile is present)
- `*.csproj` / `*.sln` → `dotnet restore`
- `pyproject.toml` / `requirements.txt` → project's documented setup
- `Cargo.toml` → `cargo build`
- `go.mod` → `go mod download`

```bash
cd "$WORKTREE_PATH" && <install-command>
```

### Error handling

- Branch / worktree exists → ask the user (reuse vs remove and re-create).
- Path exists but isn't a valid worktree → ask before removing.

**Do not proceed until worktree is set up and dependencies installed.**

---

## Phase 3 — Reproduce and Locate Root Cause

Goal: confirm the bug exists, then find the smallest change that fixes it.

### Step 1 — Read repo conventions

In the worktree, read whichever exist:
- `CLAUDE.md`
- `.github/copilot-instructions.md`
- `.cursor/rules/*.md`, `.cursorrules`
- `README.md`
- `.github/instructions/*.instructions.md`

### Step 2 — Reproduce

If a repro path is documented in the issue, walk it. Otherwise:
1. Trace from the entry point (UI handler, API endpoint, CLI command, scheduled job) toward the symptom.
2. Add temporary logging or use a debugger if the codebase supports it.
3. Confirm you can trigger the bug deterministically before changing anything.

If you cannot reproduce, **stop and ask the user** for more environment details — don't ship a guess.

### Step 3 — Explore in parallel

Use the `Agent` tool with `subagent_type: "Explore"` and `model: "haiku"` (parallel) to investigate:
- Code path from entry to bug
- Related callers / consumers that might also be affected
- Existing tests covering the area
- Similar past fixes (`git log -S` or `--grep` for related keywords)

### Step 4 — Pinpoint the root cause

State the root cause in one sentence with file:line references. Distinguish:
- **Symptom** — what the user sees
- **Proximate cause** — the immediate broken behavior
- **Root cause** — the underlying defect

Fix the root cause unless it's prohibitively risky; if you fix the proximate cause, document the deeper issue in the PR body.

---

## Phase 4 — Fix Plan

Goal: structured plan posted to the issue before code changes.

### Step 1 — Draft the plan

```
## Fix Plan

### Problem
<symptom — with file:line references>

### Root Cause
<the underlying defect>

### Fix
- **Files:** list
- **Changes:** what to do in each
- **Tests:** regression test that fails before, passes after
- **Risk:** what else might be affected

### Out of Scope
<related issues we won't fix in this PR>
```

For multi-repo bugs (rare for general use, supported): split per repo, identify dependency order. Tasks within a group run in parallel; groups serialize.

### Step 2 — Post the plan to the issue

```bash
gh issue comment <issue-number> --repo <owner>/<repo> --body "$(cat <<'EOF'
## 🗺️ Fix Plan

<paste the plan>
EOF
)"
```

---

## Phase 5 — Delegate Implementation

Goal: hand the plan to the `developer` agent.

### Step 1 — Build the task package

```
**Repository:** <name>
**Worktree path:** <absolute path>
**Work type:** fix
**Issue number:** <N>
**Base branch:** <base>
**Branch (already created):** <branch>

**Bug:** <one-sentence symptom>
**Root cause:** <one-sentence root cause with file:line>

**Fix:**
1. <step>
2. <step>

**Affected files:**
- `path/to/file1` — <change>

**Regression test:**
<what test to add and where>

**Context:**
<related code, conventions, anything the agent shouldn't have to re-discover>
```

### Step 2 — Invoke the developer agent

The agent file is bundled at `<plugin-root>/agents/developer.agent.md`. Use the `Read` tool to read it, then invoke the `Agent` tool with the content prepended to the prompt as system context:

```
1. Use the `Read` tool to read `<plugin-root>/agents/developer.agent.md`.
2. Invoke the `Agent` tool with:
     subagent_type: "general-purpose"
     model: "sonnet"
     prompt: <agent file content> + "\n\n---\n\n" + <the task package from Step 1>
```

For independent multi-repo fixes: spawn parallel `Agent` calls (optionally `run_in_background: true`). For dependent: serialize.

### Step 3 — Collect results

Branch, files changed, build/test status, commit SHA, push status. If a build/test fails, re-invoke with corrected instructions or revise the plan.

**Do not proceed to Phase 6 until builds and tests are green.**

---

## Phase 6 — Code Review

### Step 1 — Get the diff

```bash
cd "$WORKTREE_PATH"
git diff "origin/$BASE_BRANCH..HEAD"
```

### Step 2 — Invoke the reviewer agent

Bundled at `<plugin-root>/agents/reviewer.agent.md`.

```
1. Use the `Read` tool to read `<plugin-root>/agents/reviewer.agent.md`.
2. Invoke the `Agent` tool with:
     subagent_type: "general-purpose"
     model: "sonnet"
     prompt: <agent file content>
             + "\n\n---\n\nReview the following diff. Bug being fixed: <one-sentence>. Root cause: <one-sentence>. Diff: <paste>. Follow your full review protocol — pay extra attention to whether the regression test would have caught the bug."
```

### Step 3 — Process findings

Critical / High must be fixed before PR. Re-invoke developer with fix instructions, re-run review.

**Maximum 3 review rounds.** Persistent findings → list as unresolved in the PR body.

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

If tests failed in any repo, **do not create a PR** — report and stop.

### Step 1 — Verify and push

```bash
cd "$WORKTREE_PATH"
git status                                    # clean
git log --oneline "origin/$BASE_BRANCH..HEAD" # commits exist
git push -u origin "$BRANCH"
```

### Step 2 — Open the PR

```bash
gh pr create \
  --repo "<owner>/<repo>" \
  --base "$BASE_BRANCH" \
  --head "$BRANCH" \
  --title "fix: <brief description> (#<issue-number>)" \
  --label "bug" \
  --body "$(cat <<'EOF'
## Summary

Fixes #<issue-number>

### Root Cause
<one-sentence root cause with file:line>

### Changes
- `path/to/file1` — <change>
- `path/to/file2` — <change>

### Regression Test
<what test was added; what it asserts>

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
gh pr checks <pr-number> --repo <owner>/<repo>
```

### Step 4 — Multi-repo PRs (if applicable)

- One PR per repo.
- Use `Fixes #<N>` only on the primary PR; `Related to #<N>` on the others.
- Cross-reference each PR body:
  ```
  > **Related PRs:**
  > - <owner>/<repo-a>#<pr>
  > - <owner>/<repo-b>#<pr>
  ```

---

## Phase 8 — Convention Maintenance (Optional)

If the fix changed any of:
- API endpoints, request/response shapes
- Domain entities, enums, shared constants
- Project structure
- Build/test/run commands
- Auth schemes

…update `CLAUDE.md` (or the equivalent convention doc) in a follow-up commit on the same branch:

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

If nothing changed at the convention level, skip.

---

## Output Summary

```
🔧 Issue Resolution Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━
Issue: <owner>/<repo>#<N> — "<Title>"
Status: ✅ Resolved

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
3. **ALWAYS** reproduce the bug before fixing it. If you can't reproduce, ask the user.
4. **ALWAYS** add a regression test that fails before and passes after the fix (unless the codebase has no test infrastructure — note this in the PR).
5. Run all phases in order — repro/root-cause (Phase 3), plan (Phase 4), implementation (Phase 5), review (Phase 6) are mandatory.
6. **ALWAYS** open the PR automatically in Phase 7 — do not ask the user to run a separate skill.
7. Use `Fixes #N` (not `Closes #N`) — bug fixes auto-close the issue with the keyword that records the fix relationship.
8. Every commit includes the `Co-Authored-By: Claude <noreply@anthropic.com>` trailer.
9. Use `fix/` branch prefix and `fix:` commit prefix.
10. Worktree path: `<repo-parent>/worktrees/<repo-name>/fix/<issue-number>-<short-desc>`.
11. Never `git push --force`. Use `git push --force-with-lease` after amends.
12. Never hardcode `main` as base branch — always detect.
