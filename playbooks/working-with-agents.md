# Working with AI Agents

How to keep a coding agent effective across sessions, and what decides what it can actually do.

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
| **Standing fact** | Which env vars exist; which key is public | `AGENTS.md` / `CONTEXT.md` |
| **Guardrail** | "Never use the service_role key for user reads" | `AGENTS.md` or rules files |
| **Repeated procedure** | The TDD loop for a Supabase action | A **skill** |
| **Domain language** | What "connection" means; words to avoid | `CONTEXT.md` glossary |

Agents act on stated reasons. Keep reasons accurate, because a wrong reason (e.g. "hidden because age is sensitive") leads the agent to "fix" things that were intentional.

## Start a new chat at a task boundary

Start a fresh chat when a task, lesson, or feature is done, not only when the context meter is full. A long chat carries every detour and abandoned approach, and that history keeps influencing answers.

A fresh chat also tests your docs: if a new agent with no history does the next task correctly, `AGENTS.md`, `CONTEXT.md`, and your skills are doing their job.

## Snapshot files vs living files

| Type | Examples | How it changes |
| --- | --- | --- |
| **Snapshot** | Dated handover, ADR, applied migration | Written once, never edited. Replaced by a new dated file |
| **Living state** | `NOTES.md`, `CONTEXT.md`, `AGENTS.md`, learning records | Updated as things change |

A new chat reads snapshots for **background** and living files for **current state**. When they disagree, say so in the opening prompt, e.g. *"NOTES.md is more recent than the handover's project-state section."*

## Keep a human review point

Let the agent write code and commit locally; do the push and PR yourself. It's the natural moment to read the diff before it reaches the shared repo. If you later let the agent push (`gh auth login`, approved commands), keep reviewing the PR before merging.
