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
- [Lesson 7 prep] some security settings live in the dashboard, not in code, so write them down or they'll be lost.
- [Lesson 7] (principle) A login account is not a profile. Auth owns identity (auth.users); my table owns the profile; one foreign key (auth_user_id) links them.
- [Lesson 7] (mechanism) GRANT and RLS are two separate gates. GRANT decides whether a role may touch the table at all; an RLS policy decides which rows it sees. No grant → denied before policies run. Grant but no matching policy → zero rows.
- [Lesson 7] (mechanism) In a policy, `using` filters which existing rows you can act on; `with check` validates what the row looks like after an insert/update (stops reassigning auth_user_id to someone else).
- [Lesson 7] (mechanism) Write `(select auth.uid())`, not bare `auth.uid()`, in policies. Postgres evaluates it once per query instead of once per row.
- [Lesson 7] (principle) Test the negative cases, not just the happy path. Owner sees their row, a second user sees nothing, anon is denied. One passing case proves little.
- [Lesson 7] (principle) Regenerate types when the schema's shape changes, not when permissions change. Grants and policies don't touch types.
- [Lesson 7] (principle) Test data is not schema. Keep seed rows and test users out of migrations, and never commit real auth UUIDs or passwords.
- [Lesson 7] (principle) Some security settings live in the dashboard, not the repo. Anything not in a migration must be written down, or it's invisible to the next person.
- [Lesson 7] (mechanism) Supabase anonymous sign-ins give visitors the `authenticated` role, not `anon`. Leave them off unless intended, or they pass every `to authenticated` grant.
- [Lesson 7] (principle) Handovers are append-only snapshots; living docs (NOTES, learning records) get updated. Start a new chat at a lesson boundary, after the records are committed.
 
## Folded

<!-- Move entries here (or delete them) after they're in a playbook. Note which playbook. -->