---
name: coding-principles
user-invocable: false
description: >
  Universal, stack-agnostic coding principles — DRY, KISS, YAGNI, naming,
  error handling, and scope discipline — that apply to every implementation
  task regardless of language or framework. Use this skill whenever writing,
  modifying, or reviewing production code in any stack: before implementing a
  feature, fixing a bug, or refactoring. Pipeline agents (developer, tester,
  reviewer) load it as their baseline rulebook; it is equally useful in plain
  chat when the user asks to "write clean code", "refactor this", or "review
  code quality".
---

# Coding Principles

Universal rules that apply to every stack. Stack-specific conventions live in
their own skills (`react-dev`, `dotnet-dev`) — this skill is what remains true
everywhere.

## DRY — Don't Repeat Yourself

- Extract shared logic once a pattern appears a **third** time (rule of three).
  Two occurrences may be coincidence; premature abstraction costs more than
  duplication.
- Duplication of *knowledge* is the real enemy — two functions that look alike
  but encode different business rules should stay separate.
- Prefer extending an existing helper over writing a parallel one. Search the
  codebase before adding a utility.

## KISS — Keep It Simple

- Choose the simplest design that solves the *stated* problem. Cleverness is a
  maintenance tax paid by whoever reads the code next.
- Flat is better than nested: early returns over deep `if` trees, guard clauses
  over arrow-shaped code.
- If a function needs a comment to explain *what* it does, split or rename it.
  Comments explain *why*, never *what*.

## YAGNI — You Aren't Gonna Need It

- Implement what the task asks for — not the configurable, pluggable version of
  it. Speculative generality is deleted code that hasn't happened yet.
- No new dependencies, layers, or abstractions "for later". Add them when a
  concrete requirement arrives.
- Feature flags, options, and parameters that have exactly one caller and one
  value are YAGNI violations.

## Naming

- Names state intent: `remainingRetries`, not `n`; `isEligibleForDiscount`,
  not `check2`.
- Match the vocabulary of the surrounding code and domain — don't introduce a
  synonym for a concept the codebase already names.
- Booleans read as predicates (`is`/`has`/`can`), functions as verbs, values as
  nouns. Avoid negated booleans (`isNotReady`) — double negatives hurt.
- Length proportional to scope: loop indices can be short; module-level exports
  deserve full words.

## Error handling

- Handle errors at the level that can *do something* about them — don't catch
  just to log-and-rethrow at every layer.
- Never swallow errors silently. An empty catch block is a bug you scheduled
  for later.
- Fail fast on programmer errors (invalid arguments, broken invariants);
  recover gracefully from environmental ones (network, IO, user input).
- Error messages carry context: what was attempted, with what input, and what
  the caller can do about it.

## Scope discipline

- Do exactly what the task describes. Unrelated cleanup — even obvious
  improvements — belongs in its own change, not smuggled into this one.
- Match the style, idioms, and structure of the file you're editing. A diff
  should look like the original author wrote it.
- Detect before you add: check what the codebase already uses (test runner,
  API style, styling system, mocking library) and use that. Never introduce
  a second paradigm alongside an incumbent — mixed paradigms cost every
  future reader.
- Leave the code no worse: don't break existing tests, don't remove behavior
  the task didn't ask you to remove.

## Verification

- Code isn't done until it's been exercised: build it, run existing tests, and
  observe the changed behavior where feasible.
- Report verification honestly — "build clean, tests pass" only if you ran
  them. Never claim coverage you didn't create or checks you didn't run.
- A test without a meaningful assertion is worse than no test — it
  manufactures false confidence. Coverage measures execution, not
  correctness.
