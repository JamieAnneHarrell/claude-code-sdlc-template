# `/product-visioning`

An interactive session that decides what to build next and writes the next PRD.

**Type:** reference · **Primary reader:** template adopter

## Synopsis

```
/product-visioning
/product-visioning --movement "<name>"
```

- `--movement "<name>"` — name the movement explicitly. Without it, the command
  proposes a name and confirms it with you.

## When to run

At a milestone — after an MVP ships, before a large feature, when you are
planning the next version. It also runs on a brand-new project to produce the
first PRD before [`/onboard`](onboard.md). A *movement* is a strategic chunk of
work opened this way; bug fixes and cleanup are tactical and need no PRD.

## What it does

A single interactive stage in five steps:

1. **Detect the target PRD** from the existing `docs/design/PRD-<slug>-*.md`
   files: refine an existing `DRAFT`, open the next movement after an `ACTIVE` or
   `SUPERSEDED` PRD, or start `PRD-<slug>-001` if none exist.
2. **Read context and sweep open work** — `PRODUCT_VISION.md`, the latest
   `ACTIVE` PRD, `PROJECT_PLAN.md`, `REQUIREMENTS.md`, `open-questions.md`,
   deferred items in test plans and design-review checkpoints — into one
   deduplicated inventory.
3. **Reconnaissance** — states back a short picture of where the product is and
   what you seem to want, and waits for your confirmation.
4. **The guided conversation** — works through what's next and why, scope
   (in / roadmap / out), personality and positioning, and near versus far,
   writing decisions straight into the PRD.
5. **Write the `DRAFT` PRD** and rewrite `TODO.txt` to point at it and at
   [`/onboard`](onboard.md) next.

The session is re-runnable: each run refines the same `DRAFT` PRD until you agree
it is done.

## Reads

`PRODUCT_VISION.md`, the latest `ACTIVE` PRD, `PROJECT_PLAN.md`,
`REQUIREMENTS.md`, `open-questions.md`, the `§5` deferred observations in
`docs/test-plans/phase-*-exit.md`, and prior design-review checkpoints.

## Writes / owns

Exactly one file: `docs/design/PRD-<slug>-NNN.md` with `status: DRAFT`. It also
rewrites the first `TODO.txt` entry. It does not own a `CLAUDE.md` status
comment — the PRD's own frontmatter (`DRAFT` / `ACTIVE` / `SUPERSEDED`) is the
source of truth.

## Refuses when

- **A `DRAFT` PRD exists at an unexpected number** — it confirms whether to keep
  refining it.
- **The PRD frontmatter `status:` is unexpected** — it asks how to recover.
- **The current movement's phases aren't all complete** — opening a new movement
  mid-flight is unusual, so it surfaces this and confirms.
- **The conversation drifts into tactical decomposition** (phases, prompts) — it
  stops, because that is [`/onboard`](onboard.md)'s job, and captures the intent
  in the PRD instead.

## Does not do

Decompose the PRD or edit `PRODUCT_VISION.md`, `PROJECT_PLAN.md`,
`CLAUDE_CODE_PROMPTS.md`, `REQUIREMENTS.md`, or `ARCHITECTURE.md` — all of that is
[`/onboard`](onboard.md)'s. It does not flip the PRD to `ACTIVE`, archive plans,
edit decision logs, run git, or run tests.

## See also

[`/onboard`](onboard.md) · [The SDLC lifecycle](../lifecycle.md) ·
[`/design-review`](design-review.md)
