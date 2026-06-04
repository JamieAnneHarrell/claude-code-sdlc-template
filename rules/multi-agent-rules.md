# Multi-Agent Rules

This project uses the **explore-plus-plan** mode: parallel
`Explore` subagents for research, plus a `Plan` subagent for design
proposals. Implementation stays sequential — no worktree spawning,
no parallel implementation agents.

Chosen during `/onboard` because the project has load-bearing
invariant complexity in `CLAUDE.md` (status comment vocab, marker
blocks, zero-pad filename conventions, stage-detection placeholder
strings) and iterative two-stage recurring-command lifecycles
(`/design-review`, `/exit-test-plan`). Plan-subagent validation
before implementation is worth the round-trip on changes touching
those invariants.

---

## Explore — use for

- Codebase research spanning >3 queries or multiple unrelated
  areas. *Example: "Find every consumer of `ONBOARD-STATUS`
  across the six command files and rules files."*
- Open-ended "where is X?" / "which files reference Y?" when
  you'd otherwise iterate via Glob + Grep.
- Cross-referencing checks before a load-bearing edit. *Example:
  "Before renaming the AUDIT NOTE placeholder, confirm every file
  that parses it literally."*

## Explore — don't use for

- A single Glob or Grep with known shape — call the tool directly.
- Code review, design-doc auditing, or open-ended analysis
  requiring whole-file context. Explore reads excerpts; it misses
  content past its read window.
- Cross-file consistency checks. Use Plan.

## Plan — use for

- Changes touching more than one command file or more than one
  load-bearing invariant. *Example: "Design how
  `/refresh-from-repository` interacts with `ONBOARD-FILL`
  markers across both rules files."*
- Architectural decisions where trade-offs are non-obvious.
  *Example: "Should the new BLOCKED disposition be per-row or
  per-document?"*
- Pre-flight before non-trivial implementation — catches design
  problems earlier than implementation does.

## Plan — don't use for

- One-file changes with an obvious path. Just do the edit.
- Brainstorming or open-ended creative questions. Plan designs a
  specific change, not ideas.

## Not enabled in this project

- **No implementation agents.** All implementation is sequential
  and Jamie-driven. Don't spawn an agent to make code edits.
- **No git worktrees.** Single-module markdown repo; coordination
  overhead would dominate any throughput gain.

---

## Briefing rule (every subagent invocation)

Treat every subagent like **a smart colleague who just walked
into the room** — it has not seen this conversation, doesn't know
what's already been tried, and doesn't understand why this task
matters.

Brief it accordingly:

- State the goal.
- Describe what's already been ruled out.
- Provide enough context for judgment calls.
- Cap response length if you only need a summary.

Never delegate understanding. Don't write "based on your findings, fix
the bug" — synthesis is the main thread's job.
