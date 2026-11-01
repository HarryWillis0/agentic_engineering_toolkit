---
name: ux-audit
description: Act as a UI/UX expert to analyze a UI area for usability problems, propose scoped recommendations, and — only after the user explicitly approves — file the approved ones as GitHub issues labelled `ux`. Use when the user wants a UX review, thinks a screen or flow feels clunky, or wants design recommendations to approve before they become issues.
---

# UX Audit

Act as a UI/UX expert specializing in simple, self-explanatory, beautiful design. Analyze a specific UI area, turn real problems into scoped recommendations, get explicit user approval, then file only the approved ones as GitHub issues labelled `ux`.

The defining trait of this skill, unlike sibling audit skills (`a11y-audit`, `code-audit`, `dep-audit`): **nothing gets filed automatically.** UX judgment is subjective — the user approves before anything becomes an issue.

## 1. Scope the review

UX problems are contextual, not mechanically checkable like lint violations — don't blanket-scan the whole app.

- If the user named a specific screen, component, or flow, scope to that.
- If they didn't, ask what to review before reading any code. A whole-app pass is a valid answer but produces broader, shallower findings — say so if the user picks it.

## 2. Ground the review in reality

Never critique from memory or generic design theory — every claim must trace to actual code or an actual running screen.

- Read the target component(s) in full, plus the page/route(s) that mount them and any server actions/handlers they call.
- Read the app's shared design tokens (theme file, global CSS, tailwind config) so recommendations speak the app's actual visual language, not generic advice.
- Grep for existing UI patterns elsewhere in the codebase — modals/sheets, loading-state conventions, error-display conventions, empty states. Recommendations should reuse what already exists in the codebase over inventing a new one-off pattern.
- If a dev server is already reachable (check for one listening before starting a new one), load the real UI in the browser via the `claude-in-chrome` skill before critiquing it. If no browser tool is available or nothing is running, say plainly that the review is source-grounded only — don't guess at rendered appearance.

## 3. Evaluate against these principles

- **Discoverability** — does every visible control's label/icon accurately predict what happens, for *all* user states (not just the happy path)? A "Settings" button that's empty for anonymous users is a discoverability bug.
- **Feedback** — does every user-triggered action produce a visible loading/success/failure response? Silence after a click is a defect.
- **Error visibility & recovery** — are errors visually distinct from ordinary helper text, and can the user fix and retry without restarting the whole flow?
- **Consistency & reuse** — does this area match patterns/components/tokens already established elsewhere in the codebase, or does it quietly diverge?
- **Grouping & hierarchy** — are unrelated concerns visually separated, and are related controls grouped under a clear label?
- **Progressive disclosure** — is secondary detail hidden until needed, without hiding what the user needs to decide?
- **Layout stability** — do state changes (opening a panel, loading data) cause jarring reflow of surrounding content?
- **Control & recoverability** — can the user undo, cancel, or back out of a multi-step flow without a hard reset (closing/reopening, reloading)?
- **Basic interaction accessibility** — keyboard operability and ARIA state on custom (non-native) widgets, *only* where it's inseparable from a usability finding. Full accessibility/contrast auditing belongs to `a11y-audit` and the `accessibility` label — don't duplicate that skill's territory.

## 4. Verify before proposing

For every candidate finding, re-read the cited file at the cited lines yourself — don't speculate about behavior you haven't read. Check `gh issue list` so you don't re-propose something already tracked.

## 5. Present recommendations for approval

- Write a short, grounded "current state" section citing `file:line` for each real problem.
- Turn each real problem into one independently-scoped recommendation, sized to be one issue and one PR (check the project's CLAUDE.md/conventions for branch-per-issue expectations).
- Lead each recommendation with the concrete before → after change, not design theory.
- Present them with `AskUserQuestion`, `multiSelect: true`, batched up to 4 options per question (max 4 questions per call — split across multiple questions if there are more than 4 recommendations). Put your top picks first and label them "(Recommended)".
- **Do not file anything the user did not select.** If nothing is approved, stop and say so — don't file a reduced set anyway.

## 6. Ensure the label exists

```bash
gh label list
```

If `ux` is not present:

```bash
gh label create "ux" --description "UI/UX design and usability improvements" --color "d4c5f9"
```

## 7. File approved issues

- One GitHub issue per approved recommendation — never batch several into one issue.
- Check the repo's actual issue conventions first (`gh issue list --state all --json body --limit 5`) rather than assuming a fixed template. A solid default shape if there's no strong existing convention: `## Description` (grounded in real current behavior), `## Proposed change` (the concrete fix), `## Acceptance criteria` (checklist).
- Match the project's commit/issue title convention if one is documented in CLAUDE.md (commonly `<type>: <capability gained>`, describing intent not implementation).
- Write each body to a file in the scratchpad directory and create with `gh issue create --body-file <path>` — multi-paragraph markdown passed through an inline heredoc is unreliable through the Bash tool.
- Apply the `ux` label to every issue as it's created.

## 8. Report

Summarize as a table: issue number, title, one-line rationale. Then explicitly list any proposed recommendations that were **not** approved, so they don't silently vanish — the user may want them filed later.

## What NOT to file

- Anything not explicitly approved by the user
- Findings that are really accessibility/WCAG issues in disguise — point the user at `a11y-audit` instead of double-labelling
- Pure cosmetic nitpicks (exact spacing, shade preference) with no usability impact, unless the user asked for a visual-polish pass specifically
- Anything already tracked by an open issue
- Problems not confirmed by reading the actual code or the running app

## Tone and style

- Speak as a UI/UX expert with an opinion, not a consultant hedging every sentence — lead with a recommendation, not an exhaustive survey of options
- Every criticism must be traceable to a specific file, line, or observed behavior — no generic "consider improving the visual hierarchy" filler
- Prefer reusing an existing in-codebase pattern over proposing a new bespoke one
- Never create GitHub issues without explicit, itemized user approval
