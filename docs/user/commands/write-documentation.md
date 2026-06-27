# `/write-documentation`

Author and maintain audience-facing product documentation. Two stages,
auto-detected; currency is movement-aware.

**Type:** reference · **Primary reader:** template adopter

## Synopsis

```
/write-documentation
/write-documentation "<describe a change>"
/write-documentation --doc "<name>"
/write-documentation --audience "<role>"
/write-documentation --new
```

- `"<describe a change>"` — a Stage 2 *revise* pass: an interactive,
  natural-language change to the living docs.
- `--doc "<name>"` — scope a Stage 2 pass to one document.
- `--audience "<role>"` — scope the proposed set to one audience.
- `--new` — force a fresh plan (Stage 1) even when the current one is current.

The command auto-detects its stage from the newest
`docs/documentation-plans/documentation-plan-*.md`.

## When to run

When the audience-facing docs need writing or refreshing — for end users,
administrators, support staff, developers, or contributors. This is distinct from
the internal planning docs: those say *what we're building and why*; this writes
*how to install, use, operate, and extend it*.

## What it does

**Stage 1 — survey and plan.** Reads the intent docs *and* the real
implementation, infers the product type and audiences, and writes a numbered
manifest (`documentation-plan-NNN.md`, `status: AWAITING-APPROVAL`) proposing a
documentation set — one `DOC DECISION` block per doc for you to mark `approve`,
`adjust: <note>`, or `drop`. It authors no prose yet; the sign-off gate comes
first.

**Stage 2 — author, reconcile, or revise.** Three non-gated modes. In *author*
mode it writes the marked set into audience-named folders (`docs/<audience-slug>/`
— `docs/user/`, `docs/maintainer/`, and so on), applying a documentation-craft
doctrine (Diátaxis modes, per-audience voice, CLI-reference and readability
rules), fills the release-readiness ledger, stamps `documented-through`, and
flips the manifest to `ACTIVE`. In *reconcile* mode it refreshes the existing set
against what shipped since the last stamp, without a new sign-off. In *revise*
mode it takes a natural-language change you describe, proposes how it lands across
the affected docs, and applies it after you agree. Every mode applies the standing
directives in `docs/documentation-guidance.md` as binding doctrine; documentation
is downstream of behavior, so revise reflects what already shipped — it never
drives a behavior change.

Currency is **movement-aware**: the manifest records `documented-through:
{ movement, phase }`, and a re-run is stale when the active movement differs or a
later phase shipped within the same movement. There is no `LANDED` — currency is
derived.

## Reads

The planning docs, the active PRD and `PRODUCT_VISION.md`, `DEPLOYMENT.md`,
`docs/documentation-guidance.md` (binding doctrine), prior checkpoints and test
plans, the real implementation, and any existing manifest, README, and
`CONTRIBUTING.md`.

## Writes / owns

The manifest at `docs/documentation-plans/documentation-plan-NNN.md`, and the
doc sources and `images/` in their audience folders (`docs/<audience-slug>/`).
The user-facing sections of the README (by offer), and `CONTRIBUTING.md` when the
project takes contributors. It writes a product-type-aware **delivery recipe** but
owns no renderer — building the delivered docs is
[`/deployment-plan`](deployment-plan.md)'s. Manifest status values:
`AWAITING-APPROVAL`, `ACTIVE`, `SUPERSEDED <date>`.

## Refuses when

- **There's nothing built to document** — "the command documents a product; there
  must be something built to document."
- **A proposed doc is left unmarked** — it lists the unmarked docs and the legend
  rather than authoring a partially-approved set.
- **A marking is malformed** (not `approve` / `adjust:` / `drop`) — it asks.
- **A doc can't be made correct without a product change** (a wrong or missing
  `--help`, prose that can't agree with the real CLI) — it records a
  **conformance gap** in the ledger and surfaces a coding TODO; it never edits the
  product (rule 8).
- **A version needs pinning** — it flags it as possibly-stale and asks (rule 10).

## Does not do

Own a `CLAUDE.md` status comment, edit product source code or the internal
planning docs, write `docs/documentation-guidance.md` (it reads and applies that
file; [`/wind-down`](wind-down.md) owns capture and retirement), edit
`docs/PRODUCT_VISION.md` or open a movement, edit `DEPLOYMENT.md`, clobber the
README's Developer setup or Deployment sections, render or build the docs, run a
release, run git, or run tests.

## See also

[`/deployment-plan`](deployment-plan.md) · [`/wind-down`](wind-down.md) ·
[The SDLC lifecycle](../lifecycle.md)
