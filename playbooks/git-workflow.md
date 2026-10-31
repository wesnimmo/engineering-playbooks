# Git Workflow Playbook

The standard feature loop used on almost every team.

## The loop

**Branch → commit → push → PR → green CI → merge on GitHub → switch to main → pull → delete branch.**

```bash
git switch -c feat/short-description     # 1. branch from an up-to-date main
# ...work, commit in small meaningful steps...
git push -u origin HEAD                  # 2. first push of the branch (HEAD = current branch)
```

3. On GitHub: **Compare & pull request**, base `main` ← compare your branch, create the PR.
4. Wait for CI to go green. Merge on GitHub. Click **Delete branch** (removes the remote copy).

```bash
git switch main                          # 5. back to main
git pull                                 #    get the merged work
git branch -d feat/short-description     # 6. delete the local branch
git fetch --prune                        # 7. optional: clear stale remote-branch records
```

**Pull before deleting.** Otherwise local `main` doesn't have the merged work yet.

## Before you push: check for stragglers

Run `git status`. Files changed outside the agent's commits (regenerated types, a migration, a doc fix) are easy to leave behind. If the code depends on them, CI fails. Commit the migration and its regenerated types together.

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

## Choosing a merge style

| Option | Commits | When |
| --- | --- | --- |
| **Create a merge commit** | Kept separate, plus a merge commit | Keeps the story visible |
| **Rebase and merge** | Kept separate, linear history | Keeps the story, cleaner graph |
| **Squash and merge** | Combined into one | Hides the `test:` → `feat:` sequence |

Pick on purpose. Many teams squash, and that's a valid team choice; it just trades the step-by-step story for a tidier `main`.

## `-d` vs `-D`

- `git branch -d` refuses to delete a branch whose work isn't in `main`. Treat a refusal as a warning and confirm the merge first.
- After a **squash** merge, `-d` refuses even though the work is in `main` (the commits look different). Use `-D` only after confirming the PR merged.

## What triggers CI

CI runs on whatever the workflow file (`.github/workflows/*.yml`) says, often `pull_request` into `main` and `push` to `main`. A pushed feature branch with no PR may run nothing. Rules like "CI must pass before merging" in `AGENTS.md` describe policy; they don't start anything. The trigger lives in the workflow file.

## The human step

Pushing and opening PRs can stay a manual step on purpose. It's a built-in review point before an agent's work reaches the shared repo. See [Working with AI Agents](./working-with-agents.md).
