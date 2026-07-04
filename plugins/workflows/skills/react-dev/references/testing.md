# React Testing Conventions — Jest/Vitest + React Testing Library

## Detect the runner first

Read `package.json` before writing a single test. `vitest` → `vi.fn()`,
`vi.mock()`, `vi.useFakeTimers()`; `jest` → the `jest.*` equivalents. The
Testing Library API is identical under both — only the mock/timer namespace
differs. Introducing the other runner's globals produces a suite that won't
run. Examples below use `jest.*`; substitute `vi.*` one-for-one under Vitest.

## Files and naming

- Test file co-located with the component: `LoginForm.test.tsx` next to
  `LoginForm.tsx`.
- `describe` block per component/hook; test names state observable behavior:
  `it("shows a validation error when email is empty")`, not
  `it("works")` or `it("calls setState")`.

## The core rule: test what the user sees

React Testing Library's whole philosophy — a test should break when *behavior*
breaks, not when implementation is refactored.

- Query by role and accessible name first:
  `screen.getByRole("button", { name: /sign in/i })`. Fall back through
  `getByLabelText` → `getByPlaceholderText` → `getByText`. `getByTestId` is
  the last resort, for elements with no accessible identity.
- Never assert on component internals: no state inspection, no instance
  methods, no "was this hook called". If the DOM is right, the internals were
  right.
- Interact through `userEvent` (real event sequences: focus, keydown, click),
  not bare `fireEvent`:

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

it("submits entered credentials", async () => {
  const onSubmit = jest.fn();
  render(<LoginForm onSubmit={onSubmit} />);

  await userEvent.type(screen.getByLabelText(/email/i), "a@b.com");
  await userEvent.type(screen.getByLabelText(/password/i), "hunter2");
  await userEvent.click(screen.getByRole("button", { name: /sign in/i }));

  expect(onSubmit).toHaveBeenCalledWith({ email: "a@b.com", password: "hunter2" });
});
```

## Async

- `findBy*` queries (they wait) for elements that appear after async work;
  `waitFor` for non-query assertions.
- Never assert absence with `getBy*` (it throws) — use `queryBy*`:
  `expect(screen.queryByText(/error/i)).not.toBeInTheDocument()`.
- No arbitrary `setTimeout`/sleep in tests — wait for the observable result.

## Mocking

- Mock at the network boundary, not the module boundary, when feasible (MSW
  or a fetch mock) — tests then survive refactors of the api module.
- `jest.mock` for genuinely external modules (analytics, router when not under
  test). Don't mock the component tree under test — render the real thing.
- Every mock is restored: `afterEach(() => jest.restoreAllMocks())` or
  `restoreMocks: true` in config.
- Callback props are `jest.fn()` and asserted with exact expected arguments.

## Providers

Components needing context (router, query client, theme) get one custom
render wrapper in a shared `test-utils` file — not a provider stack
copy-pasted into every test:

```tsx
// test-utils.tsx
export function renderWithProviders(ui: React.ReactElement) {
  const queryClient = new QueryClient({ defaultOptions: { queries: { retry: false } } });
  return render(
    <QueryClientProvider client={queryClient}>{ui}</QueryClientProvider>
  );
}
```

If the repo already has such a wrapper, use it — a second one drifts.

## Hooks

- Test custom hooks through a component that uses them when practical; use
  `renderHook` (from `@testing-library/react`) for hooks with no natural
  component.

## What to cover

Priority order for a component's suite:

1. The primary user path (render → interact → observable result).
2. Validation / error states the user can trigger.
3. Loading and failure states of async operations.
4. Edge cases in props (empty lists, missing optionals).

Don't chase 100% line coverage through implementation-detail tests — a suite
that breaks on every refactor is worse than a smaller behavioral one.

## Anti-patterns that fail review

**False confidence (critical):**

- A test with no assertion — renders, maybe clicks, verifies nothing.
  Coverage goes up, correctness is unchecked.
- Missing `await` on `userEvent` calls or async assertions — the test passes
  before the behavior runs.
- Snapshot-everything tests nobody reads — a failing snapshot that gets
  blindly updated asserts nothing.

**Flakiness:**

- Real timers or sleeps — use fake timers or `waitFor`/`findBy*`.
- Tests that depend on execution order or leak state — each test renders
  fresh and cleans its mocks.

**Assertion quality:**

- `toBeInTheDocument()` alone is trivial — also assert the *content* the
  user cares about (values, text, disabled state).
- Assert what should *not* happen too: after a fix, the error message is
  gone (`queryBy* → not.toBeInTheDocument()`); the callback was *not* called
  on invalid submit.
