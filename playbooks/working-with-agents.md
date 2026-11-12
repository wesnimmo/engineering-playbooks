# Working with AI Agents

How to keep a coding agent effective across sessions, what decides what it can actually do, and where to draw the line on its autonomy.

## Docs shape behavior; the environment decides capability

An agent can only do what its environment allows. Three things decide that:

1. **Credentials**: is it logged in to GitHub, the database, the host?
2. **Installed tools**: is `gh`, the Supabase CLI, etc. available where it runs?
3. **Sandbox permissions**: can it make network writes or run commands outside its fence?

`AGENTS.md` tells the agent **what it should do**. It can't grant **permission**. When an agent reports "blocked," check those three before editing docs.

Example: your laptop is logged in to GitHub, but the agent runs in a sandbox that can't see that login and blocks writes to GitHub. So it can commit locally but can't push or open a PR.

## Where each kind of agent instruction lives

| Kind | Example | Where |
| --- | --- | --- |
| **Standing fact** | Which env vars exist; the repo's merge method | `AGENTS.md` / `CONTEXT.md` |
| **Guardrail** | "Never use the service_role key for user reads" | `AGENTS.md` or rules files |
| **Repeated procedure** | The TDD loop; preparing a PR | A **skill** |
| **Domain language** | What "connection" means; words to avoid | `CONTEXT.md` glossary |

Agents act on stated reasons. Keep reasons accurate, because a wrong reason (e.g. "hidden because age is sensitive") leads the agent to "fix" things that were intentional.

## Where to draw the autonomy line

**Give the agent autonomy where actions are undoable. Put a human checkpoint where work becomes shared.**

| Step | Who | Why |
| --- | --- | --- |
| Branch, commit, commit messages | Agent | Local and undoable |
| Run lint, type-check, tests before push | Agent | Repetitive; catches CI failures early |
| Draft PR title and description | Agent drafts, human reviews | Tedious to write, cheap to check |
| Push, open the PR | Human (at first) | Work leaves your machine; your review moment |
| Merge | Human | Changes `main` for everyone |

Keeping push and PR manual doesn't defeat the point of using an agent. Choosing the boundary on purpose **is** the architectural decision.

## Enforce the checkpoint in the platform, not a doc

A rule in `AGENTS.md` is good manners; an agent can ignore it. Branch protection with required checks on `main` is the real lock (see [Git Workflow](./git-workflow.md)). It's the same principle as database security: enforce at the lowest layer that can enforce it. Once the platform enforces it, it's safe to widen agent autonomy (e.g. let it push and open PRs while you keep merge).

## Skills automate procedures without standing permission

A skill runs only when you invoke it (e.g. `/prepare-pr`). So it removes a repetitive procedure without letting the agent do that procedure whenever it likes.

Pattern for a **`prepare-pr`** skill: confirm you're on a feature branch → run lint, type-check, and tests and stop on failure → check for uncommitted files and commit-message format → draft the PR text → **stop** and hand back the push command. The `AGENTS.md` rule ("agents commit locally only") stays as the guardrail; the skill is the procedure that respects it.

## Start a new chat at a task boundary

Start a fresh chat when a task, lesson, or feature is done, not only when the context meter is full. A long chat carries every detour and abandoned approach, and that history keeps influencing answers. Start the new chat **after** the records for the finished work are committed.

A fresh chat also tests your docs: if a new agent with no history does the next task correctly, `AGENTS.md`, `CONTEXT.md`, and your skills are doing their job.

## Snapshot files vs living files

| Type | Examples | How it changes |
| --- | --- | --- |
| **Snapshot** | Dated handover, ADR, applied migration | Written once, never edited. Replaced by a new dated file |
| **Living state** | `NOTES.md`, `CONTEXT.md`, `AGENTS.md`, learning records | Updated as things change |

New facts about current state go in living files. Write a new dated handover only when a chat is about to run out of room; never edit an old one. A new chat reads snapshots for **background** and living files for **current state**. When they disagree, say so in the opening prompt, e.g. *"NOTES.md is more recent than the handover's project-state section."*
