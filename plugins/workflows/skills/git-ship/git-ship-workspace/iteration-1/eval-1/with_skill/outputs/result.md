# git-ship — eval-1 (with_skill) outputs

## Branch ended on (`git rev-parse --abbrev-ref HEAD`)
```
feat/add-login
```
Newly created off `main` (never committed to the base branch).

## `git log --oneline -5`
```
1ecf828 feat(auth): add login with token issuance
babb00a chore: initial scaffold
```

## `git status --porcelain=v1 --branch`
```
## feat/add-login...origin/feat/add-login
```
Clean tree; branch tracks and is in sync with its pushed upstream.

## `git branch -r` (remote branches — confirms push)
```
  origin/feat/add-login
  origin/main
```
`origin/feat/add-login` present → push succeeded.

## Commit message chosen
```
feat(auth): add login with token issuance
```
Rationale: the only change was a new `src/login.js` exporting a `login(user, pass)`
function that validates credentials and returns a token — a user-facing capability,
hence `feat`, scoped to `auth`.

## How I'd open the PR (fallback)
No usable host CLI: origin is a local bare path
(`/tmp/git-ship-sandboxes/eval-1-with_skill/origin.git`), and `gh pr create` refused
("none of the git remotes point to a known GitHub host"); `glab` is not installed.
Per the skill's fallback, the branch is already pushed, so the PR would be opened
manually against base `main` — e.g. on a real GitHub host this would be the compare
URL `<host>/<org>/<repo>/compare/main...feat/add-login?expand=1`. Against this local
bare remote, merging is done directly (e.g. `git switch main && git merge feat/add-login`).
