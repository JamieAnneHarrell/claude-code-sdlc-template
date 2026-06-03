# Multi-Agent Rules

> *This file is a placeholder. `/onboard` rewrites it with the
> project-specific multi-agent mode (one of: `explore-only`,
> `explore-plus-plan`, or `parallel-worktrees`) based on Jamie's
> answer during onboarding.*

Until onboarding runs, the conservative default applies:

- Use parallel `Explore` subagents only when codebase research will
  span >3 queries or multiple unrelated areas. Otherwise use `Glob` /
  `Grep` directly.
- Do **not** spawn implementation agents. All implementation is
  sequential and Jamie-driven.
- Do **not** use git worktrees.

<!-- CC-TEMPLATE-BLOCK: briefing-rule -->
When briefing any subagent, treat it like "a smart colleague who just
walked into the room" — it has not seen this conversation, doesn't
know what's been tried, and doesn't understand why this task matters.
Brief it accordingly:
- State the goal.
- Describe what's already been ruled out.
- Provide enough context for judgment calls.
- Cap response length if you only need a summary.

Never delegate understanding. Don't write "based on your findings, fix
the bug" — synthesis is the main thread's job.
<!-- /CC-TEMPLATE-BLOCK -->
