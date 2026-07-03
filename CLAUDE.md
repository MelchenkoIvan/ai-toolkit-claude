# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

Plugin marketplace for Claude Code. Skills live in `plugins/<plugin-name>/skills/`.

For Copilot skills, see [ai-toolkit-copilot](https://github.com/MelchenkoIvan/ai-toolkit-copilot).

## Repository structure

```
ai-toolkit-claude/
├── .claude-plugin/
│   └── marketplace.json          ← Claude Code marketplace registry
├── plugins/                      ← Claude Code plugins (one dir per plugin)
│   ├── workflows/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       ← plugin manifest
│   │   └── skills/<name>/SKILL.md
│   └── ai-migrations/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/<name>/SKILL.md
├── README.md
├── LICENSE
└── CLAUDE.md                     ← this file
```

## How to add a new skill

### Step 1 — Decide which plugin it belongs to

| Plugin | Purpose |
|--------|---------|
| `workflows` | Developer automation: create agents, ship git changes (branch/commit/push/PR) |
| `ai-migrations` | Platform migration tools |

Add new plugin dir under `plugins/` if none fits. See "Adding a new plugin" below.

### Step 2 — Add to Claude Code plugin surface

Create `plugins/<plugin-name>/skills/<skill-name>/SKILL.md` with frontmatter:

```markdown
---
name: skill-name
description: >
  One-sentence description used for triggering. Include: what it does,
  when it activates, slash command name if any.
---

<skill content here>
```

No other files needed unless skill has supporting scripts.

### Step 3 — Done

No changes to `marketplace.json` needed for adding skills to existing plugins.

---

## How to add a new plugin

1. Create `plugins/<plugin-name>/` directory
2. Create `plugins/<plugin-name>/.claude-plugin/plugin.json` — copy from existing plugin, update fields
3. Create `plugins/<plugin-name>/skills/` and `plugins/<plugin-name>/agents/` dirs
4. Add entry to `.claude-plugin/marketplace.json` under `"plugins"` array:

```json
{
  "name": "<plugin-name>",
  "description": "Short description.",
  "source": "./plugins/<plugin-name>",
  "category": "productivity"
}
```

---

## How to add an agent

Claude Code agents live in `plugins/<plugin-name>/agents/<name>.agent.md`.

---

## Skill content guidelines

- Skills here are **generalized** — no BiznestOrg-specific paths, repo names, or internal conventions
- Use placeholders like `{repo}`, `{issue_number}` instead of hardcoded values
- Skills should work for any GitHub repository, not just BiznestOrg
- Stub format for not-yet-added content: `<!-- Skill content coming soon. -->`

---

## Install commands (for reference)

```bash
# Claude Code — add this repo as marketplace
/plugin marketplace add MelchenkoIvan/ai-toolkit-claude

# Claude Code — install a plugin
/plugin install workflows@ai-toolkit-claude
/plugin install ai-migrations@ai-toolkit-claude
```

---

## Authors

Ivan Melchenko ([@MelchenkoIvan](https://github.com/MelchenkoIvan)), Andrei Rabeshka
