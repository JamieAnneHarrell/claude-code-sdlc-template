# `/exit-test-plan`

Author and close out a manual phase-exit test plan. Two stages, iterative across
polish rounds.

**Type:** reference · **Primary reader:** template adopter

## Synopsis

```
/exit-test-plan
/exit-test-plan N
/exit-test-plan --new N
```

- `N` — force a specific phase number (for example `/exit-test-plan 3`). Without
  it, the command infers the target phase from `PROJECT_PLAN.md`.
- `--new N` — force a fresh plan for phase `N` even when a prior one is still
  open. It warns that the prior plan will be left unfinished.

## When to run

At the close of a phase whose exit criteria warrant a manual walkthrough.
[`/onboard`](onboard.md) must be `COMPLETE`, and [`/bootstrap`](bootstrap.md)
should usually be `COMPLETE` too, since a manual walkthrough generally needs a
working dev environment. The command runs the *planning*; you run the walkthrough
(rule 8).

## What it does

**Stage 1 — author the plan.** The *initial* branch writes
`docs/test-plans/phase-NNN-exit.md` with test cases (Steps / Expected / Fail
signals), a `§4` run log seeded with `[PENDING]` placeholders, and an empty `§5`.
The *addendum* branch appends a `§6.N` polish addendum after a round is filled
and dispositioned. Frontmatter stays `AWAITING-DISPOSITIONS` for the whole loop.

**Stage 2 — land dispositions.** Once you have filled the newest run log, the
command walks each note, each `[BLOCKED]` row, and each out-of-scope observation
with you, proposes a disposition (`Fix-now`, `Defer to Deferred User Stories`,
`Defer to Known Limitations`, `Spec gap`, or `Note-only`), appends rows to `§5`,
mirrors decisions to `docs/design-decisions.md`, `docs/open-questions.md`, and
`TODO.txt`, and asks whether to flip to `LANDED <date>` or open another polish
round.

## Reads

`PROJECT_PLAN.md` (the phase's exit criteria), `CLAUDE_CODE_PROMPTS.md`,
`REQUIREMENTS.md`, `ARCHITECTURE.md`, the implementation files the criteria
touch, the README's Developer setup section, and prior test plans.

## Writes / owns

`docs/test-plans/phase-NNN-exit.md` (creating `docs/test-plans/` as needed). On
the land path it also edits `docs/design-decisions.md`,
`docs/open-questions.md`, and `TODO.txt`. Frontmatter values:
`AWAITING-DISPOSITIONS`, `LANDED <date>`. The `[PENDING]` placeholder marks an
unfilled run-log row.

## Refuses when

- **`/onboard` hasn't run, or `PROJECT_PLAN.md` is missing** — it points at
  [`/onboard`](onboard.md).
- **A run log still has `[PENDING]` rows** — it lists them and reminds you this is
  a rule-8 walkthrough you run.
- **A `[BLOCKED]` row has no blocker description** — it asks you to add a
  one-sentence description keyed by the test-case ID, then re-run.
- **An addendum is requested but no Fix-now item is ready and no `[BLOCKED]` test
  is queued** — it declines to author an empty addendum.
- **A disposition implies editing `REQUIREMENTS` / `ARCHITECTURE` /
  `PROJECT_PLAN` / `CLAUDE_CODE_PROMPTS` / rules / `CLAUDE.md` / README** — it
  records a TODO; spec gaps promote through [`/design-review`](design-review.md).

## Does not do

Run the test plan (you do — rule 8), invent new project affordances to make a
step easier (rule 4), edit the internal planning docs beyond its disposition
scope, auto-land a plan, run git, or run tests.

## See also

[`/design-review`](design-review.md) · [The SDLC lifecycle](../lifecycle.md) ·
[`/wind-down`](wind-down.md)
