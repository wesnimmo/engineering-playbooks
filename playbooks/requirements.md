# Requirements

## Discovery ≠ Requirements

- **Discovery brain-dump** — raw, exploratory: feature inventory, open questions, scope musings. Ends *with questions*. Totally valid and valuable as an early artifact.
- **Requirements document** — the *distilled, scope-decided* output. Answers the questions the brain-dump raised. States what's IN for v1 and what's explicitly DEFERRED.

The clearest tell you're still in discovery: the doc ends by asking "is this scope creep?" A requirements doc has already decided.

## The most important thing a requirements doc does

**Make the scope decision.** Deferring is not abandoning — record ambition explicitly:

```markdown
## Scope
### v1 (in scope)
- Band-finding: profiles, search, connections, messaging.

### Deferred (post-v1)
- Gear marketplace (its own app-sized domain).
- Lessons marketplace (reviews, background checks, payments — its own domain).
```

## Recommended format: user stories + acceptance criteria

Reads well to teammates, forces the scope decision, and flows directly into TDD (acceptance criteria ≈ your test cases).

```markdown
### Story: Search bands by location and opening
As a musician, I want to search for bands by zip code and instrument opening,
so that I can find local bands looking for someone like me.

Acceptance criteria:
- Given a valid zip, when I search, then I see bands within range that have at least one open instrument.
- Given a zip with no matching bands, then I see an empty state, not an error.
- Given an invalid zip, then I see a validation message.
```

Notice: those criteria are almost exactly the test cases for the `searchBands` action. A requirement with acceptance criteria basically pre-writes the test.

## Other recognized formats (know they exist)

- **PRD** — broader: problem, goals, users, scope, requirements, success metrics, non-goals.
- **"Shall" statements** — formal/enterprise: *"The system shall allow a musician to list one primary instrument."* Heavier; regulated orgs.

## Don't forget non-functional requirements (NFRs)

Functional = what it does. Non-functional = qualities. Even a v1 has some:
- **Privacy/security:** "Musician contact info is visible only to logged-in users." (→ drives an RLS policy.)
- **Auth rules:** who can do what.
- **Performance, accessibility.**

## Where it lives

A **human document**, not an agent-instruction file. `docs/requirements.md` in the repo, or tracked as issues in Linear/GitHub. The agent may read it for context, but its primary readers are people.