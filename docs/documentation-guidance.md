# Documentation Guidance

Standing, project-level directives for how *this project's* documentation
should read — house voice, required structure, audience emphasis, recurring
do's and don'ts. Every entry is **binding doctrine**: `/write-documentation`
reads this file and applies all of it on every authoring, reconcile, and
revise pass.

This file is **durable-global and current-truth** (parallel to
`design-decisions.md` / `open-questions.md`): superseded guidance is *removed*,
not struck through — the notable *why* of a change becomes a
`design-decisions.md` entry. It is never archived per movement; a per-entry
`Scope note` handles the rare movement-bound case.

`/onboard` creates this file from the shipped skeleton. **`/wind-down` owns
capture and retirement** of entries (propose-and-confirm, reading this format
header first). `/write-documentation` **only reads** it — there are no
per-entry status markers (OPEN / APPLIED); guidance is current-truth and the
documentation sweep is idempotent.

**Strategic boundary — documentation is downstream of behavior.** An entry here
is a *doc directive* only: how existing, already-shipped behavior is described.
It never drives a behavior or workflow change (that routes through the
in-movement enhancement lane: plan → `/design-review` → decompose) and never
opens a movement (that runs through `/product-visioning` → PRD → `/onboard`).
This file never writes `docs/PRODUCT_VISION.md`.

## Format

Each entry uses the same shape so the file scans cleanly (mirrors
`design-decisions.md`).

```
## <Guidance title — the standing instruction, in one phrase>

**Guidance.** What to always do when authoring / reconciling / revising
(imperative, binding).

**Why.** The feedback or recurring problem that drove it.

**Scope note** (optional). Which audiences / docs / movement it applies to;
omit (or write "all docs") when unscoped.
```

---

<!-- Onboarding seeds the first entry here if the design intake stated how the
project's docs should read; otherwise this file stays header-only until
/wind-down captures the first directive. New entries go below in chronological
order; superseded ones are removed, not struck through. -->

## Lead with `/product-visioning` as the primary way to start a new project

**Guidance.** When documenting how to start a new project from the template,
present `/product-visioning` as the primary, recommended path and
dropping-a-design-doc-into-`docs/design/` as an equal-but-secondary supported
alternative. Don't frame the design-doc drop as the default starting move.

**Why.** Jamie's 2026-06-27 directive: `/product-visioning` is the first-class
entry point; the dropped-design-doc path remains fully supported but is no longer
the suggested default.

**Scope note.** The startup / seeding sections of the user-facing docs
(`docs/user/quick-start.md`, the `/onboard` + `/product-visioning` command
references) and both READMEs.
