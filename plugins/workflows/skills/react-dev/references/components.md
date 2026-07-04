# React Component Conventions

## Component shape

- **Function components only.** Hooks replaced every class-component use case;
  new class components add a second paradigm for no benefit.
- One component per file, named export matching the filename:
  `LoginForm.tsx` exports `LoginForm`. Small private subcomponents may live in
  the same file below the main export.
- Props are an explicit interface, defined next to the component:

```tsx
interface LoginFormProps {
  onSubmit: (credentials: Credentials) => void;
  disabled?: boolean;
}

export function LoginForm({ onSubmit, disabled = false }: LoginFormProps) {
  // ...
}
```

- Destructure props in the signature; give optional props their default there,
  not with scattered `??` in the body.
- Return early for empty/loading/error states before the main JSX — keeps the
  happy-path render flat and readable.

## TypeScript

- No `any` in new code. Unknown data from the wire is `unknown`, narrowed with
  a type guard or schema validation.
- Type event handlers precisely (`React.ChangeEvent<HTMLInputElement>`), don't
  cast through `any`.
- Prefer union types over enums for simple variants:
  `type Status = "idle" | "loading" | "success" | "error"`.
- Derive types instead of duplicating them: `keyof`, `Pick`, `ReturnType`,
  `z.infer` — one source of truth per shape.

## Hooks and state

- State lives as low as possible; lift it only when two siblings actually need
  it. Global stores are for genuinely global data (session, theme), not a way
  to avoid prop drilling once.
- Derive, don't duplicate: if a value can be computed from existing state or
  props during render, compute it — don't mirror it into its own `useState`.
- `useEffect` is for synchronizing with *external* systems (network,
  subscriptions, DOM APIs). Transforming data for render or handling user
  events does not belong in an effect.
- Every effect that subscribes or schedules must return a cleanup function.
- Custom hooks (`useThing`) extract *stateful* reuse; plain functions extract
  stateless reuse. Don't wrap a pure function in a hook.
- Exhaustive dependencies: fix the code, don't silence the
  `react-hooks/exhaustive-deps` lint rule.

## React 19+ (check `react` version in package.json first)

Don't introduce React 19 APIs into an 18 codebase — but when the repo is on
19, prefer the modern forms:

- `ref` is a normal prop — no `forwardRef` wrapper in new components.
- `use(promise)` + `<Suspense>` for declarative async reads; `use(Context)`
  may be called conditionally, unlike `useContext`.
- Forms: `<form action={fn}>` + `useActionState` for submission state and
  errors; `useFormStatus` in child submit buttons; `useOptimistic` for
  immediate UI pending server confirmation.
- Server Components (framework-dependent, e.g. Next.js App Router):
  server-first by default — add `"use client"` only where interactivity
  requires it, as low in the tree as possible.

## Data fetching

- Fetch in the layer the app already uses (existing query library, route
  loader, or api module) — match the codebase, don't introduce a new pattern.
- Model the full request lifecycle: loading, error, and success all render
  something intentional. An unhandled error state is a blank screen bug.
- Abort or ignore stale responses when the component unmounts or inputs change
  (AbortController, or the query library's cancellation).
- Error boundaries around risky subtrees — an uncaught render error blanks
  the entire app. Pair each `<Suspense>` boundary with an error boundary so
  async failures degrade to a contained fallback, not a white screen.

## Styling and structure

- Use the styling approach the codebase already has (CSS modules, Tailwind,
  styled-components…). Never mix a second system into an existing app.
- Co-locate: component, its styles, and its test sit together
  (`LoginForm.tsx`, `LoginForm.module.css`, `LoginForm.test.tsx`).
- Accessibility is part of done: semantic elements over `div` soup, labels
  wired to inputs, interactive elements are buttons/links (not clickable
  divs), images have `alt`.

## Performance

- Check for React Compiler first (`babel-plugin-react-compiler` in the repo):
  if enabled, write plain code — the compiler memoizes per reactive scope,
  more granularly than hand-written hooks. Don't strip existing
  `useMemo`/`useCallback` in passing; removal is a dedicated cleanup, not a
  drive-by (double-memoization is harmless, regressions aren't).
- Without the compiler: don't memoize by default. `useMemo`/`useCallback`/
  `React.memo` are for *measured* problems or referential-stability
  requirements (dependency of an effect, prop to a memoized child) —
  speculative memoization is YAGNI.
- Keys are stable identities from the data, never array indices for lists that
  reorder.

## Before you hand off

- `tsc --noEmit` passes — the type checker is the first test suite.
- Loading, error, and empty states all render something intentional.
- New interactive elements keyboard-reachable and labeled for screen readers.
- No new `any`, no index keys on reorderable lists, no effect without cleanup,
  no `forwardRef` in a React 19 codebase.
