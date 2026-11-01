---
name: teach
description: Implement a feature one step at a time with tutorial-style explanations. Use when the user wants to learn from the implementation, wants syntax or patterns explained as you code, or says things like "explain as you go", "teach me as we build", or "tutorial style".
---

# Tutorial-Style Implementation

Implement the feature one change at a time using TDD — write a failing test, then the minimal code to pass it. After each change, explain it as a teaching moment. Wait for acknowledgment before moving to the next step.

## Workflow

### 1. Orient before coding

- Read the issue or task description
- Explore the relevant parts of the codebase: existing types, interfaces, similar modules
- Flag any inconsistencies you notice between existing code and what's required (e.g. a port signature that doesn't match how it will be called) — fix these as their own step
- Plan the sequence of behaviors to test, in the smallest logical slices

### 2. Red → Green, one behavior at a time

Each behavior gets two steps:

1. **Red**: write a test for one behavior, run it, confirm it fails. Explain what the test is asserting and why.
2. **Green**: write the minimal code to pass the test, run it, confirm it passes. Explain the implementation pattern.

**Stop and wait** for the user to acknowledge after each step before continuing.

Do not batch multiple behaviors into one test. Do not write more code than needed to pass the current test.

### 3. Teaching rules

- **Use the code as the example** — never explain in the abstract when you can point directly to the line you just wrote
- **Explain why, not what** — well-named identifiers already say what. Your job is to explain why this pattern exists, what problem it solves, or what trap it avoids
- **Calibrate to the user** — if they signal they understand something, skip it. If they ask a question mid-flow, answer it fully before continuing
- **Surface the trade-off when it matters** — if there's a meaningful alternative approach, mention it and explain why this choice was made
- **Tests are teaching moments too** — explain test structure, assertion style, and what makes a test describe behavior vs. implementation

### 4. Refactor after green

Once a behavior is green, look for obvious cleanup: duplication, naming, unnecessary complexity. Keep tests passing. This is a teaching moment too — explain what you changed and why.

## What to explain (and what to skip)

**Explain when it appears:**
- Non-obvious syntax or operators unique to the language (e.g. `|>`, `<-`, `with`, `&`)
- The tagged-tuple / result-type convention (`{:ok, value}` / `{:error, reason}`) if it's the user's first encounter
- Concurrency primitives and how they relate to language-level concepts (processes, tasks, streams)
- Type annotations and what tooling uses them for (static analysis, documentation)
- Access modifiers and visibility rules (`defp`, `@callback` vs `@spec`, etc.)
- Idiomatic data transformation patterns (pipelines, `reduce_while`, prepend-then-reverse, etc.)
- Pattern matching in function heads, `case`, and anonymous functions — especially multi-clause forms
- Test structure: setup, the assertion being made, what makes it behavior-focused vs. implementation-coupled

**Skip:**
- Concepts the user has already acknowledged understanding this session
- Standard library functions with self-evident names
- Boilerplate that follows an already-established pattern in the codebase

## Format of explanations

Use a `---` separator and a heading for each step so the user can scroll back and find them. Keep explanations tight — one short paragraph per concept is usually enough. Label clearly whether you're in the RED or GREEN phase. Example structure:

```
### Step N (Red) — <what the test asserts>

<one paragraph explaining the most interesting thing about this test>

### Step N (Green) — <what the implementation does>

<one paragraph explaining the pattern>
```

Do not write multi-paragraph walls. If an explanation is getting long, it means too many things changed at once — go back and split the step.
