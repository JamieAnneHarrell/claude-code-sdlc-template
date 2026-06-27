# Command reference

Every slash command the template ships, one document each. This page routes you
to the right one and shows the order they run in.

These are *reference* pages — exact and consulted, not read front to back. For
the narrative of how the commands fit together over a project's life, read
[The SDLC lifecycle](../lifecycle.md). To get a first project running, follow
the [Quick-start](../quick-start.md).

## Which command, when

| Command | When you run it | What it produces |
|---|---|---|
| [`/onboard`](onboard.md) | First, once per project — and again to open each later movement | The planning docs (`REQUIREMENTS`, `ARCHITECTURE`, `PROJECT_PLAN`, …) |
| [`/design-review`](design-review.md) | At every high-risk transition, starting right after onboarding | A sign-off checkpoint with dispositioned findings |
| [`/bootstrap`](bootstrap.md) | After the first design review, before you write code | The README "Developer setup" section, validated |
| [`/deployment-plan`](deployment-plan.md) | When you need to plan how the project ships (deferrable) | `docs/DEPLOYMENT.md` and the README "Deployment" section |
| [`/product-visioning`](product-visioning.md) | At a milestone, to decide what to build next | The next PRD (`docs/design/PRD-<slug>-NNN.md`) |
| [`/exit-test-plan`](exit-test-plan.md) | At a phase exit that needs a manual walkthrough | `docs/test-plans/phase-NNN-exit.md` |
| [`/write-documentation`](write-documentation.md) | When the audience-facing docs need writing or refreshing | A documentation plan plus the doc sources |
| [`/wind-down`](wind-down.md) | At the end of every session | A rewritten `TODO.txt` and a commit handoff |
| [`/refresh-from-repository`](refresh-from-repository.md) | When you want later template improvements | Updated commands and block-merged rules |

## Two groups

**Configuration commands run once.** `/onboard`, `/bootstrap`, and
`/deployment-plan` set a project up. Each tracks its own status in `CLAUDE.md`
and refuses to run before its prerequisite is met.

**Recurring commands run for the life of the project.**
`/product-visioning`, `/design-review`, `/exit-test-plan`,
`/write-documentation`, and `/wind-down` repeat as the project moves through
movements and phases. `/refresh-from-repository` pulls upstream template
improvements whenever you choose.

## A note on rule 7 and rule 8

No command runs `git commit`, `git push`, or `git tag`, and no command runs your
automated tests. Commands that reach a commit boundary hand the commit to you
through `/wind-down`; commands that need a test run give you the commands and the
expected result and wait. This is deliberate — you stay the author of every
commit and every test run.
