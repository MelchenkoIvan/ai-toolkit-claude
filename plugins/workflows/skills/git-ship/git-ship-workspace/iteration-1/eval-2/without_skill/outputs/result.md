# git-ship eval-2 (without_skill) — outputs

## Branch ended on (`git rev-parse --abbrev-ref HEAD`)
```
fix/total-skip-null
```

## `git log --oneline -5`
```
42bf8af fix: skip null and undefined items in total()
e8b9061 chore: initial scaffold
```

## `git status --porcelain=v1 --branch`
```
## fix/total-skip-null...origin/fix/total-skip-null
```
(working tree clean; branch is up to date with its pushed remote tracking branch)

## `git branch -r`
```
  origin/fix/total-skip-null
  origin/main
```

## Commit message chosen
```
fix: skip null and undefined items in total()

Filter out null/undefined values before reducing so total() no longer
produces NaN when the input array contains missing entries.
```

## PR
No pull request was opened.

- The only uncommitted change was on `main` (the default branch), so I created a
  feature branch `fix/total-skip-null`, committed there, and pushed with
  `-u origin fix/total-skip-null`.
- `origin` is a local bare repository (`/tmp/git-ship-sandboxes/eval-2-without_skill/origin.git`),
  not a hosted GitHub/GitLab remote. `glab` is not installed. `gh` is installed
  but cannot open a PR against a non-GitHub remote:
  ```
  gh pr create --fill
  → none of the git remotes configured for this repository point to a known
    GitHub host. To tell gh about a new GitHub host, please use `gh auth login`
  ```
- The branch is pushed and ready; a PR can be opened once the remote is on a
  hosted platform.

## The actual change shipped
`src/util.js`:
```diff
-export function total(items){return items.reduce((a,b)=>a+b,0);}
+export function total(items){return items.filter(x=>x!=null).reduce((a,b)=>a+b,0);}
```
Filters out `null`/`undefined` entries before summing.
