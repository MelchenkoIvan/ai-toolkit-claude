---
name: git-ship
description: >
  Ship the current working-tree changes end-to-end: pick a Conventional-Commit
  message from the diff, create a correctly-named branch (feat/… or fix/…) if the
  work is still on a base branch, commit, push, and open a pull/merge request —
  autonomously, without asking the user to hand-write any of it. Works with any
  Git host: uses `gh` for GitHub, `glab` for GitLab, and falls back to pushing
  and printing the compare URL when no host CLI exists. Use this skill whenever
  the user says "commit", "commit my changes", "push", "ship it", "ship this",
  "open a PR", "raise an MR", "wrap this up", or invokes `/git-ship` — even when
  they only say "commit" and don't mention branching or PRs, because this skill
  owns the whole path from dirty tree to open PR.
---

# Git Ship

Turn a dirty working tree into an open pull/merge request in one autonomous pass.
The user says "commit" (or "push", "ship it") and you decide the commit message,
the branch name, and everything in between. Do not ask the user to write the
commit message or name the branch — deciding those from the diff *is the job*.

The pipeline is deterministic; only the message and branch name require judgment:

```
inspect → decide message → branch (if needed) → commit → push → open PR/MR → report
```

## Golden rule: read the diff before you write anything

Everything downstream — the commit type, scope, subject, and branch name — comes
from what actually changed. So always start by looking, never by guessing:

```bash
git status --porcelain=v1 --branch   # what's staged/unstaged, current + upstream branch
git diff --stat HEAD                  # shape of the change (files, ± lines)
git diff HEAD                         # the actual content (staged + unstaged)
```

If the diff is large, skim `--stat` first, then read the hunks that carry the
intent (source changes over lockfiles, generated files, or vendored code). You
are trying to answer one question: *what is the smallest true sentence that
describes this change?* That sentence drives both the commit subject and branch.

## Step 1 — Decide the commit message (Conventional Commits)

Format: `type(scope): subject`, then an optional body.

- **type** — the dominant intent of the diff:
  | type | when |
  |---|---|
  | `feat` | adds user-facing capability |
  | `fix` | corrects broken behavior |
  | `refactor` | restructures code, no behavior change |
  | `docs` | docs/comments only |
  | `test` | tests only |
  | `chore` | build, deps, tooling, config |
  | `perf` | performance improvement |
  | `style` | formatting, no logic change |
- **scope** — optional, the touched area (module, package, dir). Infer from paths
  (`plugins/workflows/…` → `workflows`). Omit if the change is cross-cutting or
  a scope would just be noise.
- **subject** — imperative mood, lowercase, no trailing period, aim ≤ 50 chars.
  "add retry to fetch", not "added retry" or "Adds retrying".
- **body** — add only when the *why* isn't obvious from the subject. Wrap ~72
  cols. Bullet the notable changes. Skip it for small, self-evident diffs.

When the diff spans clearly unrelated concerns (e.g. a feature *and* an
unrelated typo fix), prefer one honest message over a dishonest tidy one — pick
the dominant change for the subject and mention the rest in the body. Mixed diffs
are a smell; if it's severe, tell the user rather than forcing one label.

**Examples**

Input: new JWT middleware + login route
Output: `feat(auth): add JWT login and auth middleware`

Input: off-by-one in pagination that dropped the last row
Output: `fix(api): include final row in paginated results`

Input: renamed variables, extracted a helper, no behavior change
Output: `refactor(parser): extract token-normalization helper`

## Step 2 — Branch (only if you're on a base branch)

Check the current branch from the `git status --branch` output.

- **If already on a topic branch** (not `main`/`master`/`develop`): stay on it.
  Don't create a branch per commit — that fragments a PR.
- **If on a base branch** (`main`, `master`, `develop`, `trunk`): the user should
  not commit here directly, so create a branch first. This is the "create a
  branch if it doesn't exist" behavior — the topic branch for this work doesn't
  exist yet, so make it.

Branch name: `<prefix>/<slug>`
- **prefix** — `feat` when the commit type is `feat`; `fix` when it's `fix`; for
  everything else use the commit type itself as the prefix (`refactor/…`,
  `docs/…`, `chore/…`). Default to `feat` when genuinely ambiguous — most work is
  additive.
- **slug** — the commit subject, kebab-cased, ≤ ~40 chars, no trailing filler
  words. `feat(auth): add JWT login` → `feat/add-jwt-login`.
- If a ticket ID is obvious from context (branch discussion, prompt), include it:
  `feat/PROJ-123-add-jwt-login`. Never invent one.

```bash
git switch -c feat/add-jwt-login    # branch off current HEAD, then commit onto it
```

If a branch by that name already exists, append a short disambiguator
(`-2`, or a keyword) rather than clobbering it.

## Step 3 — Stage and commit

Stage intentionally. If the user said "commit" with a dirty tree, they mean the
visible work — `git add -A` is usually right, but first scan `git status` for
things that shouldn't be committed (secrets, `.env`, large binaries, stray debug
files, unrelated local scratch). If you see any, stage the intended paths
explicitly instead and note what you skipped.

```bash
git add -A
git commit -m "feat(auth): add JWT login and auth middleware"
```

For a body, use repeated `-m` flags (`-m subject -m body`) or a heredoc — never
embed literal `\n` in a single `-m`.

## Step 4 — Sync with the base branch (only if it moved ahead)

Before pushing, make sure your branch isn't stale relative to the base. If `main`
advanced while you worked, pushing a branch that forked from an old base leads to
a messy PR and surprise merge conflicts later — catching it now is cheaper.

```bash
git fetch origin                                   # refresh remote refs
git rev-list --left-right --count origin/<base>...HEAD
# output "A<TAB>B": A = commits on base you DON'T have, B = your commits ahead
```

If **A is 0**, you're up to date — skip straight to push.

If **A > 0**, the base has commits you're missing. Decide by whether a rebase
would conflict, and let that decide who's in control:

```bash
git merge-tree --write-tree origin/<base> HEAD >/dev/null 2>&1; echo $?
# exit 0 → clean merge/rebase possible;  exit 1 → conflicts expected
```

- **Clean (no conflicts expected):** rebase onto the updated base yourself — it's
  safe and keeps history linear. No need to bother the user for a mechanical,
  conflict-free update.
  ```bash
  git rebase origin/<base>
  ```
- **Conflicts expected:** stop and ask. Do **not** attempt the rebase on your own
  — resolving conflicts needs the user's judgment about intent. Tell them the base
  moved, that a rebase would conflict, and let them choose (rebase with their
  help, merge, or ship as-is). This is the one place the pipeline deliberately
  hands control back.

If you rebased a branch that was **already pushed**, its history changed, so the
next push needs `--force-with-lease` (safe: it refuses if the remote moved under
you). A brand-new branch pushes normally.

## Step 5 — Push

Push and set upstream on first push of a new branch:

```bash
git push -u origin HEAD
```

If the push is rejected because the remote moved, do **not** force-push a shared
branch. Report the rejection and let the user decide (rebase vs. merge). A plain
`--force` on a branch someone else may have is destructive; `--force-with-lease`
is the safer tool if a force is truly warranted and the branch is yours.

## Step 6 — Open the PR / MR

Detect the host from the remote and use the matching CLI. PR/MR creation is an
outward-facing action, but the user asked for it as part of "ship it," so open a
**draft** by default when the CLI supports it — it makes the intent public
without pinging reviewers, and the user can mark it ready.

```bash
git remote get-url origin    # inspect host: github.com → gh, gitlab.* → glab
```

**GitHub (`gh`):**
```bash
gh pr create --draft --fill --base <base>
# --fill seeds title/body from commits; refine the body if the commit body is thin
```

**GitLab (`glab`):**
```bash
glab mr create --draft --fill --target-branch <base>
```

- **base branch** — the branch you diverged from (usually `main`; confirm with
  `git symbolic-ref refs/remotes/origin/HEAD` or repo convention). Don't target
  `develop` unless that's clearly the trunk here.
- **title** — the commit subject (or a summary spanning multiple commits).
- **body** — what changed and why, in the reviewer's terms. If there are several
  commits, summarize the arc, don't just concatenate subjects.

**No host CLI available:** push already happened, so surface the ready-made
compare URL for the user to open manually, e.g.
`https://github.com/<org>/<repo>/compare/<base>...<branch>?expand=1`, or the
`Create merge request` URL Git prints in the push output for GitLab.

## Step 7 — Report

End with a tight summary the user can act on:

- branch name (and whether it was newly created)
- the commit message you used
- push result
- PR/MR link (or the compare URL fallback)

## Guardrails (why these matter)

- **Never commit on a base branch.** Direct commits to `main` bypass review and
  are painful to unwind — that's the whole reason Step 2 exists.
- **Never force-push a shared branch** without explicit say-so. It rewrites
  history others may have pulled; `--force-with-lease` on your own branch only.
- **Never resolve rebase conflicts unattended.** A clean, conflict-free sync onto
  an advanced base is fine to do silently; the moment conflicts are in play, stop
  and let the user decide — auto-resolving risks silently discarding their work.
- **Don't fabricate.** No invented ticket IDs, issue links, co-authors, or
  "fixes #N" unless the reference is real and present in context.
- **Respect the tree.** Don't `git add -A` past obvious secrets or junk; stage
  the real work and say what you left out.
- **Empty tree = stop.** If `git status` is clean, there's nothing to ship — say
  so instead of creating an empty commit.
- **One logical change per branch.** If the diff is a grab-bag, flag it; don't
  paper over it with a vague message.

## Nothing-to-do cases

- Clean working tree → tell the user, don't invent a commit.
- Detached HEAD → stop and explain; branch decisions are ambiguous there.
- No `origin` remote → commit locally, then report that push/PR need a remote.
