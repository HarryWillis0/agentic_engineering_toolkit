---
name: dep-audit
description: Audit packages across the codebase for security vulnerabilities, outdated dependencies, and unused packages, then file grouped GitHub issues labelled `dependencies`. Use when the user wants to check for CVEs, identify upgrade work, or generate a dependency maintenance backlog.
---

# Dependency Audit

Scan all packages for security vulnerabilities, stale dependencies, and unused packages. Group findings into logically cohesive upgrade/removal tracks and file one GitHub issue per track labelled `dependencies`.

## Process

### 1. Gather data

Run these commands and collect their full output:

```bash
pnpm audit
pnpm outdated
```

Also read every `package.json` across the monorepo (root + each workspace package) to understand what is pinned vs range-versioned, and which packages are coupled (e.g. ESLint plugins that must move with ESLint itself).

### 2. Audit pnpm overrides

Read the `pnpm.overrides` field in the root `package.json`. For each override entry:

1. Identify the parent package(s) that originally required the pinned version — check `pnpm why <overridden-pkg>` to find them.
2. For each parent, check what version of the overridden package it now ships: `pnpm list <overridden-pkg>` with the override temporarily removed is the most reliable check, but reading the parent's `package.json` in `node_modules` is also sufficient.
3. If **every parent** now ships a version that satisfies the override's constraint natively (i.e. the override no longer changes what would be installed), the override is stale and should be removed.

Example: if `"cookie": "^0.7.2"` was added because `@sveltejs/kit` shipped `0.6.0`, and kit now ships `0.8.0`, the override does nothing and should be dropped.

Flag stale overrides as a removal item. Group them into the unused-packages issue if one exists, or file a standalone `chore: remove stale pnpm overrides` issue.

### 3. Identify unused packages

For each package listed as a dependency or devDependency across all `package.json` files:

- Search the source files in that package's scope for any `import` or `require` referencing it
- For dev tools (linters, formatters, type checkers), check whether they are referenced in `package.json` scripts or config files
- Flag any package that has no references in source, scripts, or config as a candidate for removal

Do not flag packages that are:
- Peer dependencies required by another installed package
- Type declaration packages (`@types/*`) for a package that is in use
- Workspace packages (`workspace:*`)

Group removals into a single issue unless a removal is coupled to an upgrade (e.g. removing a plugin that is being replaced by a different one in the same ecosystem upgrade).

### 4. Classify findings

**Security vulnerabilities** (from `pnpm audit`):
- Note severity (critical / high / moderate / low), the vulnerable package, the advisory URL, and the fixed version
- Distinguish direct dependencies from transitive ones

**For every transitive vulnerability, verify an override is actually needed before proposing one — do not default to `pnpm.overrides`.** `pnpm.overrides` forces a version the parent was never tested against; it's a last resort, not the first move. For each transitive advisory:

1. `pnpm why <vulnerable-pkg> -r` to find every immediate parent and every resolved version in the tree.
2. For each parent, check its *declared* range for the vulnerable package — not just its currently-resolved version: `npm view <parent>@<resolved-version> dependencies.<vulnerable-pkg>`. Ranges are usually caret/tilde, not exact pins, so the parent may already permit the patched version even though the lockfile hasn't picked it up.
3. **If every parent's declared range already covers the patched version**: no override needed. Note it as "resolves via lockfile refresh" — the fix is `pnpm update <vulnerable-pkg> --recursive` (or equivalent), not a `pnpm.overrides` entry. This is the common case — most transitive advisories are just a stale lockfile entry within an already-compatible range, not a genuine version conflict.
4. **If the package is peer-resolved only** (no real `dependencies`/`devDependencies` edge anywhere in the graph reaches it — check with `pnpm why`, no importer listed) — `pnpm.overrides` cannot reach it; overrides only rewrite dependency-graph edges, not auto-installed peers. If the peer is genuinely used by the project (check for its own config file, e.g. `wrangler.jsonc` for `wrangler`), the correct fix is adding it as an **explicit devDependency** at a version whose declared range covers the fix, not an override. Note this explicitly in the issue so `/dep-implement` doesn't reach for an override that won't work.
5. **Only if no parent's declared range reaches the patched version** is an override genuinely required. When it is, prefer overriding the *outermost* package whose own release already ships the fix as a lockstep-tested unit (e.g. override a wrapper/CLI tool that pins its own sub-dependency by exact version) over force-overriding the innermost transitive package directly into a pairing its immediate parent never tested.

Write the issue's override table only from what survives this check — expect it to be much shorter than the raw advisory list. Call out in the issue body which advisories resolve via plain update vs. which genuinely need an override, so `/dep-implement` doesn't redo this analysis from scratch.

**Outdated packages** (from `pnpm outdated`):
- Classify each as patch, minor, or major bump
- Identify ecosystem groups: packages that are tightly coupled and must upgrade together (e.g. `eslint` + all its plugins; `@typescript-eslint/eslint-plugin` + `@typescript-eslint/parser`)
- Research breaking changes for every major bump before grouping

### 5. Group into upgrade tracks

Order tracks by urgency and risk:

1. **Security patches first** — any vulnerability-fixing update, even if it's also a major bump. Patch/minor security fixes that don't require config changes go in one issue together.
2. **Coupled ecosystem majors** — groups that must move together (e.g. ESLint + all its plugins). One issue per ecosystem group.
3. **Low-risk standalone majors** — major bumps with minimal config impact (e.g. a Prettier plugin). One issue each.
4. **High-risk standalone majors** — majors with significant breaking changes (e.g. TypeScript itself). One issue each, clearly calling out the breaking changes to investigate.
5. **Unused package removals** — one issue covering all confirmed unused packages.

Within each track, list exactly which packages move and from which version to which.

### 6. Research breaking changes

For every major version bump, before writing the issue:
- Check the package's CHANGELOG or GitHub releases for breaking changes
- Note any config file changes, API renames, or new requirements (e.g. minimum Node version, peer dependency changes)
- If the project already satisfies a prerequisite (e.g. already on flat ESLint config), call that out — it reduces the scope

### 7. File issues

One GitHub issue per track. Each issue body must include:

**For upgrade issues:**
- **Packages**: table of package name, current version, target version
- **Key breaking changes**: bullet list of what needs attention — be specific (flag names, API shapes, config format changes). If there are none, say so.
- If the track includes transitive security fixes, split them into two groups per the verification in step 4: **resolves via lockfile refresh** (plain `pnpm update`, no override) vs. **requires an override or explicit devDependency** (name the exact parent-range gap that forces it). Don't present a flat override table — most entries usually belong in the first group.
- **Acceptance criteria**: `pnpm audit` clean (if security), `pnpm typecheck` passes, `pnpm lint` passes, all tests pass

**For unused package removal issues:**
- **Packages to remove**: list each package, which `package.json` it appears in, and confirmation that no source/config references were found
- **Stale overrides to remove**: list each `pnpm.overrides` entry, name the parent that triggered it, and show that the parent now ships a satisfying version natively
- **Acceptance criteria**: packages and overrides removed, `pnpm install` succeeds, `pnpm audit` still clean, `pnpm typecheck` passes, all tests pass

Title format: `chore: <what moves and why>` — e.g. `chore: upgrade ESLint ecosystem to v9/v10` or `chore: remove unused dependencies`.

After creating all issues, apply the `dependencies` label. Create the label (`--color "0366d6"`) if it doesn't exist.

### 8. Report

Summarise findings as tables:

**Security vulnerabilities:**
| Severity | Package | Advisory | Fixed in |

**Upgrade issues filed:**
| Issue | Track | Packages | Risk |

**Unused packages flagged:**
| Package | Found in | Reason flagged |

**Stale overrides flagged:**
| Override | Original reason | Parent now ships |

## Grouping principles

- **Never batch a security fix with a non-security upgrade** — security patches need to ship immediately; don't let them wait on a larger upgrade
- **Keep coupled packages together** — filing separate issues for `eslint` and `@typescript-eslint` creates impossible ordering constraints
- **One PR per issue** — each issue should map to a self-contained branch and PR
- **High-risk majors get their own issue** — TypeScript, Node, or framework major bumps should never share an issue with anything else

## What NOT to file

- Patch or minor bumps with no security advisory and no meaningful changelog — these can be handled by Dependabot or a blanket `pnpm update`
- Packages already at their latest version
- Dev-only transitive dependencies with no user-facing impact and no advisory
