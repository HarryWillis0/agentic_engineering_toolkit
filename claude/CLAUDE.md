# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) across all repositories.

## Commit messages

Format: `<type>: <short description>` (~50 chars on the first line), optional body for context.

Types: `feat` | `fix` | `refactor`

- Describe **intent**, not the code — what capability does the user gain, or what problem is fixed? Not what files changed or what component was added.
- Add a body only when the why isn't obvious from the title
- When closing a GitHub issue, add `Closes #N` in the body

```
# Bad — describes the implementation:
feat: add persistent stake/profit footer to picks page

# Good — describes the capability:
feat: let users calculate parlay profit from the picks page
```

More examples:
```
feat: let users filter activities by gear

fix: prevent duplicate gear API calls on startup

refactor: extract filter logic into pure module

Closes #1
```

## Git history

Keep history linear — one commit after another, no merge commits. When updating a branch from its base (`dev`/`main`) or another branch, use `rebase`, never `merge`. Never run `git pull` without `--rebase` (or configure `pull.rebase=true`). If a merge commit shows up anyway (e.g. from a `pull` without rebase), it should be flagged, not left in place.

## Coding style

### Boolean conditions

In ternaries and conditional expressions, check the `true` case first:

```ts
// Bad — negated condition puts false case first:
const label = !selected ? 'Add' : 'Remove';

// Good:
const label = selected ? 'Remove' : 'Add';

// Bad — negative number check first:
return value < 0 ? `-$${abs}` : `$${abs}`;

// Good:
return value >= 0 ? `$${abs}` : `-$${abs}`;
```

Apply to ternaries, `if/else`, and `&&`/`||` chains.

### Async/Promise handling

- **Fire-and-forget** (event handlers, sync callback-slot props like `onPress`/`onClick`): `void asyncFn()` is the correct idiom. This is intentional, not a bug to fix.
- **Everything else** (`useEffect`, anything gating a loading/error state, anything where a silent failure matters): use `.then().catch()`, never `void`. `void` only silences the linter — it does not handle a rejection at runtime.
- **Never leave a `.catch()` empty.** Log the error (e.g. `console.error`) so a real failure stays visible even while the app degrades gracefully with a fallback.

### Comments

Keep comments to 1-3 lines: state the intent or non-obvious context, not the full history of how you arrived there. A comment that reads like a changelog entry or an incident writeup is too long — trim it to what the next reader needs to not repeat the mistake.

```ts
// Bad — a story:
// connect-src carries an exception for RevenueCat because its Web Billing
// SDK needs to reach api.revenuecat.com — confirmed by checking the actual
// browser console during a live purchase attempt on 2026-08-13, since the
// SDK's own bundle also references e.revenue.cat for analytics events and
// we didn't want to add domains we hadn't verified were actually hit...

// Good — states intent, briefly:
// api.revenuecat.com: RevenueCat Web Billing SDK, confirmed via live CSP violation
```
