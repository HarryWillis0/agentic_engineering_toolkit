---
name: commit
description: Create a git commit following project conventions. Derives rules from git history, stages changes, writes the message, and pushes the branch.
---

# Commit

Create a git commit following the project's conventions.

## Rules

1. **Conventional Commits format**: `<type>: <description>` — no scope in parentheses
2. **Lowercase everything**: type and description are both all-lowercase
3. **Imperative present tense**: start the description with a verb ("add", "remove", "use", "fix")
4. **Body for larger changes**: when a commit touches multiple concerns or the subject line alone doesn't convey intent, add a blank line after the subject and write a short body explaining *why* — not what the code does
5. **No trailing period**
6. **Types**: `feat`, `fix`, `refactor` are standard; extend with `ci`, `chore`, `docs` when the change warrants it
7. **Intent, not code**: the subject describes what capability the user gains or what problem is solved — not what component was added or what file changed. Ask "what can someone now do that they couldn't before?" Bad: `add stake footer to picks page`. Good: `let users calculate parlay profit from the picks page`.

## Process

1. Run `git log --oneline -20` to confirm current conventions
2. Run `git diff --cached` to understand what's staged
3. Check if the current work is tied to a GitHub issue: look at the branch name (e.g. `feat/42-thing` or `issue-42`) or ask the user if it's not clear. If an issue number is known, add `Closes #<N>` (or `Fixes #<N>`) as a trailer in the commit body.
4. Draft a subject line; add a body if the change spans multiple concerns, the intent isn't obvious from the subject, or there's an issue to close
5. Commit, then push — if the branch has no upstream, set it with `git push -u origin <branch>`
6. If the user asks to open a PR after committing, create one with `gh pr create`. The body should describe the intent of the changes (what capability is gained or problem solved — not what code was added), plus a `Closes #<N>` trailer if there's an issue. Do **not** include a test plan section.

## Keep history linear — no merge commits

History should read as one commit after another. Never create a merge commit:

- Updating a branch from `dev`/`main` or another branch: use `git rebase`, not `git merge`.
- Pulling: use `git pull --rebase`, never a plain `git pull`.
- If a merge commit is already present in the branch's history, flag it to the user rather than silently leaving it — don't rewrite shared history without asking.

## Squashing pipeline fixes

If subsequent commits exist solely to fix CI failures caused by the immediately preceding commit (lint errors, typecheck cascades, broken test fixtures), squash them back into that commit before the PR is reviewed:

1. `git reset --soft HEAD~<N>` — uncommit the original + all fix commits, keeping everything staged
2. Recommit with the **original message only** — do not mention the fixes in the commit message
3. `git push --force-with-lease`

The fix work belongs in the original commit; it should never appear as separate history. Only squash commits that fix the CI fallout of the commit immediately before them — unrelated fixes stay separate.
