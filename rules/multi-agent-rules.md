# Multi-Agent Rules

This project uses the **explore-plus-plan** multi-agent mode:
parallel `Explore` subagents for research, plus a `Plan` subagent
for design proposals. Implementation stays sequential — no
worktree spawning, no parallel implementation agents.

The mode was chosen during `/onboard` because the project has
load-bearing invariant complexity in `CLAUDE.md` (status comment
vocabulary, marker blocks, zero-pad filename conventions,
stage-detection placeholder strings) and recurring-command
lifecycles (`/design-review` and `/exit-test-plan` are both
two-stage and iterative). Validating an approach with a Plan
subagent before implementing is worth the round-trip on changes
that touch those invariants.

---

## Use Explore for

- Codebase research that will span >3 queries or multiple
  unrelated areas. *Example: "Find every consumer of the
  `ONBOARD-STATUS` comment across the six command files and the
  rules files."*
- Open-ended "where is X?" / "which files reference Y?"
  questions where you'd otherwise iterate via Glob + Grep.
- Cross-referencing checks before a load-bearing edit. *Example:
  "Before renaming the AUDIT NOTE placeholder, confirm every file
  that parses it literally."*

## Don't use Explore for

- A single Glob or Grep that you already know the shape of —
  call the tool directly.
- Code review, design-doc auditing, or any open-ended analysis
  that needs the reader to hold the whole file in their head.
  Explore reads excerpts; it misses content past its read window.
- Cross-file consistency checks. Use `Plan` for those.

## Use Plan for

- Designing a change that touches more than one command file or
  more than one load-bearing invariant. *Example: "Design how
  `/refresh-from-repository --merge` interacts with the
  `ONBOARD-FILL` markers across both rules files."*
- Architectural decisions where the trade-offs are not obvious.
  *Example: "Should the new BLOCKED disposition be a per-row
  marker or a per-document status?"*
- Pre-flight check before a non-trivial implementation. The
  round-trip catches "this design has a problem I missed" earlier
  than the implementation does.

## Don't use Plan for

- One-file changes with an obvious implementation path. Just do
  the edit.
- Brainstorming or open-ended creative questions. Plan is for
  designing a specific change, not generating ideas.

## What's NOT enabled in this project

- **No implementation agents.** All implementation is sequential
  and Jamie-driven. Do not spawn an agent to make code edits on
  your behalf.
- **No git worktrees.** No `parallel-worktrees` mode for this
  project. The project is a single-module markdown repo;
  worktree coordination overhead would dominate any throughput
  gain.

---

## Briefing rule (every subagent invocation)

Treat every subagent like **a smart colleague who just walked
into the room** — it has not seen this conversation, doesn't know
what's already been tried, and doesn't understand why this task
matters.

Brief it accordingly:

- **State the goal.** Not the immediate next step — the goal that
  step is in service of.
- **Describe what's already been ruled out.** Failed approaches,
  rejected designs, decisions Jamie has already made.
- **Provide enough context for judgment calls.** File paths,
  invariant names, the load-bearing constraints the subagent
  could trip over without knowing they exist.
- **Cap response length if you only need a summary.** Subagent
  output that lands in the main context costs tokens forever.
- **Never delegate understanding.** Don't write "based on your
  findings, fix the bug" or "based on the research, implement
  the change." Synthesis is the main thread's job; the subagent
  produces evidence the main thread reasons over.

The briefing rule is embedded here (not just referenced from
global `~/.claude/CLAUDE.md`) so it survives global-file edits.
