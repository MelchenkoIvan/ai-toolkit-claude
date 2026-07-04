---
name: react-dev
description: >
  React + TypeScript development conventions — components, hooks, state,
  styling, and Jest/React Testing Library testing. Use this skill whenever
  implementing, modifying, or testing frontend code in a React codebase: the
  task mentions React, TSX/JSX, components, hooks, or props, or the repo has a
  package.json with a react dependency. Load it even for small frontend
  tweaks — conventions apply to one-line changes too. Pipeline agents
  (developer, tester) load it as the React stack pack.
---

# React Development

Thin index for React/TypeScript work. Conventions live in `references/` —
read the file that matches your task:

| Task | Read |
|---|---|
| Writing or changing components, hooks, state, props, styling | `references/components.md` |
| Writing or changing tests (Jest + React Testing Library) | `references/testing.md` |

Implementing *and* testing? Read both. Always pair this skill with
`coding-principles` (universal rules — DRY/KISS/YAGNI, naming, error handling).

## Stack signals

You're in this skill's territory when:

- `package.json` lists `react` (with `typescript` → prefer `.tsx`).
- Files end in `.tsx` / `.jsx`, or imports reference `react`.
- The task mentions components, hooks, props, JSX, or frontend UI.

## Non-negotiables (summary — details in references)

- Function components + hooks only; no class components in new code.
- TypeScript-first: typed props, no `any` in new code.
- Tests query the DOM the way users do (roles/labels), not implementation
  details — see `references/testing.md`.
