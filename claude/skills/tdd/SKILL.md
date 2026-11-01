---
name: tdd
description: Test-driven development with red-green-refactor loop. Use when user wants to build features or fix bugs using TDD, mentions "red-green-refactor", wants integration tests, or asks for test-first development.
---

# Test-Driven Development

## Philosophy

**Core principle**: Tests should verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

**Good tests** are integration-style: they exercise real code paths through public APIs. They describe _what_ the system does, not _how_ it does it. A good test reads like a specification - "user can checkout with valid cart" tells you exactly what capability exists. These tests survive refactors because they don't care about internal structure.

**Bad tests** are coupled to implementation. They mock internal collaborators, test private methods, or verify through external means (like querying a database directly instead of using the interface). The warning sign: your test breaks when you refactor, but behavior hasn't changed. If you rename an internal function and tests fail, those tests were testing implementation, not behavior.


## Anti-Pattern: Horizontal Slices

**DO NOT write all tests first, then all implementation.** This is "horizontal slicing" - treating RED as "write all tests" and GREEN as "write all code."

This produces **crap tests**:

- Tests written in bulk test _imagined_ behavior, not _actual_ behavior
- You end up testing the _shape_ of things (data structures, function signatures) rather than user-facing behavior
- Tests become insensitive to real changes - they pass when behavior breaks, fail when behavior is fine
- You outrun your headlights, committing to test structure before understanding the implementation

**Correct approach**: Vertical slices via tracer bullets. One test → one implementation → repeat. Each test responds to what you learned from the previous cycle. Because you just wrote the code, you know exactly what behavior matters and how to verify it.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  RED→GREEN: test3→impl3
  ...
```

## Workflow

### 1. Planning

When exploring the codebase, use the project's domain glossary so that test names and interface vocabulary match the project's language, and respect ADRs in the area you're touching.

Before writing any code:

- [ ] Confirm with user what interface changes are needed
- [ ] Confirm with user which behaviors to test (prioritize)
- [ ] Identify opportunities for [deep modules](deep-modules.md) (small interface, deep implementation)
- [ ] Design interfaces for [testability](interface-design.md)
- [ ] List the behaviors to test (not implementation steps)
- [ ] Get user approval on the plan

Ask: "What should the public interface look like? Which behaviors are most important to test?"

**You can't test everything.** Confirm with the user exactly which behaviors matter most. Focus testing effort on critical paths and complex logic, not every possible edge case.

### 2. Tracer Bullet

Write ONE test that confirms ONE thing about the system:

```
RED:   Write test for first behavior → test fails
GREEN: Write minimal code to pass → test passes
```

This is your tracer bullet - proves the path works end-to-end.

### 3. Incremental Loop

For each remaining behavior:

```
RED:   Write next test → fails
GREEN: Minimal code to pass → passes
```

Rules:

- One test at a time
- Only enough code to pass current test
- Don't anticipate future tests
- Keep tests focused on observable behavior

### 4. Refactor

Apply Uncle Bob's clean coding principles during refactor:

- [ ] **Each function does ONE thing** — if you need "and" to describe it, split it
- [ ] **No hidden side effects** — a function named as a query must not mutate state (Command-Query Separation)
- [ ] **Small functions** — if it doesn't fit on a screen, it's doing too much
- [ ] **Meaningful names** — names should tell you *why*, not *how*
- [ ] **No flag parameters** — if a boolean changes what a function does, split it into two functions
- [ ] Extract duplication
- [ ] Deepen modules (move complexity behind simple interfaces)
- [ ] **Apply port interfaces for third-party dependencies** — when a library import leaks a foreign interface into application code (awkward type casts, unsafe-member access, hard-to-mock behaviour), define your own interface (the port) and wrap the library behind it. The adapter owns all the noise; callers depend only on the port. This makes dependencies swappable and keeps the domain free of framework churn.
- [ ] Apply SOLID principles where natural
- [ ] Consider what new code reveals about existing code
- [ ] Run tests after each refactor step

**Never refactor while RED.** Get to GREEN first.

### Formatting

Format every file as soon as your edit to it is accepted — don't batch formatting until the end of the session. Use the project's configured formatter (e.g. `prettier --write <file>` for JS/TS, `gofmt -w` for Go, `black` for Python). Check for a config file (`.prettierrc`, `pyproject.toml`, a `format` script in `package.json`, etc.) before assuming none exists.

### 5. Refactor review

Once all cycles are green, step back and look at the full implementation. Ask: does anything now look like a code-audit finding? Check for:

- **SRP** — does any function or module do more than one thing? (e.g. a component that both fetches data and applies business logic)
- **DRY** — is any logic duplicated that should be shared?
- **Service extraction** — are network calls inline in a component or hook instead of a dedicated service?
- **Deep modules** — can a complex block be hidden behind a simpler interface?
- **Naming** — do names describe *why*, not *how*?
- **Clean boundaries** — are third-party SDKs leaking into application code rather than wrapped behind a port?

For each finding: propose the refactor and its scope, get user approval, implement, run tests. These are separate commits from the feature work if the change is significant.

**This step is not part of the red/green cycle.** Don't touch tests unless the refactor changes a public interface.

### 6. ADR check

Ask: does this implementation involve a non-obvious architectural decision that a future developer would need to understand? If yes, propose writing an ADR. Signals that warrant one:

- A real alternative was rejected (and the reason isn't obvious from the code)
- A constraint exists that could bite someone later (e.g. "breaks under horizontal scaling", "lost on restart")
- The interface was designed a specific way for a non-obvious reason

Signals that don't warrant one: pure wiring, CRUD, renaming, moving files, adding a straightforward new adapter that follows an existing pattern.

## Checklist Per Cycle

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] Code is minimal for this test
[ ] No speculative features added
[ ] Changed files formatted with the project formatter
```

