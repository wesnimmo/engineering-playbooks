# Data-Layer Security Playbook

How to protect data so the protection can't be bypassed. The principles hold on any stack. Postgres/Supabase mechanics are in their own section at the end.

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

### 6. Green tests don't prove security

Mocked tests (MSW) check that your code calls the right endpoint with the right query. They never touch the real database, so they can't check permissions or policies. Verify security against the real database (SQL Editor, or integration tests against a test database).

## The AI-agent angle

When a query returns `[]`, an agent will often suggest "use the service_role / admin key." It works, and it removes your security. Guard against it in `AGENTS.md` with a direct rule, for example: *"Never use the service_role key for user-facing reads. An empty result usually means a missing grant or policy; fix that instead."* Docs shape what the agent proposes, so state the safe path explicitly.

## Schema changes only move forward

- Never edit a migration that has already been applied. Add a **new** migration file.
- After every schema change, **regenerate types**. The types file is a snapshot of the schema, so it goes stale the moment the schema changes.
- Commit the migration and the regenerated types **together**. Otherwise CI type-checks code against old types and fails.

## Postgres / Supabase mechanics

Standard Postgres (works on Supabase, Neon, Aurora, any Postgres host):

- **`GRANT` / `REVOKE`**: which roles can touch which tables, views, and columns.
- **View**: a saved, named `SELECT`. Use it as the public contract. Can include derived columns (e.g. age from birthdate).
- **RLS (Row Level Security)**: per-row rules, checked on every query.

Supabase-specific:

- Roles: `anon` (logged-out visitor), `authenticated` (logged-in user), `service_role` (bypasses RLS, server-only).
- Supabase **auto-exposes tables and views as a REST API**. The browser can reach them directly, so grants and RLS are the only real lock. That's why this topic matters more on Supabase than on a backend where only your own code reaches the database.
- Regenerate types after schema changes: `npx supabase gen types typescript --project-id <ref> > types/supabase.ts`. A view appears under `Views`, not `Tables`.
