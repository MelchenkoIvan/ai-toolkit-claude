---
name: migrate-skill
description: >
  Migrates skills bidirectionally between GitHub Copilot and Claude Code formats.
  Copilot skills live in .github/skills/NAME/SKILL.md; Claude skills live in
  ~/.claude/skills/NAME/SKILL.md (global) or .claude/skills/NAME/SKILL.md
  (project-level). Migrates connected instruction files and agent definitions too.
  Adapts agent delegation, tool references, model selection, and descriptions for
  the target platform. Both source and target versions always coexist — nothing is
  deleted. Use this skill whenever the user says: "migrate skill to Claude",
  "port skill to Copilot", "convert this skill for Claude Code", "make this Copilot
  skill work in Claude", "migrate skill from Claude", "/migrate-skill", or asks to
  move/copy/convert any skill between the two platforms.
---

# Migrate Skill

Bidirectional skill migration between GitHub Copilot (`.github/skills/`) and Claude Code
(`~/.claude/skills/` or `.claude/skills/`). Migrates the skill and all connected
dependencies (instruction files, agent definitions). Source is always preserved.

## Dependency locations

| Asset | Copilot | Claude Code |
|---|---|---|
| Skills | `.github/skills/<name>/` | `.claude/skills/<name>/` or `~/.claude/skills/<name>/` |
| Instruction files | `.github/instructions/*.instructions.md` | `.claude/instructions/*.md` |
| Agent definitions | `.github/agents/*.agent.md` | `.claude/agents/*.md` |

Claude Code has no auto-loaded instruction files (unlike CLAUDE.md). Migrated instruction
files land in `.claude/instructions/` and skills explicitly `Read` them when needed —
same pattern as Copilot skills reference `.github/instructions/`.

Agent definitions in `.claude/agents/` store the system prompt. When a Claude skill
delegates via the `Agent` tool, it reads the agent file and passes the system prompt
as context in the `prompt` parameter.

---

## Inputs

Collect from the user's message:

| Parameter | Required | Example |
|---|---|---|
| **Skill name** | Yes | `solve-issue`, `branch-cleanup` |
| **Direction** | Yes | `copilot→claude` or `claude→copilot` |
| **Scope** (copilot→claude only) | No | `global` or `project` — auto-detected if omitted |

Ask before proceeding if skill name or direction is missing.

---

## Direction: Copilot → Claude

### Step 1 — Locate source

Source root: `{workflows_local}/.github/skills/<name>/`

Read `SKILL.md`. List all other files in the directory.
If `SKILL.md` not found, stop and report.

### Step 2 — Scan for dependencies

Scan the source SKILL.md for references to:
- `.github/instructions/` — collect every referenced `.instructions.md` filename
- `.github/agents/` — collect every referenced `.agent.md` filename

For each referenced file, check if it exists in `{workflows_local}`. Collect the
full list before proceeding — you'll migrate them in Step 4.

### Step 3 — Determine target scope

If the user specified `global` or `project`, use that.

Otherwise, auto-detect from source content:

**Project-level** (`.claude/skills/<name>/` inside this repo) if the skill body references
any of: `{workflows_local}`, `{repos_root}`, `{worktrees_root}`, BiznestOrg repo names
(`Shops.API`, `Users.API`, `EasyManagementMobile`, etc.), `biznest-developer`,
`biznest-reviewer`, or `.github/instructions/`.

**Global** (`~/.claude/skills/<name>/`) if the skill is a general-purpose utility
with no BiznestOrg-specific references.

Tell the user the detected scope and dependency list. Let them override before writing.

### Step 4 — Adapt the skill for Claude Code

Read the full source SKILL.md and produce an adapted version.

#### Frontmatter

- **`name`**: keep unchanged.
- **`description`**: Claude uses this for auto-triggering. Enhance it:
  - Add trigger phrases: "Use when user says...", "Auto-triggers when..."
  - Be assertive: "Always use this skill whenever..." rather than "Use for..."
  - Preserve the skill's purpose; add triggering vocabulary around it.
- **`compatibility`**: add only if the skill requires specific tools. Usually omit.

#### Agent delegation and model selection

Copilot delegates via named agents (`task` tool). Claude uses the `Agent` tool.

**Model selection guide** — choose based on task complexity:

| Task complexity | Model |
|---|---|
| Default / general implementation | `model: "sonnet"` (Sonnet 4.6) |
| Simple, low-reasoning tasks (file reads, formatting, grep) | `model: "haiku"` (Haiku 4.5) |
| Complex orchestration, multi-agent coordination, deep reasoning | `model: "opus"` (Opus 4.7) |

**Agent mapping:**

| Copilot pattern | Claude equivalent |
|---|---|
| `task` → `biznest-developer` (`claude-sonnet-4.6`) | `Agent`, `subagent_type: "general-purpose"`, `model: "sonnet"` |
| `task` → `biznest-reviewer` (`gpt-5.3-codex`) | `Agent`, `subagent_type: "caveman:cavecrew-reviewer"`, `model: "sonnet"` |
| `explore` sub-agents | `Agent`, `subagent_type: "Explore"`, `model: "haiku"` |
| Any Copilot agent using a model you judge as "orchestrator / complex reasoning" | `model: "opus"` |

**If the Copilot agent has a `.github/agents/<name>.agent.md` definition:**
After migrating the file to `.claude/agents/<name>.md` (Step 5), update the Agent call
to read and pass that system prompt:

```
Read `.claude/agents/<name>.md` first, then pass its content as `system_prompt` in the
Agent tool call.
```

When rewriting delegation instructions:
- Preserve the full context-package format.
- Preserve parallelism rules: "parallel" → spawn multiple `Agent` calls in the same turn.
- Preserve dependency ordering between agent groups.
- Never strip detail from delegated prompts.

#### Tool references in prose

| Copilot implies | Claude equivalent |
|---|---|
| Reading a file | `Read` tool |
| Writing / editing a file | `Edit` or `Write` tool |
| Running a shell command | `Bash` tool |
| Searching the codebase | `Bash` with grep, or `Agent` `subagent_type: "Explore"` |
| Sub-agent exploration | `Agent` tool, `subagent_type: "Explore"` |

Only annotate where the action is genuinely ambiguous or instructional.

#### Instruction file references

Replace every `.github/instructions/<file>.instructions.md` reference with
`.claude/instructions/<file>.md` (drop the `.instructions` suffix). The skill should
explicitly `Read` each instruction file it needs — same pattern as before, just new path.

#### Commit trailers

```
Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
```
→
```
Co-Authored-By: Claude <noreply@anthropic.com>
```

#### Path placeholders

Keep `{repos_root}`, `{worktrees_root}`, `{workflows_local}` unchanged.

#### BiznestOrg ecosystem content

Keep all BiznestOrg-specific knowledge unchanged.

### Step 5 — Migrate dependencies

**Instruction files:** For each `.github/instructions/<file>.instructions.md` found in Step 2:
- Target: `{workflows_local}/.claude/instructions/<file>.md`
- Copy content as-is (instructions are platform-agnostic prose).
- Create the directory if it doesn't exist.

**Agent files:** For each `.github/agents/<name>.agent.md` found in Step 2:
- Target: `{workflows_local}/.claude/agents/<name>.md`
- Copy content as-is.
- Create the directory if it doesn't exist.
- Note: in Claude Code the agent file is the system prompt; the calling skill reads it
  and passes it to the `Agent` tool. No structural changes to the file content are needed.

### Step 6 — Copy skill's supporting files

Copy all non-SKILL.md files from `.github/skills/<name>/` to the target skill directory.
Scripts, references, and assets are platform-agnostic.

### Step 7 — Write target skill

Create target directory if needed, write adapted SKILL.md and all copied supporting files.

**Project-level:** `{workflows_local}/.claude/skills/<name>/`
**Global:** `~/.claude/skills/<name>/`

### Step 8 — Report

```
✅ Skill migrated: Copilot → Claude

Source:  {workflows_local}/.github/skills/<name>/SKILL.md
Target:  <target-path>/SKILL.md
Scope:   <global | project-level>

Adaptations made:
- Description enhanced with Claude auto-trigger phrases
- <agent delegation mappings applied, with model selections>
- <tool references added>
- Commit trailer updated to Claude format
- <N> supporting files copied

Dependencies migrated:
- Instructions: <list of .github/instructions/ → .claude/instructions/ files, or "none">
- Agents: <list of .github/agents/ → .claude/agents/ files, or "none">

⚠️  Manual review recommended:
- <patterns that couldn't be cleanly translated>
- <Copilot-specific behavior with no Claude equivalent>
```

---

## Direction: Claude → Copilot

### Step 1 — Locate source

Check both locations (in order):
1. Global: `~/.claude/skills/<name>/SKILL.md`
2. Project-level: `{workflows_local}/.claude/skills/<name>/SKILL.md`

If found in both, ask which to use. If neither, stop and report.

### Step 2 — Scan for dependencies

Scan the source SKILL.md for references to:
- `.claude/instructions/` — collect every referenced filename
- `.claude/agents/` — collect every referenced filename

Check if each file exists. Collect before proceeding.

### Step 3 — Check target

Target: `{workflows_local}/.github/skills/<name>/SKILL.md`

If already exists, warn and ask:
- **Overwrite**
- **Rename** — create as `<name>-from-claude`
- **Cancel**

### Step 4 — Adapt the skill for Copilot

#### Frontmatter

- **`name`**: keep unchanged.
- **`description`**: Copilot shows this as a tooltip — not used for auto-triggering.
  Write a **1-2 sentence description only** (aim for ≤150 chars). Remove all trigger
  phrases ("Use when user says...", "Auto-triggers when...", "Always use this skill
  whenever..."). Keep only the core "what it does" statement.

  Example — Claude description:
  > "Migrates skills between GitHub Copilot and Claude Code formats. Use this skill
  > whenever the user says 'migrate skill', 'port to Copilot', '/migrate-skill'..."

  Copilot description (correct):
  > "Migrate skills bidirectionally between GitHub Copilot and Claude Code formats."

- **`compatibility`**: remove (not used by Copilot).

#### Agent delegation and model mapping

| Claude pattern | Copilot equivalent |
|---|---|
| `Agent`, `subagent_type: "general-purpose"`, `model: "sonnet"` | `task` → `biznest-developer`, `model="claude-sonnet-4-6"` |
| `Agent`, `subagent_type: "general-purpose"`, `model: "opus"` | `task` → `biznest-developer`, `model="claude-opus-4-7"` |
| `Agent`, `subagent_type: "caveman:cavecrew-reviewer"` | `task` → `biznest-reviewer`, `model="gpt-5.3-codex"` |
| `Agent`, `subagent_type: "Explore"` | `explore` sub-agents |

Preserve context-package content. Preserve parallelism instructions.

If the skill reads `.claude/agents/<name>.md` before calling `Agent` — the agent file
already maps to `.github/agents/<name>.agent.md` (handled in Step 5). Update the
reference path in the adapted skill.

#### Tool references in prose

Copilot agents infer tool use — explicit references are unnecessary. Naturalize:

| Claude | Copilot |
|---|---|
| "Use the `Read` tool to read..." | "Read..." |
| "Use the `Bash` tool to run..." | "Run..." |
| "Use the `Edit` tool to update..." | "Update..." |
| "Use the `Agent` tool with `subagent_type: 'Explore'`..." | "Use `explore` sub-agents to..." |

Keep shell command blocks verbatim.

#### Instruction file references

Replace `.claude/instructions/<file>.md` with `.github/instructions/<file>.instructions.md`
(add the `.instructions` suffix back).

#### Claude Code-specific features

| Claude Code feature | Action |
|---|---|
| Memory system (`~/.claude/projects/...`) | Remove — no Copilot equivalent |
| `Skill` tool invocations | Convert to `invoke /skill-name` |
| `/caveman`, `/caveman-commit` etc. | Remove; flag for review |
| `ScheduleWakeup`, `CronCreate` | Remove; flag for review |

#### Commit trailers

```
Co-Authored-By: Claude <noreply@anthropic.com>
```
→
```
Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
```

#### Path placeholders

Keep all placeholders unchanged.

### Step 5 — Migrate dependencies

**Instruction files:** For each `.claude/instructions/<file>.md` found in Step 2:
- Target: `{workflows_local}/.github/instructions/<file>.instructions.md`
- Copy content as-is.

**Agent files:** For each `.claude/agents/<name>.md` found in Step 2:
- Target: `{workflows_local}/.github/agents/<name>.agent.md`
- Copy content as-is (the system prompt becomes the agent definition).

### Step 6 — Copy skill's supporting files

Copy all non-SKILL.md files from the source skill directory to `.github/skills/<name>/`.

### Step 7 — Write target skill

Create `{workflows_local}/.github/skills/<name>/` if needed, write adapted SKILL.md and
all copied files.

### Step 8 — Report

```
✅ Skill migrated: Claude → Copilot

Source:  <source-path>/SKILL.md
Target:  {workflows_local}/.github/skills/<name>/SKILL.md

Adaptations made:
- Description simplified to 1-2 sentence tooltip
- <agent delegation mappings applied>
- <tool references naturalized>
- Commit trailer updated to Copilot format
- <N> supporting files copied

Dependencies migrated:
- Instructions: <list of .claude/instructions/ → .github/instructions/ files, or "none">
- Agents: <list of .claude/agents/ → .github/agents/ files, or "none">

⚠️  Manual review recommended:
- <Claude-only features removed>
- <agent behavior differences>
```

---

## Rules

1. **Never delete the source.** Both versions always coexist after migration.
2. **Preserve path placeholders.** `{repos_root}`, `{worktrees_root}`, `{workflows_local}` unchanged in both directions.
3. **Preserve agent context packages.** Full prompt detail must survive — only wrapping syntax changes.
4. **Migrate dependencies.** Always check for and migrate referenced instruction and agent files. Never migrate the skill alone if it has dependencies.
5. **Flag what you can't cleanly translate.** List it in the manual review section.
6. **Ask before writing** if scope is ambiguous or target already exists.
7. **One skill per invocation.** Multiple skills → handle sequentially.
