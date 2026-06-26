# `/onboard`

Decompose a design intake into the project's planning documents. The first run
configures a new project; later runs open each new movement.

**Type:** reference · **Primary reader:** template adopter

## Synopsis

```
/onboard
```

No flags or arguments. The command auto-detects which of its two paths to run
from the `ONBOARD-STATUS` comment in `CLAUDE.md`.

## When to run

- **First PRD** — a new project where `/onboard` has never run
  (`ONBOARD-STATUS: UNCONFIGURED`). The intake is either a design doc you dropped
  into `docs/design/` or the first PRD that [`/product-visioning`](product-visioning.md)
  wrote.
- **Later movement** — an onboarded project (`ONBOARD-STATUS: COMPLETE`) that has
  a newer, undecomposed PRD whose movement number is higher than the current
  `PROJECT_PLAN.md`.

After either path, the next step is [`/design-review`](design-review.md).

## What it does

**First PRD.** Reads the design intake and writes the project's planning set:
`docs/REQUIREMENTS.md`, `docs/ARCHITECTURE.md`, `docs/PRODUCT_VISION.md`,
`docs/PROJECT_PLAN.md` (with a `movement` and `source-prd` header),
`docs/CLAUDE_CODE_PROMPTS.md`, and first entries in `docs/design-decisions.md`
and `docs/open-questions.md`. It fills the project-scope block in
`rules/project-rules.md`, rewrites `rules/multi-agent-rules.md` to your chosen
mode, writes a README stub, creates `.gitignore` and `LICENSE` if absent, adopts
the intake as `docs/design/PRD-<slug>-001.md` (`status: ACTIVE`), and flips
`ONBOARD-STATUS` to `COMPLETE <date>`.

**Later movement.** Applies the new PRD without re-running setup: applies the
PRD's `PRODUCT_VISION` revisions, archives the prior plan to
`docs/project-plans/project-plan-NNN.md` and `claude-code-prompts-NNN.md`,
authors a fresh `PROJECT_PLAN.md` and `CLAUDE_CODE_PROMPTS.md` for the new
movement, and flips the target PRD `DRAFT → ACTIVE` while marking the prior
`ACTIVE → SUPERSEDED <date>`.

## Reads

`CLAUDE.md` (for `ONBOARD-STATUS`), every file in `docs/design/` except its
`README.md`, and — for a later movement — `PRODUCT_VISION.md`, `REQUIREMENTS.md`,
`ARCHITECTURE.md`, and the in-flight `PROJECT_PLAN.md`.

## Writes / owns

The planning docs above, `PRODUCT_VISION.md`, the in-flight `PROJECT_PLAN.md` and
`CLAUDE_CODE_PROMPTS.md`, their `docs/project-plans/` archives, PRD status
transitions, and the `ONBOARD-STATUS` comment (values `UNCONFIGURED`,
`COMPLETE <date>`).

## Refuses when

- **No design doc in `docs/design/`** — drop one there, then re-run.
- **The intake is too thin** to derive requirements — it surfaces what's missing
  rather than inventing requirements.
- **A later-movement PRD has unresolved scope** — it points you to
  [`/product-visioning`](product-visioning.md) to finish the PRD.
- **The current plan still has unfinished phases** (later movement) — it
  confirms before archiving.
- **The movement is already decomposed** — "run `/product-visioning` to plan the
  next, or `/design-review` to review the current."
- **An existing file would be overwritten with non-template content** — it shows
  the difference and asks per file.

## Does not do

Plan the developer environment ([`/bootstrap`](bootstrap.md)), plan deployment
([`/deployment-plan`](deployment-plan.md)), author the PRD body
([`/product-visioning`](product-visioning.md)), run a design review
([`/design-review`](design-review.md)), write test plans
([`/exit-test-plan`](exit-test-plan.md)), or run git or tests.

## See also

[The SDLC lifecycle](../lifecycle.md) · [`/product-visioning`](product-visioning.md) ·
[`/design-review`](design-review.md) · [`/bootstrap`](bootstrap.md)
