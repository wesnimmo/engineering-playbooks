# Environment Variables & Deployment Playbook

How secrets and config travel from your laptop to CI to the host — and the one security boundary that governs all of it. Provider-neutral; Supabase-specific notes are called out.

## The one security boundary: `NEXT_PUBLIC_`

In Next.js, the prefix is the whole game:

- **No prefix → server-only.** The value stays on the server and is never sent to the browser. This is the safe default.
- **`NEXT_PUBLIC_` → shipped to the browser.** It gets inlined into the client bundle at build time. Anyone can read it in DevTools.

Rules that follow from this:

- Default to **no prefix**. Only add `NEXT_PUBLIC_` when *every visitor* is allowed to read the value.
- **Never** prefix a key that bypasses your database security (a `service_role` / secret key). Prefixing it publishes your master key to the world.
- A value being public is not automatically a bug — a publishable/anon key is *designed* to be public because row-level security governs what it can do. The bug is prefixing a key that has no such guardrail.

## `.env.example` vs `.env.local`

Two files, opposite jobs:

| File | Committed? | Contents | Job |
| --- | --- | --- | --- |
| `.env.example` | **Yes** | Variable names with **blank** values | The "blank form" — documents *which* vars a dev must fill in |
| `.env.local` | **No** (gitignored) | The **real** values | The "filled form" — your machine's actual secrets |

- The `.gitignore` deliberately ignores `.env*.local` but allows `.env.example` through — that's why the blank template is the one thing committed.
- Values in `.env.local` are **literal strings** — no quotes, no `process.env.X` references, no trailing whitespace.
- Next.js reads env files **at startup only**, and `NEXT_PUBLIC_` values are inlined at build. **Restart the dev server after any env edit.**

## "Three computers don't share a folder"

Your laptop, your CI runner, and your host (Vercel/etc.) are three separate machines. **None of them can see another's `.env.local`.** Each gets its keys independently:

- **Laptop** → `.env.local`
- **CI** → its own secrets store (and often it runs on *mocks*, so it may need no real keys at all)
- **Host** → the platform's environment-variables settings, per environment (production / preview / development)

The trap this prevents: **green CI does not mean the host has your keys.** CI passing on mocked tests tells you nothing about whether production can reach the database. "Works locally, deploy is blank" is almost always a missing key on the *host*, not a code bug.

## Least privilege at the boundary

- A server-side function (e.g. a Server Action) still uses the **least-privileged key that works** — the publishable/anon key — so database security (RLS) governs it, not a master key.
- Only reach for a secret/`service_role` key when the task genuinely requires bypassing those guardrails, and keep it server-only.
- Don't add a secret to the host "just in case." Add it when code actually reads it.

## The ongoing convention (do this every time you add a var)

When you introduce a new env variable name in code:

1. Add it to `.env.example` (**blank**).
2. Add it to the **host**, filled in, in the right environment(s).
3. Do both **before the code merges**, so a deploy never lands referencing a var the host doesn't have.

## Fact vs. procedure — where env knowledge is documented

A reusable documentation heuristic:

- **A fact that's always true** (which vars exist, which are public, where values live) → belongs in `AGENT.md` / `CONTEXT.md`, because the agent should always know it.
- **A repeated procedure** (the exact click-path to add a var on the host's dashboard) → belongs in a **skill**, not `AGENT.md`. Procedures are steps you run on demand, not standing facts.

## Supabase specifics

- **Publishable / anon key** → public, safe for the browser → store in `NEXT_PUBLIC_SUPABASE_ANON_KEY`. RLS decides what it can do.
- **Secret / `service_role` key** → server-only, bypasses RLS → store as `SUPABASE_SERVICE_ROLE_KEY`, **never** with `NEXT_PUBLIC_`.
- **Project URL** → the bare origin (`https://<ref>.supabase.co`), no trailing slash and no `/rest/v1` — the client appends the path itself.
- Supabase renamed keys: dashboard says "Publishable / Secret," libraries still expect the `...ANON_KEY` / `...SERVICE_ROLE_KEY` variable *names*. Keep the old variable name, paste the new key value in.