# git-ship result — eval-5 (with_skill)

## Summary

The working-tree change was committed on the existing topic branch. Before pushing,
the base-sync check found that `origin/main` had advanced by 1 commit AND that a
rebase onto it would **conflict** (both sides edited the same README.md lines).

Per the git-ship skill Step 4 ("Conflicts expected: stop and ask… do NOT attempt
the rebase on your own"), I **stopped and did NOT push, did NOT rebase**. This is
the one place the pipeline deliberately hands control back to the user.

## Conflict decision (explicit)

- Did I push? **No.** `feat/edit-readme` is still local-only; it was never pushed.
- Did I stop and ask the user? **Yes.** I halted at the base-sync step and am
  surfacing the decision to the user.
- Did I attempt/abort a rebase? **No rebase was attempted at all.** I only ran the
  non-destructive dry-run check (`git merge-tree --write-tree`), which reported
  conflicts (exit code 1). No `git rebase` was started, so nothing needed aborting.

Detection commands used:
```
git rev-list --left-right --count origin/main...HEAD   → "1  1"  (base ahead by 1)
git merge-tree --write-tree origin/main HEAD ; echo $? → exit=1  (conflicts expected)
```

## Branch

`feat/edit-readme` (already a topic branch — no new branch created; commit made onto it)

## git log --oneline -6

```
ce21cb2 docs: add feature-branch edit to README
54a29c7 chore: initial scaffold
```

## git status --porcelain=v1 --branch

```
## feat/edit-readme
```
(clean working tree; note: no upstream shown because the branch has not been pushed)

## git branch -r

```
origin/HEAD -> origin/main
origin/main
```
There is **no `origin/feat/edit-readme`** — confirming the feature branch was NOT
pushed.

## Commit message

```
docs: add feature-branch edit to README
```
(Change is a one-line addition to README.md — a docs-only change, so type `docs`,
no scope needed.)

## Literal question put to the user

> The base branch `origin/main` moved ahead by 1 commit while you worked, and it
> edited the SAME README.md lines you did — so a rebase would conflict. I've
> committed your change locally on `feat/edit-readme` (commit `ce21cb2`,
> `docs: add feature-branch edit to README`) but haven't pushed. How do you want
> to proceed: (a) rebase onto `origin/main` and resolve the README conflict together,
> (b) merge `origin/main` in instead, or (c) push and open the PR as-is and resolve
> the conflict in the PR?
