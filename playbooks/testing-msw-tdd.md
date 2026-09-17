# Testing: MSW + TDD

## Why mock the network boundary

Unit tests should not hit the real database or network. Mocking with **MSW (Mock Service Worker)** makes tests:

- **Fast** — no network round-trip.
- **Deterministic** — same result every run (real DB data makes tests flaky).
- **Controllable** — mock an empty result, a full result, or an error on demand, to test every branch. You can't reliably make a real DB error mid-test.

Consequence: an **empty database is fine** for these tests. TDD even says write the failing test *before* the feature or any data exists. Tests assert against the *mocked* response, never live data.

## The MSW file structure (standard, reusable)

```
mocks/
  handlers.ts   ← the mocks themselves (environment-agnostic). Where the real work is.
  server.ts     ← Node adapter (setupServer) for Jest. Patches global fetch.
  browser.ts    ← Browser adapter (setupWorker) for in-app dev mocking. OPTIONAL — add later.
```

One `handlers.ts`, two thin adapters. For a TDD-of-a-server-action lesson you only need `handlers.ts` + `server.ts`.

## Mocking Supabase specifically

Supabase's REST layer (PostgREST) has its own URL shape:
- Requests go to `${SUPABASE_URL}/rest/v1/<table>`.
- Use a **wildcard** path (`*/rest/v1/genres`) so tests don't depend on which project/env is loaded — env-independent and won't break when the URL changes.
- Filters are **query params** (`?select=*`, `?id=eq.123`), not path segments.

## The global-fetch gotcha

MSW's Node interceptor only sees requests made through `globalThis.fetch`. If the Supabase client captured its own fetch reference, MSW never sees it. Fix: pass a **late-bound** fetch to the client so it resolves the global at call time (after MSW has patched it):

```ts
{ global: { fetch: (...args) => fetch(...args) } }
```

- `fetch: fetch` → binds the reference NOW (may freeze the real fetch before MSW patches it). ✗
- `fetch: (...args) => fetch(...args)` → resolves at call time → hits MSW's patched fetch. ✓

Apply this to whichever client the tests exercise. A **Server Action uses the SERVER client** — so test the server client, not just the browser one.

## The test-environment rule (match the runtime, per-file)

`testEnvironment` sets what globals exist. Match it to where the code-under-test actually runs:

- Keep the **global default as `jsdom`** (you'll want it for component tests).
- For **server-side code**, override per-file with a docblock — don't change global config:

```ts
/**
 * @jest-environment node
 */
```

Per-file override is the senior move: "test environment matches runtime" applies per *unit*, not globally. It also sidesteps jsdom-vs-Node fetch-primitive quirks in MSW v2.

## The two recipes (same handlers, different entry point)

| | Recipe A | Recipe B |
| --- | --- | --- |
| Unit under test | the Server Action, called directly | a component/form that triggers the action |
| Environment | `@jest-environment node` | jsdom (global default) |
| Assert | return value / thrown error / data shape | what the user sees (loading → rendered) |
| Extra tooling | none | @testing-library/react, user-event |
| Mock | shared `handlers.ts` | shared `handlers.ts` |

- **Recipe A = testing server-side logic** (Server Actions, route handlers) → node.
- **Recipe B = testing UI** (usually a Client Component) → jsdom.
- Note: a *Server Action* is not a *Server Component*. Recipe A tests the action function directly, no rendering.

## Commit rhythm for TDD

Commit in two steps: `test:` (the failing test) then `feat:` (the implementation that makes it pass). The history reads as red → green.