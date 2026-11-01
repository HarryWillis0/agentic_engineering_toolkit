# agentic-engineering-toolkit

Skills, agents, and rules for agentic software engineering. Everything here was built and run in [Claude Code](https://claude.com/claude-code), so it lives under `claude/`. Other tools can sit alongside it.

```
claude/
├── CLAUDE.md          global rules: commit format, linear git history, coding style
├── settings.json      permission defaults
├── agents/            subagent definitions
└── skills/            one folder per skill, each with a SKILL.md
```

To use a skill, copy its folder into `~/.claude/skills/` (or a project's `.claude/skills/`) and invoke it as `/<name>`.

## Skills

### Planning

| Skill | Why it exists | Example |
|---|---|---|
| [`grill-me`](claude/skills/grill-me/SKILL.md) | Plans fail at their unexamined branches. This walks the decision tree one question at a time until nothing is left ambiguous. | `/grill-me` then paste a design |
| [`roadmap`](claude/skills/roadmap/SKILL.md) | Turns a product interview into GitHub milestones, so planning ends up in the tracker rather than in a doc. | `/roadmap` |
| [`to-prd`](claude/skills/to-prd/SKILL.md) | Captures a conversation's decisions as a PRD before they scroll away. | `/to-prd` |
| [`to-tasks`](claude/skills/to-tasks/SKILL.md) | Splits a PRD into thin vertical slices that can each be picked up independently. | `/to-tasks docs/prd.md` |

### Implementation

| Skill | Why it exists | Example |
|---|---|---|
| [`tdd`](claude/skills/tdd/SKILL.md) | Keeps the agent in a strict red-green-refactor loop instead of writing tests after the fact. | `/tdd add CSV export` |
| [`implement-milestone`](claude/skills/implement-milestone/SKILL.md) | Burns down a whole milestone: picks the next unblocked issue, builds it with TDD, opens a PR, repeats. | `/implement-milestone v1.0` |
| [`teach`](claude/skills/teach/SKILL.md) | Builds a feature step by step with tutorial-style explanations, for learning an unfamiliar stack. | `/teach add auth middleware` |
| [`commit`](claude/skills/commit/SKILL.md) | Derives the repo's commit conventions from its history, then commits and pushes to match. | `/commit` |

### Audits and backlog burn-down

Each audit files GitHub issues under a label. Its `implement-*` companion works through that label one branch at a time.

| Audit | Companion | Label | Why it exists |
|---|---|---|---|
| [`code-audit`](claude/skills/code-audit/SKILL.md) | [`implement-excellence`](claude/skills/implement-excellence/SKILL.md) | `software-excellence` | Finds clean-code violations. Fixes are pure refactors, verified for no behavior change. |
| [`a11y-audit`](claude/skills/a11y-audit/SKILL.md) | [`implement-a11y`](claude/skills/implement-a11y/SKILL.md) | `accessibility` | Finds accessibility gaps such as missing ARIA and keyboard traps. |
| [`dep-audit`](claude/skills/dep-audit/SKILL.md) | [`implement-dep`](claude/skills/implement-dep/SKILL.md) | `dependencies` | Finds CVEs, outdated packages, and unused ones, grouped into upgrade tracks. |
| [`ux-audit`](claude/skills/ux-audit/SKILL.md) | — | `ux` | Proposes usability recommendations and files only the ones you approve. |

Example: `/code-audit`, then `/implement-excellence`.

### Process

| Skill | Why it exists | Example |
|---|---|---|
| [`retro`](claude/skills/retro/SKILL.md) | Mines a session for corrections and friction, then proposes durable edits to skills and rules. | `/retro` |

## Agents

- [`task-implementer`](claude/agents/task-implementer.agent.md): works through a PRD and tasks folder, picks an unblocked task, asks for approval before starting and before marking it done, and implements with `tdd`.

## License

MIT
