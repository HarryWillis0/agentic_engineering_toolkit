---
name: code-audit
description: Audit the codebase for Uncle Bob clean code violations and file them as GitHub issues labelled software-excellence. Use when the user wants to find refactor opportunities, audit code quality, or generate a backlog of engineering improvements.
---

# Code Audit

Scan the codebase for clean code violations, group them into focused actionable GitHub issues, and label them `software-excellence`.

## Principles to audit against

Check every source file for breaches of these rules:

**Command-Query Separation (CQS)**
- Functions that both mutate state AND return a value
- Queries that have hidden side effects (e.g. a `load` function that also prunes stale data)

**Single Responsibility Principle (SRP)**
- Classes implementing more than one interface for unrelated concerns
- Functions whose description requires "and"
- Files mixing multiple formatting/transformation/fetch concerns

**Small functions**
- Functions longer than ~20 lines that could be extracted into named steps
- Anonymous inline logic (arrow functions > 5 lines inside `.map`/`.filter`) that deserves a name

**Deep nesting**
- Three or more levels of nesting — extract the innermost body into a named function

**Long parameter lists**
- Functions taking 4+ positional parameters — group related args into a context object

**Hidden side effects**
- Broad try/catch blocks that swallow all errors and return a default, hiding real failures
- Silent failures where a caller cannot distinguish "not found" from "fetch failed"

**Don't repeat yourself**
- Functions called twice in the same expression to test truthiness then get the value
- Duplicate logic that could be a shared helper

**Naming**
- Names that describe *what* (implementation) rather than *why* (intent)
- Inconsistent naming conventions across related functions in the same module

## Process

### 1. Explore

Spawn an Explore agent to read ALL source files across the project's package structure. Ask it to find concrete violations of each principle above, with file paths and line numbers. Cast a wide net — medium to thorough breadth.

### 2. Verify

Before filing, read the cited files yourself at the flagged line numbers to confirm each finding is real and the fix is clear. Drop findings that are:
- Purely stylistic with no behaviour impact
- Already documented or intentional (check comments/ADRs)
- Too vague to be actionable

### 3. Group into issues

One issue per violation (not per file). Each issue body must include:
- **Problem**: what rule is broken and why it matters, with a code snippet showing the violation
- **Fix**: concrete description of what the change looks like (can include a before/after snippet)
- **Acceptance criteria**: bullet list of what done looks like

Keep titles in the project's commit convention format: `refactor: <intent>` or `fix: <intent>`.

### 4. Label

After creating all issues, apply the `software-excellence` label to each one. Create the label if it doesn't exist.

## What NOT to file

- Naming inconsistencies that are purely aesthetic
- Missing comments or documentation
- Performance optimisations (unless caused by a structural issue like double-calling a function)
- Issues already tracked in open GitHub issues
- Violations in test files, generated files, or config

## Output

Summarise what was filed as a table: issue number, title, rule violated.
