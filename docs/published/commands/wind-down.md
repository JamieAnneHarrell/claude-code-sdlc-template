# `/wind-down`

Close out a session: rewrite `TODO.txt`, update the tracking docs, and hand you
the commit.

**Type:** reference · **Primary reader:** template adopter

## Synopsis

```
/wind-down
```

No flags or arguments.

## When to run

At the end of every session, before you step away. Run it explicitly, or it is
suggested when you signal you are done ("calling it", "off to bed", "that's
enough for today"). Other commands also call it at an artifact boundary — for
example [`/design-review`](design-review.md) and
[`/exit-test-plan`](exit-test-plan.md) at a landing. It is never run
autonomously; the suggestion waits for your go-ahead.

## What it does

A five-step ritual:

1. **Take stock** — recaps the session's goal, what landed, what is unfinished,
   and what was discovered, as a few bullets for you to confirm.
2. **Rewrite `TODO.txt`** — never appends. Completed items are removed (no
   strikethroughs); the first entry is the next session's pick-up. Built-in
   safety nets scan for an open design-review checkpoint, an open exit-test plan,
   or a `DRAFT` PRD and set the right first entry. When every phase is complete,
   it surfaces a menu (release / opportunistic work / next movement) without
   acting on it.
3. **Update tracking docs** — edits only docs with a real change:
   `CLAUDE_CODE_PROMPTS.md` deviation footers, `design-decisions.md`,
   `open-questions.md`, `REQUIREMENTS.md`, `ARCHITECTURE.md`, `PROJECT_PLAN.md`
   phase status, the `CLAUDE.md` banner, and the currency of
   `docs/published/` and the READMEs. It reports the status of files it does not
   own rather than editing them.
4. **Surface the commit handoff** — gives you the pre-commit sequence and a
   commit message (subject plus zero or one body sentence, body bullets citing
   doc IDs). It does not run the commit.
5. **Final report** — what was rewritten and updated, whether a commit is
   pending, and any open questions.

## Reads

`TODO.txt`, `PROJECT_PLAN.md`, the design-review checkpoints, the exit-test
plans, the PRDs, the tracking docs, the `CLAUDE.md` banner, and the build/run/test
commands in `rules/environment-rules.md`.

## Writes / owns

`TODO.txt` (rewritten), and the tracking docs in step 3 where they have a real
diff. The commit handoff is surfaced as commands for you to review and paste —
`/wind-down` owns the commit-handoff ritual but never runs it (rule 7).

## Refuses when

It does not run `git commit`, push to remotes, or edit files it does not own —
design-review checkpoints, test plans, PRDs, plan archives, `REVIEWS.md`, or the
`docs/published/` set and user-facing README sections (it routes drift there to
[`/write-documentation`](write-documentation.md)). It surfaces a warning when
`TODO.txt` doesn't reflect an open checkpoint or plan.

## Does not do

Run or amend commits, push, edit past-session history, skip steps, or reach into
another command's owned files.

## See also

[The SDLC lifecycle](../lifecycle.md) · [`/design-review`](design-review.md) ·
[`/exit-test-plan`](exit-test-plan.md)
