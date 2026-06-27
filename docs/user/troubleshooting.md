# Troubleshooting

Two kinds of trouble show up when working with the template: a command **refuses**
to run, or the workflow quietly **drifts** out of shape over time. This page
covers both. Each refusal entry is stated as Symptom → Cause → Fix.

## When a command refuses

The commands refuse explicitly and point you at the fix rather than guessing. That
is a feature — a refusal protects a downstream document from being built on a
missing prerequisite.

### "Bootstrap requires `/onboard` to have run first"

- **Symptom:** [`/bootstrap`](commands/bootstrap.md) stops immediately with a
  pointer to `/onboard`.
- **Cause:** `ONBOARD-STATUS` in `CLAUDE.md` is still `UNCONFIGURED` — the project
  hasn't been onboarded, so there's no plan or environment context to bootstrap
  against.
- **Fix:** Run [`/onboard`](commands/onboard.md) (and usually
  [`/design-review`](commands/design-review.md)) first, then re-run
  `/bootstrap`. The same shape applies to [`/deployment-plan`](commands/deployment-plan.md),
  which requires bootstrap to be `COMPLETE`.

### A re-run did the wrong stage

- **Symptom:** You re-ran [`/design-review`](commands/design-review.md) or
  [`/exit-test-plan`](commands/exit-test-plan.md) expecting it to advance, and it
  re-authored or refused instead.
- **Cause:** These commands auto-detect their stage from the artifact's state. A
  design review advances to Stage 2 only when every finding in the latest round is
  marked; a test plan advances only when the run log has no `[PENDING]` rows left.
  An unmarked finding or a pending row holds it at the earlier stage.
- **Fix:** Finish marking the findings (replace each placeholder line with a
  decision) or fill every run-log row, then re-run. The command tells you exactly
  which findings or rows are still outstanding.

### "A proposed doc is left unmarked"

- **Symptom:** [`/write-documentation`](commands/write-documentation.md) refuses
  to author and lists unmarked docs.
- **Cause:** At least one `DOC DECISION` block in the documentation plan still
  holds its placeholder line, so the set is only partially approved.
- **Fix:** Mark every block `approve`, `adjust: <note>`, or `drop`, then re-run.

### Refresh keeps asking about a block you already own

- **Symptom:** [`/refresh-from-repository`](commands/refresh-from-repository.md)
  asks about the same edited rule block on every run.
- **Cause:** The block is still `template-owned`, so each refresh sees your edit as
  a divergence. Choosing "keep mine" is what records it as yours.
- **Fix:** When asked, choose **keep mine** — that marks the block `forked` and
  refresh stops asking. See [Keeping the template up to date](keeping-up-to-date.md).

### A refresh offered to overwrite a rule you edited

- **Symptom:** You changed a rule and a refresh proposed replacing it with the
  upstream version.
- **Cause:** Refresh never silently overwrites — it surfaces the divergence and
  asks. This is the ask-once moment, not a bug.
- **Fix:** Choose **keep mine** to fork the block, **hand-merge** to combine, or
  **take upstream** to accept the new version. If you would rather a block never be
  considered, move it out of its `CC-TEMPLATE-BLOCK` markers.

## Common drifts

These don't announce themselves with an error. They accumulate quietly, and they
erode the value the lifecycle is supposed to protect. Watch for them and correct
course.

### The prompt log records what was done, not what changed

- **Drift:** `CLAUDE_CODE_PROMPTS.md` deviation footers fill up with a narrative of
  what happened in a phase.
- **Why it hurts:** The footer's job is to record *deviations* from the prompt —
  where the build diverged from the plan — so the next reader sees the delta at a
  glance. A play-by-play buries that signal.
- **Correct course:** Keep footers to what changed against the prompt. The
  session narrative belongs in the commit and the wind-down recap, not the prompt
  log.

### `TODO.txt` becomes a log or a backlog

- **Drift:** `TODO.txt` grows a history of completed work, or fills with user
  stories and ideas that aren't actually the next thing.
- **Why it hurts:** `TODO.txt` is a session handoff — its first entry is what you
  pick up next. When it becomes a log or a backlog, the handoff is buried and the
  next session starts by sifting.
- **Correct course:** Let [`/wind-down`](commands/wind-down.md) rewrite it each
  session: completed items removed, first entry the next pick-up. Non-blocking
  ideas belong in `docs/open-questions.md` or a future PRD, not the handoff.

### Re-discovering the environment every session

- **Drift:** Each session re-derives how to build, run, or test the project,
  because that knowledge never landed anywhere durable.
- **Why it hurts:** It wastes the start of every session and invites
  inconsistency.
- **Correct course:** Capture it once where it belongs — the README "Developer
  setup" section and the environment block in `rules/environment-rules.md`, both
  written by [`/bootstrap`](commands/bootstrap.md). If you keep rediscovering a
  command, that's a sign bootstrap's output is incomplete; re-run it.

### Decisions made in chat that never reach a doc

- **Drift:** A real decision gets made in conversation and never lands in
  `design-decisions.md`, so it's re-litigated later.
- **Why it hurts:** The reasoning evaporates between sessions, and the same
  question comes back.
- **Correct course:** Let `/wind-down` move resolved questions into
  `design-decisions.md` and unresolved ones into `open-questions.md`. If a decision
  reverses a signed-off design choice, route it through
  [`/design-review`](commands/design-review.md) rather than editing in place.
