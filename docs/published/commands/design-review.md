# `/design-review`

Run a sign-off-ready design review at a high-risk transition. Two stages,
iterative across rounds.

**Type:** reference · **Primary reader:** template adopter

## Synopsis

```
/design-review
/design-review --new
/design-review --trigger "<reason>"
```

- `--new` — force a fresh checkpoint even when a prior one is still open. It
  warns that the prior checkpoint's findings will be left unapplied.
- `--trigger "<reason>"` — set the checkpoint's trigger string explicitly.
  Without it, the command infers the trigger from project state. The addendum
  branch inherits the trigger from the existing file.

The command auto-detects its stage and branch from the newest
`docs/design/design-review-checkpoint-*.md`.

## When to run

At every high-risk transition over the project's life, starting right after
[`/onboard`](onboard.md). The first review is a post-onboarding sanity check that
gates [`/bootstrap`](bootstrap.md).

## What it does

**Stage 1 — author findings.** Writes severity-tiered findings (Blockers,
Recommendations, Notes), one decision per finding. The *initial* branch writes a
new checkpoint when none is open; the *addendum* branch appends an
`## Addendum N` section of re-opened and new findings when you have asked for
another round. The checkpoint's frontmatter is `AWAITING-DECISIONS` for the whole
loop.

**Stage 2 — walk dispositions.** After you mark each finding, the command records
your decisions in the Disposition log and asks whether to **land** the checkpoint
or **open another round**. Landing applies every accumulated decision to
`REQUIREMENTS.md`, `ARCHITECTURE.md`, `PROJECT_PLAN.md`, and
`CLAUDE_CODE_PROMPTS.md`, flips the frontmatter to `LANDED <date>`, and hands off
to [`/wind-down`](wind-down.md).

You mark each finding's `AUDIT NOTE` block with one of five shapes: `Accepted`,
`Accepted with caveats: …`, `Defer Approved`, `DECISION: <choice>`, or
`REJECTED: <reframing prose>`. An unmarked finding still shows the placeholder
line, which is how the command tells marked from unmarked.

## Reads

`REQUIREMENTS.md`, `ARCHITECTURE.md`, `PROJECT_PLAN.md`, `CLAUDE_CODE_PROMPTS.md`,
the README, `DEPLOYMENT.md` if present, the design intake, and every prior
checkpoint.

## Writes / owns

`docs/design/design-review-checkpoint-NNN.md` and the
`docs/design/REVIEWS.md` index. On the land path only, it also edits
`REQUIREMENTS.md`, `ARCHITECTURE.md`, `PROJECT_PLAN.md`, and
`CLAUDE_CODE_PROMPTS.md`. Frontmatter values: `AWAITING-DECISIONS`,
`LANDED <date>`.

## Refuses when

- **`/onboard` hasn't run** — "Design review requires `/onboard` to have run
  first … Run `/onboard`, then come back."
- **The latest round has unmarked findings** — it lists which findings and which
  round.
- **An `AUDIT NOTE` line is malformed** (not one of the five shapes) — it asks
  rather than interpreting novel wording.
- **An addendum is requested but nothing needs re-opening** — it declines to
  author an empty addendum and points you back to the land choice.
- **A disposition implies editing a file outside the four targets** — it records
  a TODO instead of editing rules, `CLAUDE.md`, the README, or design intake.

## Does not do

Own a `CLAUDE.md` status comment, edit design-intake docs or rules, auto-mark
findings, auto-land a checkpoint, run git, or run tests. Its commit handoff fires
only at artifact boundaries (initial checkpoint, landing); mid-iteration rounds
hand off nothing.

## See also

[`/onboard`](onboard.md) · [`/exit-test-plan`](exit-test-plan.md) ·
[The SDLC lifecycle](../lifecycle.md) · [`/wind-down`](wind-down.md)
