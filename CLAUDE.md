# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

Plugin marketplace for Claude Code and skill distribution hub for GitHub Copilot.
Skills live in two surfaces — keep both in sync when adding content.

## Repository structure

```
ai-toolkit/
├── .claude-plugin/
│   └── marketplace.json          ← Claude Code marketplace registry
├── plugins/                      ← Claude Code plugins (one dir per plugin)
│   ├── workflows/
│   │   ├── .codex-plugin/
│   │   │   └── plugin.json       ← plugin manifest
│   │   ├── skills/<name>/SKILL.md
│   │   └── agents/<name>.agent.md
│   └── presentation/
│       ├── .codex-plugin/
│       │   └── plugin.json
│       └── skills/<name>/SKILL.md
├── skills/                       ← Copilot skills (gh skill install target)
│   └── <name>/SKILL.md
├── agents/                       ← Copilot agents
│   └── <name>.agent.md
├── README.md
├── LICENSE
└── CLAUDE.md                     ← this file
```

## How to add a new skill

### Step 1 — Decide which plugin it belongs to

| Plugin | Purpose |
|--------|---------|
| `workflows` | Developer automation: implement, solve, migrate |
| `presentation` | Pitch decks, planning, analysis |

Add new plugin dir under `plugins/` if neither fits. See "Adding a new plugin" below.

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

### Step 3 — Add to Copilot skill surface

Create identical `skills/<skill-name>/SKILL.md` (same content, same frontmatter).
This is the install target for `gh skill install MelchenkoIvan/ai-toolkit/skills/<skill-name>`.

### Step 4 — Done

No changes to `marketplace.json` needed for adding skills to existing plugins.

---

## How to add a new plugin

1. Create `plugins/<plugin-name>/` directory
2. Create `plugins/<plugin-name>/.codex-plugin/plugin.json` — copy from existing plugin, update fields
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

Two surfaces:

- **Claude Code (within plugin):** `plugins/<plugin-name>/agents/<name>.agent.md`
- **Copilot (root-level):** `agents/<name>.agent.md`

Keep both in sync.

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
/plugin marketplace add MelchenkoIvan/ai-toolkit

# Claude Code — install a plugin
/plugin install workflows@ai-toolkit
/plugin install presentation@ai-toolkit
/plugin install ai-migrations@ai-toolkit

# Copilot — install individual skill
gh skill install MelchenkoIvan/ai-toolkit/skills/implement-task
gh skill install MelchenkoIvan/ai-toolkit/skills/copilot-claude-migrate
```

---

## Authors

Ivan Melchenko ([@MelchenkoIvan](https://github.com/MelchenkoIvan)), Andrey Robeshko
