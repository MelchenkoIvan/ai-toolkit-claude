---
name: dotnet-dev
description: >
  .NET / C# development conventions — project structure, async, dependency
  injection, LINQ, error handling, and xUnit/Moq testing. Use this skill
  whenever implementing, modifying, or testing backend code in a .NET
  codebase: the task mentions C#, .NET, ASP.NET, or Entity Framework, or the
  repo contains .csproj / .sln files. Load it even for small backend tweaks —
  conventions apply to one-line changes too. Pipeline agents (developer,
  tester) load it as the .NET stack pack.
---

# .NET Development

Thin index for .NET/C# work. Conventions live in `references/` — read the
file that matches your task:

| Task | Read |
|---|---|
| Writing or changing services, controllers, models, EF, DI, async code | `references/patterns.md` |
| Writing or changing tests (xUnit + Moq) | `references/testing.md` |

Implementing *and* testing? Read both. Always pair this skill with
`coding-principles` (universal rules — DRY/KISS/YAGNI, naming, error handling).

## Stack signals

You're in this skill's territory when:

- The repo contains `.csproj`, `.sln`, or `global.json` files.
- Files end in `.cs`, or the task mentions C#, .NET, ASP.NET Core, or
  Entity Framework.
- The task targets an API/backend in a solution laid out as `src/` + `tests/`.

## Non-negotiables (summary — details in references)

- Async all the way down — no `.Result` / `.Wait()` sync-over-async.
- Dependencies flow through constructor injection; no service locator.
- Tests are Arrange-Act-Assert with xUnit + Moq — see `references/testing.md`.
