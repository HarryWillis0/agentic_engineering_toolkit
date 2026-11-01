---
name: implement-milestone
description: Implement all open issues in a GitHub milestone end-to-end. Surveys the milestone, picks the next unblocked issue, implements with TDD on a fresh branch, commits, opens a PR, then repeats until all issues are done.
---

# Implement Milestone

Work through every open issue in a GitHub milestone, one at a time, until they all have merged commits or open PRs.

## Invocation

```
/implement-milestone [milestone-name-or-number]
```

If no argument is given, list available milestones and ask the user to pick one.

## Phase 1 — Survey

1. Run `gh issue list --milestone "<name>" --state all` to get every issue in the milestone.
2. Run `gh pr list --state open` to see what is already in flight.
3. For each issue, determine its state:
   - **Done**: issue is closed, or a PR with `Closes #N` in the body has been merged into `dev`.
   - **In PR**: an open PR references it (check PR bodies for `Closes #N`, or a branch named after the issue number).
   - **Blocked**: at least one "Blocked by" dependency is not yet Done or In PR.
   - **Ready**: open, and every dependency is either Done or In PR.
4. Print a one-line status table so the user can see the full picture before any code is written.

## Phase 2 — Pick next issue

Select the highest-priority **Ready** issue. Priority order: lowest issue number first (issues were written in dependency order).

If nothing is Ready:
- If everything is Done or In PR → the milestone is complete. Report it and stop.
- If some issues are Blocked by things not yet started → report the blockage and stop.

## Phase 3 — Branch

Determine the correct base branch:

1. For each "Blocked by" dependency of this issue, check whether the blocking PR is **merged** (`gh pr view <N> --json state` → `MERGED`) or still **open**.
2. **If all blockers are merged into `dev`**: `git checkout dev && git pull && git checkout -b <issue-num>-<short-slug>`
3. **If any blocker is still in an open PR**: identify the most recently created blocking branch (the tip of the dependency chain), then: `git checkout <blocking-branch> && git pull && git checkout -b <issue-num>-<short-slug>`

Slug: 2–4 lowercase words from the issue title, hyphen-separated (e.g. `43-skater-stats`).

Never create a branch without first pulling the base to make sure it is up to date.

## Phase 4 — Implement with TDD

Read the full issue body with `gh issue view <N>`. Follow the `/tdd` workflow:

1. Read the acceptance criteria carefully.
2. Explore the relevant code — find existing patterns, interfaces, and tests to follow.
3. Write failing tests first that cover the acceptance criteria.
4. Run the tests to confirm they are red.
5. Implement the minimum code to make the tests pass.
6. Run the tests to confirm they are green.
7. Run `pnpm format` on the files you changed.
8. Run `pnpm typecheck` and the full test suite for every affected package.
9. Fix any lint, type, or test failures before committing.

Do not add features, error handling, or abstractions beyond what the acceptance criteria require.

## Phase 5 — ADR check

Before committing, ask: does this implementation involve a non-obvious architectural decision that a future developer would need to understand? Write an ADR if any of these apply:

- A real alternative was rejected and the reason isn't obvious from the code
- A constraint exists that could bite someone later (e.g. "breaks under horizontal scaling", "lost on restart", "cannot be used with multiple instances")
- The interface or scoping model was designed a specific way for a non-obvious reason

**If yes:** write `docs/adr/NNNN-<slug>.md` (next sequential number). Follow the format in existing ADRs: frontmatter with status/date/decision-makers, then Context, Decision Drivers, Considered Options, Decision Outcome, Consequences, Pros and Cons, More Information.

**If no:** move on. Pure wiring, CRUD, new adapters following an existing pattern, and renaming do not need ADRs.

## Phase 6 — Commit and PR

Run `/commit` to stage, commit (with `Closes #N` in the body), and push the branch.

Then open a PR:

```bash
gh pr create --title "<commit subject>" --base <base-branch> --body "..."
```

- `--base`: use the same branch you branched from (either `dev` or the blocking PR's branch).
- PR body: one short paragraph describing the capability gained, plus `Closes #N`.
- **No test plan section.**

## Phase 7 — Loop

Go back to **Phase 2** and pick the next Ready issue. Continue until all issues in the milestone are Done or In PR.

## Stopping conditions

- All issues are Done or In PR → declare the milestone complete and list the PRs.
- No Ready issues remain and the user must merge something first → pause, list what is blocking, and tell the user which PRs to merge to unblock the next issue.
- The user interrupts → stop cleanly, summarise what was done and what remains.

## Rules

- **One issue per branch.** Never mix two issues on the same branch.
- **Branch from the right base.** Check blocker PR state before branching — branch from the blocking branch if it is not yet merged, from `dev` if it is.
- **Always pull the base branch** before creating the new branch.
- **TDD is non-negotiable.** Write failing tests before writing implementation code.
- **Green before moving on.** All tests, lint, and typecheck must pass before the PR is opened.
- **No speculative work.** Only implement what the current issue's acceptance criteria require.
- **Commit messages follow project conventions** — use the `/commit` skill, not a raw `git commit`.
