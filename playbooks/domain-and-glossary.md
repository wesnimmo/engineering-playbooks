# Domain & Glossary

## What "domain" means

The **domain** is the real-world area of knowledge and activity the software is about — the problem space, independent of any code. It includes the **language and rules that domain experts use**, not the technical implementation.

- **Domain** = what the app is *about* (nouns, rules, vocabulary). Stable; barely moves.
- **Features** = what the app *does*. Change constantly.
- **Requirements** = what you're committing to build for a given version.

A quick test for whether something is domain: *"Would a real subject-matter expert recognize this, whether or not this app ever exists?"* If yes → domain. If it only makes sense once the app exists ("the search page has a filter sidebar") → feature, not domain.

## Why write it down first

It's the most stable foundation you have. Front-loading it means everyone — you, teammates, and the AI agent — speaks one language before code locks in assumptions.

## Two ideas from Domain-Driven Design worth stealing

1. **Ubiquitous language** — ONE shared vocabulary used identically in conversation, docs, and code. If the domain says "connection," the table is `connections`, the function is `createConnection`, and the UI says "Connect." Never "friend request" in one place and "connection" in another.
2. **Bounded context** — the scope within which a word has one specific meaning. ("Member" means one thing here; it'd mean something else in a different app. The boundary keeps the meaning stable.)

## Where it lives

In `CONTEXT.md` at the project root, as the agent's briefing. Note: `CONTEXT.md` is an emerging convention in AI-assisted development, not a fixed industry standard — the *practice* (capture the domain + its language early) is standard; the *filename/format* is your team's choice.

## A reasonable structure for a domain/glossary section

> This is one convention, not THE format. Adapt it.

```markdown
## Domain Glossary

### Core entities
- **Musician** — an individual user with a profile. Has exactly one *primary instrument*.
- **Band** — a group a musician owns. Has *openings* for instruments.
- **Opening** — an unfilled instrument slot in a band. (Band status is DERIVED from whether openings exist — never stored.)
- **Connection** — a mutual link between two musicians. Exactly one per pair.

### Ubiquitous language — use these words
- "Looking to join a band" (not "seeking gig")
- "Opening" (not "vacancy" / "job")
- "Connection" (not "friend request" / "follow")

### _Avoid_ these words
- "Friend" — we say *connection*.
- "Available" as a stored flag — availability is a set of *availability types*, and band status is *derived*.

### Key rules
- A musician has one primary instrument (enforced by a partial unique index).
- A connection is unique per unordered pair of musicians.
```

## Relationship to the data model

The domain gives you the **nouns**; the requirements tell you **which nouns and rules v1 needs**. Together they feed the schema. That's the ordering: domain + requirements → data model → schema.