# `/bootstrap`

Plan the developer environment and write a working README "Developer setup"
section — then validate it.

**Type:** reference · **Primary reader:** template adopter

## Synopsis

```
/bootstrap
```

No flags or arguments. The command runs in one of two stages, chosen from the
`BOOTSTRAP-STATUS` comment in `CLAUDE.md`.

## When to run

After [`/onboard`](onboard.md) is `COMPLETE`, and typically after the first
[`/design-review`](design-review.md). `/bootstrap` is a hard prerequisite to
writing code (Phase 0). Run it twice: once to write the setup instructions, once
to validate them.

## What it does

**Stage 1 — write instructions** (`BOOTSTRAP-STATUS: UNCONFIGURED`). Reads the
planning docs and design intake, asks stack-tailored questions (shell and
editor, stack bootstrap, tooling beyond the language, dev secrets and config),
and writes the README "Developer setup" section — Prerequisites, Verify your
setup, Bootstrap, Run locally, Test, and Secrets and config — plus the
environment block in `rules/environment-rules.md`. It then flips
`BOOTSTRAP-STATUS` to `INSTRUCTIONS-WRITTEN <date>`.

**Stage 2 — validate** (`BOOTSTRAP-STATUS: INSTRUCTIONS-WRITTEN`). Runs each
command under "Verify your setup" in your chosen shell and captures the exit
codes. On success it flips `BOOTSTRAP-STATUS` to `COMPLETE <date>`. On failure it
changes nothing and reports each failed check with its command, exit code, and a
pointer to Prerequisites.

Both stages flag any version before it lands in a file (rule 10) and offer to
confirm the current stable or LTS release.

## Reads

`CLAUDE.md` (`ONBOARD-STATUS`, `BOOTSTRAP-STATUS`), `REQUIREMENTS.md`,
`ARCHITECTURE.md`, `PROJECT_PLAN.md` (especially Phase 0), the design intake,
`rules/environment-rules.md`, and the existing README when validating.

## Writes / owns

The README "Developer setup" section, the `<!-- ONBOARD-FILL: environment -->`
block in `rules/environment-rules.md`, and the `BOOTSTRAP-STATUS` comment (values
`UNCONFIGURED`, `INSTRUCTIONS-WRITTEN <date>`, `COMPLETE <date>`).

## Refuses when

- **`ONBOARD-STATUS` is `UNCONFIGURED`** — "Bootstrap requires `/onboard` to have
  run first. Run `/onboard`, then come back."
- **`BOOTSTRAP-STATUS` is `COMPLETE`** — it asks before re-running, since
  re-running overwrites the setup section and downgrades the status.
- **The design doesn't pin a stack** — it asks you directly rather than inventing
  one; if the stack genuinely isn't settled, "bootstrapping is premature."
- **The bootstrap steps wouldn't work on a fresh clone** — it pushes back and
  names what's missing rather than writing a broken setup section.
- **The README setup section or the environment markers were hand-edited** — it
  shows the difference and asks before touching them.
- **Stage 2 finds a tool missing or the wrong version** — it reports the failed
  checks and does not flip the status.

A missing or `AWAITING-DECISIONS` design-review checkpoint is a soft reminder,
not a refusal.

## Does not do

Plan deployment ([`/deployment-plan`](deployment-plan.md)), install dependencies
or create environments (it documents how; Phase 0 runs them), write CI
workflows, run git, or run tests. Stage 2 only checks whether tools are on PATH.

## See also

[`/onboard`](onboard.md) · [`/deployment-plan`](deployment-plan.md) ·
[Quick-start](../quick-start.md)
