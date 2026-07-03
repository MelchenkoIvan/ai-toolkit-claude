---
name: marketplace-sync
description: >
  Keeps the ai-toolkit-claude marketplace repo in a consistent state after you add,
  remove, or edit a skill, agent, or plugin. Syncs the README plugins/skills table,
  marketplace.json, the affected plugin.json (version bump + keywords + description),
  and the CLAUDE.md structure tree so nothing drifts. Use this WHENEVER a skill, agent,
  or plugin under plugins/ is created, renamed, moved, or deleted — including right after
  the create-agent or create-skill workflows finish, since a finished create-* run is a
  strong signal that new content just landed and the marketplace metadata is now stale.
  Also trigger when the user says things like "sync the marketplace", "update the readme",
  "bump the plugin version", "I just added a skill", or "keep the repo in the right state".
---

# marketplace-sync

Purpose: after any change to what the repo _ships_ (a skill, an agent, or a whole
plugin), several metadata files must be updated by hand or they silently drift out of
sync with reality. This skill is the single source of truth for that reconciliation. It
edits files only — it does not commit, branch, or open PRs (leave that to the user or the
git-ship skill).

## Why this matters

The truth about what the repo contains lives in the filesystem: the directories under
`plugins/*/skills/`, `plugins/*/agents/`, and each `plugin.json`. But that truth is
_also_ copied into human- and tool-facing metadata: the README table, the marketplace
registry, and CLAUDE.md's docs. When someone adds a skill and forgets to update those
copies, the marketplace lies about itself. (This has already happened once — a skill was
added to `workflows` but never made it into the README table.) Your job is to make the
copies match the filesystem again.

## The sync procedure

Work in this order. Each step is "read the ground truth from the filesystem, then fix the
metadata that describes it."

### 1. Establish ground truth

Enumerate what actually exists right now:

```bash
# every skill (plugin/skill name)
find plugins -path '*/skills/*/SKILL.md' | sed 's#plugins/\([^/]*\)/skills/\([^/]*\)/SKILL.md#\1 -> \2#'
# every agent
find plugins -name '*.agent.md'
# every plugin manifest
find plugins -name plugin.json
```

Ignore anything under a `*-workspace/` directory — those are eval artifacts from
skill-creator, not shipped content. Also read the `name` and `description` frontmatter of
any newly added SKILL.md so you can describe it accurately in the metadata.

### 2. Reconcile `.claude-plugin/marketplace.json`

This registry lists **plugins**, not individual skills. Only touch it when a whole plugin
directory was added or removed under `plugins/`. For each plugin it must have an entry
with `name`, `description`, `source` (`./plugins/<name>`), and `category`. If a plugin's
purpose has broadened (e.g. it gained a skill in a new domain), refine its one-line
`description` so it still covers everything the plugin now does.

### 3. Reconcile the affected `plugin.json`

For the plugin that changed (`plugins/<name>/.claude-plugin/plugin.json`):

- **Version bump** — the reason this skill exists is to make version changes automatic and
  predictable:
  - **New skill or agent added** → bump the **minor** version (`0.1.0` → `0.2.0`).
  - **Existing skill/agent edited, or a skill/agent removed** → bump the **patch** version
    (`0.1.0` → `0.1.1`).
  - Never touch the major version automatically — a breaking change is a human judgment
    call. If you think one happened, flag it and ask.
- **`keywords`** — if the new content introduces a concept the keyword list doesn't cover,
  add a keyword. Don't pad it; keywords should stay a tight, honest list.
- **`description` / `interface` text** — if the plugin now does something materially new,
  update the `description`, `shortDescription`, and `longDescription` so they still
  describe the whole plugin. Small edits don't need this.

### 4. Reconcile `README.md`

The README has a plugins table:

```
| Plugin | Description | Skills |
|--------|-------------|--------|
```

The **Skills** column must list every skill in that plugin, comma-separated, in backticks.
This is the column most likely to be stale. Make it match step 1's ground truth exactly.
Also update the install command block if a whole new plugin was added.

### 5. Reconcile `CLAUDE.md`

CLAUDE.md documents the repo for future contributors. Update:

- The **structure tree** if a new plugin directory appeared.
- The **"Decide which plugin it belongs to"** table (Plugin | Purpose) if a plugin's
  purpose changed or a plugin was added.

Prose that's still accurate needs no edit — only fix what the change made wrong.

## Verify before you finish

After editing, prove the state is consistent rather than assuming it:

- Every plugin dir under `plugins/` has an entry in `marketplace.json`, and vice versa.
- Every skill on disk appears in the README Skills column for its plugin; no README skill
  is missing from disk.
- The changed `plugin.json` version moved by exactly the rule in step 3, and the JSON
  still parses (`python -m json.tool <file> >/dev/null`).

Then give the user a short summary: which files you touched, the old → new version, and
anything you flagged for their judgment (e.g. a suspected major bump). Do not commit — the
user or the git-ship skill handles git.

## Scope guardrails

- Don't invent skills, agents, or plugins that aren't on disk. The filesystem is the truth;
  metadata follows it, never the reverse.
- Don't reformat or rewrite files beyond what the change requires — minimal, surgical edits
  keep diffs reviewable.
- `*-workspace/` directories are never shipped content. Skip them everywhere.
