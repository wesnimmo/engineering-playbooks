# Engineering Playbooks

My personal, evolving knowledge base for building software well — written as I learn, refined as I grow. Optimized for quick review (phone-friendly) and as a shared reference for teammates.

> These are living documents. They are meant to be revised often. If something here looks thin, that means I haven't learned that part deeply yet — check the commit history to see how the thinking evolved.

## Contents

| Playbook | What it covers |
| --- | --- |
| [Project Startup](./playbooks/project-startup.md) | The end-to-end routine for starting a new app: phases from domain understanding through the first proven vertical slice. |
| [Domain & Glossary](./playbooks/domain-and-glossary.md) | What "domain" means, ubiquitous language, and how to structure a domain/glossary section (Domain-Driven Design basics). |
| [Requirements](./playbooks/requirements.md) | Discovery vs. requirements, user stories + acceptance criteria, scope tiers, and non-functional requirements. |
| [Testing: MSW + TDD](./playbooks/testing-msw-tdd.md) | Mocking the network boundary with MSW, the test-environment rule, and the two testing recipes. |
| [Env & Deployment](./playbooks/env-and-deployment.md) | The `NEXT_PUBLIC_` security boundary, `.env.example` vs `.env.local`, why green CI ≠ deployed, and least privilege. |

## How I use this

- **On my laptop:** apply it while building.
- **On my phone:** read it like flash cards during downtime (GitHub renders Markdown cleanly on mobile).
- **With teammates:** copy the relevant playbook into a project's `docs/` folder so the team — and their AI agents — share the same conventions.

## The one principle behind all of it

Do the provider-agnostic **thinking** (domain, requirements, data model) before the **tooling** (framework, database), and **prove the machine works** (tests + one end-to-end slice) before scaling features.