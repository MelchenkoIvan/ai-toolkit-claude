---
name: run-unit-tests
description: >
  Run a codebase's unit tests for a change and drive them from red to green
  with proper test-first discipline — detect the runner, run the right tests,
  confirm a new test fails for the right reason before any implementation,
  implement the minimum to pass, and never weaken a test to force green. Use
  whenever the user says "run the unit tests", "run the tests for this change",
  "does this test fail yet", "make the tests pass", or when a developer or
  tester agent needs to execute tests inside a TDD loop. Routes to the stack's
  own testing conventions (react-dev / dotnet-dev references/testing.md) for
  the concrete runner command, test-scoping, and file naming.
---

# Run Unit Tests

Run the unit tests for a change and interpret the result with test-first
discipline. This skill owns **how to run tests and what red/green mean**; the
**framework specifics** (exact command, how to scope to one test, file naming)
come from the stack skill's `references/testing.md`, which this skill routes to
rather than repeating. That split keeps one source per concern: the runner
details live with the stack, the discipline lives here.

## When to use

- A developer or tester agent is in a TDD loop and needs to execute tests.
- The user asks to "run the unit tests", "run the tests for this change", or
  "make the tests pass".
- You wrote a test and need to confirm it fails *before* implementing (RED), or
  wrote code and need to confirm the target tests pass (GREEN).

**Not for:** writing the tests' content or picking assertions (that's the stack
skill's `references/testing.md` conventions) — this skill runs them and judges
red/green.

## Detect the runner — route to the stack skill

Never guess the command from memory. Detect the stack, then read its testing
reference for the exact runner:

1. `.tsx`/`.jsx` files or a `package.json` with `react` → load `react-dev`,
   read `references/testing.md` (Jest vs Vitest detection, `npm test`).
2. `.cs` files or `.csproj`/`.sln` → load `dotnet-dev`, read
   `references/testing.md` (xUnit/Moq, `dotnet test`).
3. Neither → detect the project's own runner from its manifest/scripts and use
   that; state which runner you found.

## Run the tests — scope to what changed

- Run the **target** tests (the ones covering the change) scoped as narrowly as
  the runner allows — a single file or filter — so red/green is about *this*
  change, not the whole suite.
- Run the **full** suite when the task is integration/merge verification (the
  orchestrator's post-merge step), not per-change iteration.
- Report the **exact command** and the **real** pass/fail counts. Never
  summarize a run you didn't execute.

## Red-green discipline (the core)

**RED — a failing test must fail for the right reason.**
- Run the new test *before* the implementation exists. It must fail on an
  **assertion/expectation** ("expected error, got none"), not on a
  compile/collection error ("cannot find name `LoginForm`", "module not
  found"). A compile error is not a valid RED — it means the test can't run
  yet. Add just enough scaffolding (the empty function/component, the export)
  so the test *runs and fails on its assertion*, then stop.
- A new test that **passes with no implementation** is a broken test — it
  asserts nothing real. Fix the test until absence of the feature makes it
  fail.

**GREEN — the minimum that passes, verified by real output.**
- Implement the smallest change that turns the target tests green. Do not add
  behavior no failing test demands (YAGNI).
- Re-run the target tests and confirm they pass from **actual** run output.
  "Should pass now" is not GREEN; a green run you saw is.

**REFACTOR — clean up under a green bar.**
- With tests green, improve names/structure, re-running the tests after each
  change to keep the bar green. Refactoring never changes behavior, so the same
  tests keep passing.

**Honesty (non-negotiable).**
- **Never weaken, skip, `xit`/`Skip`, or delete a test to make the suite pass**,
  and never lower a coverage threshold. A green suite bought by a gutted test
  is worse than a red one — it hides the failure.
- If the implementation is wrong, the test staying red is the *correct*
  outcome — report it; don't edit the test to match the bug.

## No runner in the repo

If the project has no test runner or scripts, say so explicitly and stop —
don't install a framework unprompted or invent pass/fail counts. Report "no
test runner detected" so the caller can decide (the orchestrator degrades to
build-verify + review).

## Reporting

Return the command(s) run, the true counts, and the red/green judgment:

```
Runner:  jest (npm test)               # or: vitest / dotnet test / none-detected
Scope:   src/auth/LoginForm.test.tsx   # the target tests, or "full suite"
Result:  RED — 3 fail (assertion), 0 pass    # or: GREEN — 3 pass, 0 fail
Notes:   fails on 'expected error, got none' (valid RED)
```
