---
name: create-agent
description: >
  Create a new Claude Code subagent — interviews for intent, then writes a
  complete `agents/<name>.agent.md` file with correct frontmatter (name,
  description, model, color, tools) and a structured system prompt following
  Anthropic's agent best practices. Use when the user says "create an agent",
  "add an agent", "write a subagent", "make a new agent", "scaffold an agent",
  or invokes `/create-agent`. Always use this skill whenever the user wants a
  new agent, even if they describe the behavior without using the word "agent".
---

# Create Agent

Turn a one-line request into a production-ready Claude Code subagent file. The output is a single Markdown file at `agents/<name>.agent.md` (or `agents/<name>.md`) with YAML frontmatter plus a second-person system prompt.

**What an agent is for:** autonomous, multi-step work that an orchestrating skill or the main thread delegates and walks away from. Agents are *not* user-facing slash commands — if the user wants something they trigger directly, they want a skill, not an agent. Surface this distinction if the request sounds like a command.

---

## Workflow

1. **Capture intent** — extract or ask for what the agent does, when it triggers, what it returns.
2. **Draft frontmatter** — name, description, model, color, tools.
3. **Write the system prompt** — role, responsibilities, process, output contract, edge cases, rules.
4. **Validate** — run the checklist.
5. **Save** — write to `agents/<name>.agent.md` and tell the user how to use it.

Move fast when the request is already clear. Only interview for the gaps.

---

## Phase 1 — Capture Intent

The conversation may already contain the answer (e.g., "make an agent that reviews my migrations"). Extract first, ask only for what's missing. Get clarity on:

1. **Purpose** — what task does this agent own, start to finish?
2. **Triggering** — what situations should dispatch it? Both proactive (an orchestrator hands off) and reactive (the user asks).
3. **Boundaries** — what must it NOT do? (e.g., "never opens PRs", "read-only", "never touches unrelated files")
4. **Input contract** — what does the agent receive when invoked? (a structured task, a file path, free text)
5. **Output contract** — what does it return to the caller?
6. **Tools** — does it need to write/run code, or just read and report?

If the request is vague, ask a short batch of questions rather than guessing. Don't proceed to Phase 2 until purpose, triggering, and boundaries are clear.

---

## Phase 2 — Frontmatter

```yaml
---
name: agent-identifier
description: "Use this agent when <conditions>. Typical triggers include <scenario 1>, <scenario 2>, and <scenario 3>. See 'When to invoke' in the agent body for worked scenarios."
model: inherit
color: blue
tools: ["Read", "Grep", "Glob"]
---
```

### name (required)
- lowercase letters, numbers, hyphens only; 3–50 chars; starts and ends alphanumeric.
- Specific, not generic. `migration-reviewer` ✅ — `helper` ❌.
- The file is named to match: `agents/<name>.agent.md`.

### description (required)
This is **the most important field** — it's what the harness reads to decide whether to dispatch the agent. Write it to be a little *pushy*, because agents tend to under-trigger.

Include:
1. Triggering conditions — "Use this agent when…".
2. A short prose summary of 2–4 trigger scenarios.
3. A pointer to the "When to invoke" body section.
4. When NOT to use it, if there's a tempting near-miss.

### model (required)
`inherit` (default — matches the parent session), or pin `opus` / `sonnet` / `haiku` when the task has a specific cost/capability profile. Heavy reasoning → `opus`; high-volume cheap work (search, triage) → `haiku`; balanced → `sonnet`.

### color (required)
One of `blue`, `cyan`, `green`, `yellow`, `magenta`, `red`. Pick something distinct from sibling agents in the same plugin. Loose convention: blue/cyan = analysis/review, green = build/success, yellow = validation, red = critical/security, magenta = generation/creative.

### tools (optional — but set it)
Grant the **minimum** the agent needs (least privilege). Omit the field only for genuinely general agents.

| Agent kind | Tools |
|---|---|
| Read-only analysis / review | `["Read", "Grep", "Glob"]` |
| Code generation | `["Read", "Write", "Edit", "Grep", "Glob"]` |
| Build / test runner | `["Read", "Bash", "Grep", "Glob"]` |
| Full-stack implementer | `["Read", "Write", "Edit", "Bash", "Grep", "Glob"]` |

Match the existing tool-naming in this repo's agents when in doubt.

---

## Phase 3 — System Prompt

The Markdown body becomes the agent's system prompt. Write in the **second person** ("You are…", "You will…"). Explain *why* things matter rather than stacking bare `MUST`s — the model has good theory of mind and follows reasoning better than edicts.

Use this skeleton (drop sections that don't apply):

```markdown
# <Agent Title>

You are <role> specializing in <domain>. You receive <input> and deliver <output>.

**Golden rule:** <the one principle that overrides everything — e.g., "Do exactly what the task describes; never touch unrelated code.">

## When to invoke

- **<Scenario name>.** <What the situation looks like and what you do.>
- **<Scenario name>.** <…>

## Input Contract

<What the agent receives, ideally as a table of fields.>

## Process

1. <Step — specific and actionable>
2. <Step>
3. <Step>

## Output Contract

<Exact structure the agent returns to the caller — a fenced template works well.>

## Edge Cases

- **<Situation>:** <how to handle it>

## Rules

1. **ALWAYS** <non-negotiable>.
2. **NEVER** <hard boundary>.
```

Guidance:
- **Input/Output contracts are the highest-value part.** An agent is only useful if the caller knows what to pass and what comes back. Make both concrete.
- **2–4 worked scenarios** under "When to invoke" — prose, not keyword lists.
- Keep the body focused (roughly under ~10k characters). Push deep reference material into sibling files the agent reads on demand if it gets large.
- Reserve all-caps `ALWAYS`/`NEVER` for genuine hard boundaries (safety, scope, irreversible actions). For everything else, explain the reasoning.

---

## Phase 4 — Validate

Before saving, check:

- [ ] `name`: 3–50 chars, lowercase + hyphens, starts/ends alphanumeric, matches the filename.
- [ ] `description`: states triggering conditions, names 2–4 scenarios, points to "When to invoke", and is a touch pushy.
- [ ] `model` and `color` present and valid.
- [ ] `tools`: least privilege (or deliberately omitted).
- [ ] System prompt: second person, has role + process + output contract.
- [ ] "When to invoke" section exists with 2–4 prose scenarios.
- [ ] Boundaries from Phase 1 appear in the Rules section.
- [ ] This is genuinely an agent (autonomous delegate), not a skill (user-triggered workflow).

---

## Phase 5 — Save & Hand Off

1. Determine the target plugin's `agents/` directory (default: the plugin the user named, else ask). In this repo agents live at `plugins/<plugin-name>/agents/`.
2. Write the file as `agents/<name>.agent.md`.
3. Confirm the plugin manifest exposes agents (`"agents": "./agents/"` in `plugin.json`) — add it if missing.
4. Report:

```
🤖 Agent created
━━━━━━━━━━━━━━━━
Name:  <name>
File:  plugins/<plugin>/agents/<name>.agent.md
Model: <model>   Tools: <tools>

Triggers on: <one-line summary>
Returns:     <output contract summary>
```

Mention the agent is auto-discovered once the plugin is installed, and that an orchestrating skill or the main thread can dispatch it via the `Agent` tool with `subagent_type: "<name>"`.

---

## Worked Example

**Request:** "Make an agent that reviews database migrations for safety before they ship."

**Result — `agents/migration-reviewer.agent.md`:**

```markdown
---
name: migration-reviewer
description: "Use this agent when a database migration needs a safety review before merge. Typical triggers include a new migration file added to a PR, a developer asking 'is this migration safe?', and an orchestrating skill handing off generated schema changes. See 'When to invoke' in the agent body. Do NOT use it to write migrations — only to review them."
model: sonnet
color: yellow
tools: ["Read", "Grep", "Glob", "Bash"]
---

# Migration Reviewer

You are a database reliability engineer. You receive one or more migration files and deliver a safety verdict with specific, actionable findings. You never modify the migration — you review it.

**Golden rule:** Flag anything that risks data loss, locking, or downtime, with the exact line and a safer alternative.

## When to invoke

- **Pre-merge review.** A migration file appears in a diff; assess it before it ships.
- **Direct question.** A developer asks whether a specific migration is safe.

## Input Contract

| Field | Description |
|-------|-------------|
| Migration file(s) | Path(s) to the migration to review |
| Target engine | Postgres / MySQL / etc. |

## Process

1. Read each migration and the schema it touches.
2. Check for destructive ops (DROP, NOT NULL without default, type narrowing).
3. Check for locking risks on large tables (index builds, rewrites).
4. Verify reversibility / down-migration.

## Output Contract

A verdict (✅ safe / ⚠️ risky / ❌ unsafe) followed by a findings table: severity, file:line, issue, suggested fix.

## Rules

1. **NEVER** edit the migration — review only.
2. **ALWAYS** cite file:line for each finding.
```

---

## Rules

1. **ALWAYS** confirm purpose, triggering, and boundaries before drafting.
2. **ALWAYS** write the description to trigger reliably — name concrete scenarios, lean slightly pushy.
3. **ALWAYS** define an explicit input and output contract in the system prompt.
4. **ALWAYS** apply least-privilege tools.
5. **NEVER** write the body in first person.
6. **NEVER** produce an agent when the user actually wants a skill (user-triggered) — say so and redirect.
7. Explain the *why* behind instructions instead of stacking bare `MUST`s.
