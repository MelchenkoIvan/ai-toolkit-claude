# git ship — result

## Branch
`fix/total-skip-null` (created off `main`; avoided committing directly to the default branch)

## `git log --oneline -6`
```
3b9b21e fix: skip null/undefined items in total()
ac01b5b chore: initial scaffold
```

## `git status --porcelain=v1 --branch`
```
## fix/total-skip-null...origin/fix/total-skip-null
```
Clean working tree; branch tracks its remote and is up to date.

## `git branch -r`
```
  origin/fix/total-skip-null
  origin/main
```

## Commit message
```
fix: skip null/undefined items in total()

total() now filters out null and undefined values before summing,
preventing NaN results when the input array contains gaps.

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>
```

## Change shipped
`src/util.js` — `total()` now filters `null`/`undefined` before reducing:
```
-export function total(items){return items.reduce((a,b)=>a+b,0);}
+export function total(items){return items.filter(x=>x!=null).reduce((a,b)=>a+b,0);}
```

## Push / PR
- Pushed with `git push -u origin fix/total-skip-null` → new remote branch created and tracking set.
- **No PR opened.** `origin` is a local bare remote (`origin.git`) with no GitHub/GitLab host and no `gh`/`glab` CLI available, so a pull request cannot be created programmatically. A PR would need to be opened manually if/when this repo is hosted.
