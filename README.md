# ai-toolkit-claude

AI skill plugins for Claude Code.

**Authors:** Ivan Melchenko ([@MelchenkoIvan](https://github.com/MelchenkoIvan)), Andrei Rabeshka

---

## Claude Code — install as marketplace

```bash
/plugin marketplace add MelchenkoIvan/ai-toolkit-claude
```

Then install individual plugins:

```bash
/plugin install workflows@ai-toolkit-claude
/plugin install presentation@ai-toolkit-claude
/plugin install ai-migrations@ai-toolkit-claude
```

---

## Plugins

| Plugin | Description | Skills |
|--------|-------------|--------|
| `workflows` | Developer workflow automation | `implement-task`, `solve-issue`, `copilot-claude-migrate` |
| `presentation` | Pitch and planning skills | `pitch-presentation`, `analyse-and-plan` |
| `ai-migrations` | Platform migration tools | `copilot-claude-migrate` |

## Agents

| Agent | Description |
|-------|-------------|
| `developer` | Implementation agent — writes code, creates PRs |
| `reviewer` | Code review agent — read-only, structured feedback |

---

## GitHub Copilot

For Copilot skills, see [ai-toolkit-copilot](https://github.com/MelchenkoIvan/ai-toolkit-copilot).

---

## Contributing

See [CLAUDE.md](CLAUDE.md) for how to add new skills and plugins.
