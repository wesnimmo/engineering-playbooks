# Project Startup Playbook

The repeatable routine for starting a new app. Structured as **phases**, because the phase boundaries — not the individual steps — are what make the routine transferable to teammates and to other tech stacks.

## Guiding principle

> Do the provider-agnostic **thinking** (Phases 1–2) before the **tooling** (Phase 3), and **prove the machine** (Phase 4) before scaling features.

Phases 1, 2, and 4 are largely stack-independent — they transfer to a Neon/Prisma project, a different framework, or a teammate. Only Phase 3 is specific to the chosen stack (here: Next.js + Supabase).

---

## Phase 1 — Understand (provider-agnostic; all thinking, no tooling)

1. **Domain brain-dump → `CONTEXT.md`.** Capture the real-world subject matter and its vocabulary. Build a ubiquitous-language glossary (what each word means, plus an `_Avoid_` list of words NOT to use). See [Domain & Glossary](./domain-and-glossary.md).
2. **Lo-fi wireframes + user goals → discovery notes.** Sketch cheaply. Write down what users are trying to accomplish. Let the sketches and goals argue with each other. Do NOT build a hi-fi prototype yet — it's expensive and gets invalidated the moment the data model teaches you something.
3. **Distill discovery into requirements.** Turn the brain-dump into settled decisions: user stories + acceptance criteria, grouped by scope tier (v1 / deferred). This is where the scope decision gets MADE. See [Requirements](./requirements.md).
4. **Design the data model (its own phase — the hardest thinking).** Entities, relationships, and **ADRs** for the expensive/irreversible calls (derive-vs-store, enum-vs-lookup, normalization, uniqueness rules). The ER diagram is the *output*, not the activity. Keep it provider-agnostic — it's just entities and relationships, true regardless of database.

## Phase 2 — Commit to the look (now hi-fi earns its cost)

5. **Hi-fi Figma prototype — AFTER the model exists**, so the UI reflects real fields and real derived-vs-stored distinctions. Reconcile it against the requirements from Phase 1.
   - *Caveat:* designer-led teams sometimes legitimately go Figma-first and derive the model from it. Model-first is chosen here because the goal is a transferable, data-integrity-focused routine.

## Phase 3 — Set up the machine (tooling; the swappable phase)

6. **Scaffold + agent config.** Create the Next.js project, install dependencies, write `AGENT.md` and rules files. `git init`, first commit, connect remote (`git push -u origin main` on the first push).
7. **Pick and connect the database.** (Supabase here; this is the swappable slot.) Then translate the model into a **versioned migration** (`supabase/migrations/<timestamp>_name.sql`), apply it, and **generate types** into `types/supabase.ts` (import as `@/types/supabase`).
   - Apply via SQL Editor → Run for a hosted project (no CLI/Docker needed). Verify in Table Editor (empty RLS-protected tables = correct).
   - `.env.local` holds literal values (never committed): URL as bare origin (no trailing slash / no `/rest/v1`), publishable key → `NEXT_PUBLIC_SUPABASE_ANON_KEY`, secret key → `SUPABASE_SERVICE_ROLE_KEY`. Restart the dev server after editing env.

## Phase 4 — Prove it works (de-risk before feature sprawl)

8. **Stand up the test harness** (Jest + MSW) and any reusable **skills** (e.g. a TDD-for-Supabase-actions skill). See [Testing: MSW + TDD](./testing-msw-tdd.md).
9. **Build ONE vertical slice end-to-end, TDD'd** (one action + one screen) to validate the whole stack before writing more features. Write the failing test first.

---

## The three documents and where they live

| Doc | Primary audience | Lives where | Nature |
| --- | --- | --- | --- |
| `CONTEXT.md` / `AGENT.md` | The AI agent (and devs) | Project **root**, in repo | Instructions the agent reads each session |
| Requirements | Humans (team, future you) | `docs/` or a tracker (Linear/GitHub issues) | Source of truth for *what to build* |
| Data model / ADRs | Devs | `docs/` in repo | The reasoning behind schema decisions |

**Domain (nouns) + Requirements (which nouns/rules v1 needs) → feed the Data Model.** That's why both precede the schema.

## When to go lighter

For a tiny app or throwaway prototype, collapse Phases 1–2 into a quick sketch and skip the ADRs. The full routine earns its keep on real, multi-person, long-lived projects.