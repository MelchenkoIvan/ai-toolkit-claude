# ai-toolkit

AI skill plugins for Claude Code and GitHub Copilot.

**Authors:** Ivan Melchenko ([@MelchenkoIvan](https://github.com/MelchenkoIvan)), Andrei Rabeshka

---

## Claude Code — install as marketplace

```bash
/plugin marketplace add MelchenkoIvan/ai-toolkit
```

Then install individual plugins:

```bash
/plugin install workflows@ai-toolkit
/plugin install presentation@ai-toolkit
```

## GitHub Copilot — install individual skills

```bash
gh skill install MelchenkoIvan/ai-toolkit/skills/workflows
gh skill install MelchenkoIvan/ai-toolkit/skills/presentation
```

---

## Plugins

| Plugin | Description | Skills |
|--------|-------------|--------|
| `workflows` | Developer workflow automation | `implement-task`, `solve-issue`, `migrate-skill` |
| `presentation` | Pitch and planning skills | `pitch-presentation`, `analyse-and-plan` |

## Agents

| Agent | Description |
|-------|-------------|
| `developer` | Implementation agent — writes code, creates PRs |
| `reviewer` | Code review agent — read-only, structured feedback |

---

## Contributing

See [CLAUDE.md](CLAUDE.md) for how to add new skills and plugins.
