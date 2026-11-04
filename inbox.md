# Inbox

Raw notes waiting to be folded into the playbooks. Jot here after each lesson; fold every 2–3 lessons, then clear what's been folded.

**Filter before folding:**

- **Principle**: true on any stack → goes in a playbook.
- **Mechanism**: how my stack does it (Postgres, Supabase, Next.js) → goes in a playbook's "specifics" section.
- **Project detail**: only true for one app → stays in that project's docs. Don't fold.

**Entry format:** `- [Lesson N] (principle | mechanism) The idea, in one or two sentences.`

---

## Waiting to fold

- [Lesson 7 prep] (principle) A linter or security advisor is a smoke alarm, not a judge. A flagged warning can be the correct design. When I keep one on purpose, I write an ADR so the next person (or agent) doesn't "fix" it.
- [Lesson 7 prep] (principle) Verify security by impersonating the user, not by reading the code. Run the query as the public role and confirm the forbidden read fails and the allowed read succeeds.
- [Lesson 7 prep] (mechanism) In the Supabase SQL Editor, switch the role dropdown from `postgres` to `anon` to test what the public can actually read.
- [Lesson 7 prep] (mechanism) A SECURITY DEFINER view runs with its owner's rights and ignores RLS on the underlying table. Any "hide these rows" rule has to be a `WHERE` clause in the view itself.
- [Lesson 7 prep] (mechanism) Postgres functions are executable by everyone by default. Revoke `EXECUTE` from `anon` and `authenticated` on any function the public shouldn't call over RPC.
- [Lesson 7 prep] (principle) Every change gets a branch, even a small chore. Committing straight to `main` skips the PR, and the PR is where CI runs.

## Folded

<!-- Move entries here (or delete them) after they're in a playbook. Note which playbook. -->