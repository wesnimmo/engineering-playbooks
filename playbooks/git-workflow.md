# Git Workflow Playbook

The standard feature loop used on almost every team.

## Every change gets a branch

Even a one-line chore. Committing straight to `main` skips the PR, and the PR is where CI runs and where you review.

## The loop

**Branch → commit → push → PR → green CI → merge on GitHub → switch to main → pull → delete branch.**

```bash
git switch -c feat/short-description     # 1. branch from an up-to-date main
# ...work, commit in small meaningful steps...
git push -u origin HEAD                  # 2. first push of the branch (HEAD = current branch)
```

3. On GitHub: **Compare & pull request**, base `main` ← compare your branch, create the PR.
4. **Wait for CI to go green**, then merge on GitHub. Click **Delete branch** (removes the remote copy).

```bash
git switch main                          # 5. back to main
git pull                                 #    get the merged work
git branch -d feat/short-description     # 6. delete the local branch
git fetch --prune                        # 7. optional: clear stale remote-branch records
```

**Pull before deleting.** Otherwise local `main` doesn't have the merged work yet.

## Before you push: check for stragglers

Run `git status`. Files changed outside the agent's commits (regenerated types, a migration, a doc fix) are easy to leave behind. If the code depends on them, CI fails. Commit the migration and its regenerated types together. A `prepare-pr` skill can run this check for you (see [Working with AI Agents](./working-with-agents.md)).

## Commit messages (Conventional Commits)

| Prefix | Use for |
| --- | --- |
| `test:` | Adding or changing tests (e.g. the failing TDD test) |
| `feat:` | New behavior |
| `fix:` | Bug fix |
| `docs:` | Documentation only |
| `chore:` | Tooling, config, generated files |
| `refactor:` | Restructure without changing behavior |

TDD rhythm: `test:` (red) then `feat:` (green), so the history shows the test came first.

## Merge style: pick one per repo and write it down

| Option | Commits | Effect |
| --- | --- | --- |
| **Create a merge commit** | Kept separate, plus a merge commit | Keeps the step-by-step story |
| **Rebase and merge** | Kept separate, linear history | Keeps the story, cleaner graph |
| **Squash and merge** | Combined into one | Hides the `test:` → `feat:` sequence |

- **State the policy, don't offer a menu.** An agent given two options may pick differently each time. Record the method in `AGENTS.md` (e.g. *"Merge PRs with a merge commit"*).
- **Never squash a branch that has a `test:` then `feat:` pair.** A single-commit branch has nothing to hide, so it doesn't matter there.
- Many teams squash, and that's a valid choice; it trades the step-by-step story for a tidier `main`.
- Enforce it on GitHub: **Settings → General → Pull Requests**, uncheck the methods you don't use.

## Protect `main` in the platform

A rule in a doc ("CI must pass before merging") can be skipped. A platform rule can't. On GitHub, **Settings → Branches (or Rulesets)** for `main`:

- Require a pull request before merging
- Require status checks to pass (select your CI job)
- Block force pushes

With required checks on, the merge button stays disabled until CI passes. That also makes it safer to let an agent push later: the worst it can do is open a PR you decline.

## `-d` vs `-D`

- `git branch -d` refuses to delete a branch whose work isn't in `main`. Treat a refusal as a warning and confirm the merge first.
- After a **squash** merge, `-d` refuses even though the work is in `main` (the commits look different). Use `-D` only after confirming the PR merged.

## What triggers CI

CI runs on whatever the workflow file (`.github/workflows/*.yml`) says, often `pull_request` into `main` and `push` to `main`. A pushed feature branch with no PR may run nothing. Rules in `AGENTS.md` describe policy; they don't start anything.

## When CI is stuck

If runs sit on **Queued** and refreshing does nothing, check the platform's status page (e.g. githubstatus.com) **before** debugging your config. Queued usually means no runner is available, not broken code. After the incident is resolved, cancel and re-run any runs that stay stuck. Don't merge while CI hasn't run.

## The human step

Pushing, opening PRs, and merging can stay manual on purpose. It's a built-in review point before an agent's work reaches the shared repo. See [Working with AI Agents](./working-with-agents.md).
