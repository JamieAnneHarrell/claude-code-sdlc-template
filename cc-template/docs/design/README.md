# Design intake

Drop your design doc(s) into this folder before running `/onboard`.

`/onboard` reads every file in this directory (other than this
`README.md`) as the source-of-truth design intake for the project.
The richer the design doc, the better the generated REQUIREMENTS,
ARCHITECTURE, and PROJECT_PLAN will be.

## What a good design doc looks like

The example Jamie has been working with (a multi-tenant niche-site
platform) covered:

- **Concept** — one-paragraph summary of what the product is
- **Architectural premise** — layers, dependencies, MVP scope
- **Stack** — language, framework, key libraries (with rationale)
- **Data model** — table-by-table or class-by-class, key columns
- **Runtime / data flow** — how requests, jobs, and data move through
  the system
- **Phased plan** — what ships in MVP, what's roadmap, what's
  explicitly NOT in scope
- **Runtime / deployment intent** — where this is meant to run
  (local-only, single VPS, cloud, container) and any topology
  intent (dev→prod, dev→test→prod, etc.). `/bootstrap` and
  `/deployment-plan` ask tailored questions based on what's here;
  if you don't know yet, leave it blank and the relevant command
  will offer to defer.
- **Open questions** — things deferred or not yet decided

A design doc that's just a paragraph of vibes will produce a thin
PROJECT_PLAN. A design doc that walks through MVP scope, data model,
and phase boundaries produces a plan Jamie can actually start
building from in the next session.

## What `/onboard` extracts

- **REQUIREMENTS.md** — numbered FRs and NFRs lifted from explicit
  requirements language in the design.
- **ARCHITECTURE.md** — the architectural narrative: layers, modules,
  data flow, key technical decisions.
- **PROJECT_PLAN.md** — phased plan derived from the design's phasing
  (or inferred from the architecture if no phases are stated).
- **CLAUDE_CODE_PROMPTS.md** — one prompt per phase, ready to paste.
- **`design-decisions.md`** seed entry — the language / tooling /
  multi-agent-mode choices made during onboarding.

## Multiple design files

If the design evolves across multiple revisions or splits into
separate concerns (e.g. a `data-model.md` and a `runtime.md`), drop
all of them here. `/onboard` reads them all and synthesizes.

If two files conflict, `/onboard` will surface the conflict and ask
Jamie which to follow.

## Format

Markdown is preferred. Plain text and `.docx` work too. PDF works for
small documents but is harder to extract from cleanly — convert to
markdown if you can.

## After onboarding

This folder stays around as historical record. Don't delete the
design intake after onboarding completes — future sessions may want
to reference what the original intent was vs. how the project
actually evolved.
