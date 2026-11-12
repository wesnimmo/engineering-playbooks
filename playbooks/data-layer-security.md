# Data-Layer Security Playbook

How to protect data so the protection can't be bypassed. The principles hold on any stack. Postgres/Supabase mechanics are in their own sections at the end.

## Sort every idea into three layers

| Layer | What it is | Transfers? | Example |
| --- | --- | --- | --- |
| **Principle** | A design idea true on any stack | Always | Deny by default |
| **Mechanism** | How a technology does it | When you use that tech | Postgres `GRANT`/`REVOKE`, RLS |
| **Project detail** | Specific to one app | No | A particular view's column list |

Playbooks hold principles and some mechanisms. Project details stay in the project.

## Principles

### 1. Enforce security at the lowest layer that can enforce it

App code can be bypassed. Database rules can't. Trimming `.select("*")` to a short column list in TypeScript is good manners, not security: anyone with the public key can call the API directly and ask for every column. The database decides what can be read.

### 2. Deny by default, then grant only what's needed

Start with no access and add the narrowest permission that makes the feature work. A missing grant shows up quickly as an empty result. An extra grant shows up much later as a leak.

### 3. Publish a contract, not your storage

Give the public a narrow interface (a view, an API response type) that shows exactly the fields it needs and hides the rest. Example: a search card shows derived **age** and never exposes **date of birth**. The raw table stays private. This is the same idea as a component that only receives the props it needs.

### 4. Put each protection where it is actually enforced

Write down *which layer* guards *what*. For example: "the view protects the birthdate; the login gate protects full profiles and messaging." Vague reasons ("hidden because age is sensitive") lead people and agents to add protections in the wrong place or remove features that were intentional.

### 5. Least privilege for code too

Server-side code still uses the least-privileged key that works, so database rules govern it. Reach for an admin key only when a task truly must bypass those rules, and keep it server-only.

### 6. "App vs. owner" is about the database role, not where code runs

- **The app** = anything that reaches the database with the public key: Client Components **and** Server Components, Server Actions, route handlers. It runs as `anon` / `authenticated` and is limited by grants and RLS.
- **The owner** = the database acting on its own authority: migrations, the SQL Editor as `postgres`, triggers, `security definer` functions and views. Not limited by grants or RLS.

Server-side does not mean trusted. Only an admin key (`service_role`) moves server code to the owner side, which is why it's forbidden for user-facing work. Browser vs. server is about **secrets**; app vs. owner is about **data access**.

### 7. A login is not a profile

Identity (the auth system's users table) and the profile (your own table) are separate. One foreign key links them. The profile is created by the owner (e.g. a trigger on signup), so the app never needs permission to insert profiles.

### 8. Green tests don't prove security

Mocked tests (MSW) check that your code calls the right endpoint with the right query. They never touch the real database, so they can't check permissions or policies. Verify security against the real database.

### 9. Verify by impersonating, and test the denied cases

Don't verify security by reading code. Run the query **as** each role and check the result:

- The owner of a row sees it.
- A **different** signed-in user sees nothing.
- The public is **denied**.

One passing happy path proves little. The negative cases are the ones that prove the rule works.

### 10. A security advisor is a smoke alarm, not a judge

Linters and advisors flag patterns that are *often* wrong. A flagged item can be the correct design (e.g. a definer view that publishes a narrow contract). When you keep one on purpose, write an **ADR** so the next person or agent doesn't "fix" it.

### 11. Settings outside the repo must be written down

Some security lives in a dashboard, not in code (leaked-password protection, anonymous sign-ins, plan-gated features). Nothing in the repo records it, so write it in a living doc (`NOTES.md`, a learning record), including what was deliberately left off.

### 12. Test data is not schema

Migrations describe the structure of **every** database, including production. Keep seed rows and test users out of them (use a separate seed file if needed), and never commit real user ids or passwords.

## The AI-agent angle

When a query returns `[]`, an agent will often suggest "use the service_role / admin key." It works, and it removes your security. Guard against it in `AGENTS.md` with a direct rule, for example: *"Never use the service_role key for user-facing reads. An empty result usually means a missing grant or policy; fix that instead."* Docs shape what the agent proposes, so state the safe path explicitly.

Logic that lives in the database (triggers, definer functions) is invisible from app code. Add a line to `AGENTS.md` that says it exists (e.g. *"Signup creates the profile via a database trigger; the app must not insert profiles"*) so the agent doesn't duplicate it.

## Schema changes only move forward

- Never edit a migration that has already been applied. Add a **new** migration file.
- **Regenerate types when the schema's shape changes** (tables, columns, views), not when permissions change. Grants and policies don't affect types.
- Commit the migration and the regenerated types **together**. Otherwise CI type-checks code against old types and fails.

## Postgres mechanics

Standard Postgres (works on Supabase, Neon, Aurora, any Postgres host):

- **`GRANT` / `REVOKE`**: which roles can touch which tables, views, columns, and functions.
- **View**: a saved, named `SELECT`. Use it as the public contract. Can include derived columns (e.g. age from birthdate).
- **RLS (Row Level Security)**: per-row rules, checked on every query.

### GRANT and RLS are two separate gates

| Grant? | Matching policy? | Result |
| --- | --- | --- |
| No | (not checked) | `permission denied` |
| Yes | No | Zero rows (no error) |
| Yes | Yes | Only the rows the policy allows |

"Permission denied" points at a missing **grant**. An unexpected empty result points at a missing or wrong **policy**.

### Writing policies

- **`using`** filters which existing rows you can see or act on.
- **`with check`** validates what a row looks like **after** an insert or update. On an update policy it stops a user from reassigning their row to someone else.
- Write the caller's id as **`(select auth.uid())`**, not bare `auth.uid()`. Postgres evaluates it once per query instead of once per row.

### Security definer: views and functions

- A **`security definer` view** runs with its owner's rights and ignores RLS on the underlying table. Any "hide these rows" rule must be a `WHERE` clause in the view itself.
- **Functions are executable by everyone by default** (granted to `PUBLIC`). For any function the public shouldn't call over the API: `revoke execute on function ... from public, anon, authenticated;`
- In a `security definer` function, set **`search_path = ''`** and fully qualify every object (`public.musicians`) so it can't be tricked into touching the wrong one.

### Creating a profile on signup

- Put an **`after insert` trigger on the auth users table** that calls a `security definer` function to insert the profile row. A policy on the profile table never sees signup happen, and there may be no session yet.
- A trigger runs **inside the signup transaction**: if the function fails, signup fails. Test any change to it (or to required profile columns) by creating a user.
- A function that **`returns trigger`** isn't exposed over the API, so don't revoke EXECUTE on it the way you would on a callable function.
- Don't seed the public display name from the email. Use a placeholder.

## Supabase specifics

- Roles: `anon` (logged-out visitor), `authenticated` (logged-in user), `service_role` (bypasses RLS, server-only).
- Supabase **auto-exposes tables, views, and functions as a REST API**. The browser can reach them directly, so grants and RLS are the only real lock.
- **Anonymous sign-ins** give visitors the `authenticated` role, not `anon`, so they pass every `to authenticated` grant. Leave them off unless intended.
- **Leaked-password protection** is a dashboard toggle and may require a paid plan. Record it either way.
- **Test as a role** in the SQL Editor: switch the role dropdown from `postgres` to `anon`, or to `authenticated` and pick a user to impersonate.
- Regenerate types after shape changes: `npx supabase gen types typescript --project-id <ref> > types/supabase.ts`. A view appears under `Views`, not `Tables`.
