---
name: a11y-audit
description: Audit the codebase for accessibility violations in all frontend UIs and file them as GitHub issues labelled `accessibility`. Use when the user wants to find accessibility problems, identify missing ARIA attributes, or generate a backlog of a11y improvements.
---

# Accessibility Audit

Scan every frontend UI in the repository for accessibility violations and file one GitHub issue per distinct finding, labelled `accessibility`.

## Frontends to audit

This skill is shared across repos with different layouts — don't assume a fixed path. Before auditing, locate every frontend implementation in the current repo: look for `package.json` files with `svelte`/`react`/`vue`/etc. as a dependency, and common layouts like `apps/*`, `frontends/*`, or a single `src/` at the repo root. If unsure, ask the user rather than guessing.

Audit ALL frontends found (skip none — each is a separate implementation). For each, note its framework and component file extensions (e.g. `.svelte`, `.tsx`/`.jsx`) so the Explore step below can target the right files, and exclude `__tests__/`/test files from the scan.

## Principles to audit against

### 1. Semantic HTML
- `<div>` or `<span>` used as interactive elements (button, link, checkbox) instead of native elements
- Missing or wrong landmark elements (`<header>`, `<nav>`, `<main>`, `<footer>`, `<aside>`)
- `<section>` or `<article>` without a heading to label them
- Lists rendered as styled `<div>`s instead of `<ul>`/`<ol>`/`<li>`
- Tables missing `<caption>`, `<th scope>`, or `<thead>`

### 2. Images and media
- `<img>` elements missing `alt` attribute
- Decorative images without `alt=""` (empty string)
- Complex images (charts, diagrams) without a long description (`aria-describedby` or `longdesc`)
- `<figure>` elements without `<figcaption>` when the image conveys meaning

### 3. Forms and interactive controls
- `<input>`, `<select>`, `<textarea>` without an associated `<label>` (via `for`/`id` pair or `aria-label` or `aria-labelledby`)
- `<button>` with no accessible name (no text content, no `aria-label`, no `aria-labelledby`)
- Form controls missing required-field indicators (both visual and `aria-required`)
- Error messages not associated with their field via `aria-describedby`
- Missing `autocomplete` attributes on common fields (name, email, etc.)

### 4. Keyboard navigation
- Interactive elements (`<div onClick>`, `<span onClick>`) that aren't reachable via keyboard
- Missing `tabindex` (or incorrect values) on custom interactive components
- No visible focus indicator (e.g. `outline: none` / `focus:outline-none` with no replacement)
- Logical tab order broken by `tabindex` values > 0
- Modal dialogs / popovers that don't trap focus while open

### 5. ARIA usage
- `aria-label` or `aria-labelledby` missing on icon-only buttons, landmark regions with duplicate roles, and complex widgets
- `aria-hidden="true"` applied to elements that are still keyboard-focusable
- ARIA roles applied to elements whose native role conflicts (e.g. `role="button"` on an `<a>` without `href`)
- Required ARIA child/parent relationships missing (e.g. `role="listbox"` without `role="option"` children)
- Dynamic content that updates without `aria-live` or `aria-atomic` where a screen reader would miss the change

### 6. Colour and contrast
- Flag usage of hard-coded colour values or Tailwind colour utilities where contrast between text and background is likely below WCAG AA (4.5:1 for normal text, 3:1 for large text). Look for light-on-light or dark-on-dark pairings.
- Text rendered solely via `::before`/`::after` CSS pseudo-elements (invisible to screen readers)

### 7. Page-level structure
- `<html>` element missing `lang` attribute
- `<title>` missing or empty on any route/page
- Multiple `<h1>` elements on a single page, or heading hierarchy that skips levels (e.g. `<h1>` → `<h3>`)
- Navigation landmarks without `aria-label` to distinguish multiple `<nav>` elements

### 8. Links
- `<a>` with non-descriptive text ("click here", "read more", "link") with no additional context via `aria-label`
- `<a>` with no `href` used as a button (should be `<button>`)
- Links that open in a new tab/window without warning the user (`target="_blank"` without indication)

## Process

### 1. Explore

Spawn an Explore agent to read ALL component and route files across every frontend located above (breadth: very thorough). Instruct it to find concrete violations of each principle above, with exact file paths and line numbers. Ask it to note which frontend each finding belongs to.

### 2. Verify

Before filing, read each cited file at the flagged line numbers yourself to confirm the finding is genuine. Drop findings that are:
- False positives (e.g. `aria-label` added dynamically via props that the static analysis missed)
- Already mitigated by framework-level behaviour (e.g. SvelteKit's automatic `<svelte:head>` title management)
- Present in test files, generated files, or config (not shipped UI)

Confirm each finding is a real, actionable problem a user with a disability would encounter.

### 3. Ensure the label exists

Before filing any issues, check whether the `accessibility` label exists in the repo:

```bash
gh label list
```

If it does not exist, create it:

```bash
gh label create "accessibility" --description "Accessibility improvements and WCAG compliance" --color "0052cc"
```

### 4. Group into issues

**One issue per distinct violation** (not one per file). If the same violation appears in multiple frontends, file a single issue that calls out every location. Each issue body must include:

- **Frontend(s) affected**: which frontend(s) the violation exists in
- **Problem**: which WCAG criterion or accessibility principle is violated, why it matters to users with disabilities, and a code snippet showing the violation
- **Fix**: concrete description of the change with a before/after snippet
- **Files to update**: list of file paths and line numbers
- **Acceptance criteria**: bullet list of what done looks like (e.g. "all `<img>` elements have a non-empty `alt` attribute")

Title format: `fix: <capability gained>` — e.g. `fix: make article images accessible to screen readers`.

### 5. Apply the label

Apply the `accessibility` label to every issue as it is created (pass `--label "accessibility"` to `gh issue create`).

### 6. Report

Summarise all filed issues as a table:

| Issue | Title | WCAG criterion | Frontend(s) |
|-------|-------|----------------|-------------|

## What NOT to file

- Theoretical violations with no real user impact
- Violations already fixed in one frontend but not another — file a single cross-frontend issue instead of separate ones
- Issues already tracked in open GitHub issues (check with `gh issue list --label accessibility` before filing)
- Styling nits with no assistive-technology impact
- Violations in `__tests__/` directories or build artefacts
