---
checkpoint: 005
date: 2026-06-24
reviewer: Claude (Opus 4.8) with Jamie
status: LANDED 2026-06-25
trigger: Phase 2.3 Block 1 gate — validate /write-documentation invariant ownership and resolve the render architecture (as-built in-manifest/in-session model is provisional) before Block 2 sibling wiring and the dogfoods
---

# Design Review — Checkpoint 005

## Context

Phase 2.3's core skill, [`/write-documentation`](../../cc-template/.claude/commands/write-documentation.md),
is built, along with its source-only design-doc edits (FR-14, ARCHITECTURE,
root `CLAUDE.md` invariants, `design-decisions.md`). This is the light
invariant-ownership checkpoint the build plan calls for (construction sequence
step 2) before Block 2 (sibling wiring: `/deployment-plan`, `/wind-down`,
`/onboard`), the `/refresh-from-repository` source-mode dogfood, and the
`/write-documentation` dogfood that closes the phase.

The review has one load-bearing decision and two housekeeping items. The
decision is **render/build ownership**: as first built, `/write-documentation`
owns rendering as an in-session `--render` action whose recipe lives in the
documentation-plan manifest, with `/deployment-plan` *referencing* that recipe.
That model is encoded — provisionally, pending this review — in five places:
the command file (§ Render configuration, Step S2.5, the `--render` arg),
[`ARCHITECTURE.md`](../ARCHITECTURE.md) (§ manifest shape L255–262, § render
note L354–357), [`REQUIREMENTS.md`](../REQUIREMENTS.md) FR-14 (L168–169),
[`design-decisions.md`](../design-decisions.md) (§ render-architecture-under-review
L1256–1263), and the root `CLAUDE.md` `/write-documentation` invariant. The
2026-06-24 deferred story in [`open-questions.md`](../open-questions.md)
(§ Deferred User Stories) flagged the model and is the seed for B1; this
checkpoint greenfields the question rather than ratifying that story's proposal.

No prior checkpoint is load-bearing here — 001–004 are all `LANDED` and concern
onboarding and `/refresh-from-repository`; this is the first checkpoint to touch
`/write-documentation`. The hard constraint the render findings are judged
against: **NFR-6 / runtime neutrality** — `cc-template` ships runtime-free and
prescribes no runtime downstream; a freshly-seeded project has no runtime, and
nothing the template emits may assume one. This repo's own Python exists only to
maintain this repo's docs (a maintainer-environment artifact), not as part of
the distribution.

## How to mark up this review

For each finding below, replace the placeholder line under
`AUDIT NOTE — JAH:` with one of:

- `Accepted`
- `Accepted with caveats: <your caveats>`
- `Defer Approved`
- `DECISION: <explicit choice>`
- `REJECTED: <your reframing prose>` — none of the listed
  recommendations is satisfactory. This opens another addendum
  round; your rejection prose is the authoritative input the next
  addendum reframes the finding from.

Each finding has its own AUDIT NOTE block — mark each independently.
Inline detail, caveats, and rationale are welcome alongside the decision
keyword (the keyword is what Stage 2 keys off; the surrounding text is the
audit trail).

Then re-run `/design-review`. Stage 2 will walk each marked disposition with
you, append rows to the Disposition log, and ask whether to **land the
document** or **open another addendum round**.

**Do not fill the Sign-off Summary table at the bottom** — Stage 2 fills that
when the doc lands.

## Findings

### Blockers — fix before Block 2 sibling wiring

#### B1. Render/build ownership — what does `/write-documentation` own toward a delivered doc, and who renders?

**Sources.** [`write-documentation.md`](../../cc-template/.claude/commands/write-documentation.md)
§ Render configuration (manifest template L407–413), Step S2.5 "Render" (L539–553),
`--render` argument (L69–70), "single source of truth" claim (L550–553);
[`ARCHITECTURE.md`](../ARCHITECTURE.md) L255–262 + L354–357;
[`REQUIREMENTS.md`](../REQUIREMENTS.md) FR-14 L168–169, NFR-6 L270–273, NFR-8 L283+;
[`design-decisions.md`](../design-decisions.md) L1256–1263; root `CLAUDE.md`
§ Load-bearing invariants → `/write-documentation` block (render single-owned);
[`open-questions.md`](../open-questions.md) § Deferred User Stories (2026-06-24
render story); the ds-zip-tax POC
[`scripts/build-docs.py`](../../../ds-zip-tax-for-odoo/scripts/build-docs.py)
(single-target PDF, authored by `/deployment-plan` in Phase 7).

**Problem.** As built, `/write-documentation` is a *writing* skill carrying a
*renderer's* responsibilities: it records a render recipe inside the doc-content
manifest, exposes an in-session `--render`, and claims to be the single source of
truth for how the project renders, which `/deployment-plan` merely references.
Three things break under scrutiny. (1) **Wrong owner.** Polished, rendered docs
are part of a *deliverable*, and the template already has a deliverable/build
owner — `/deployment-plan` — which builds distributions "like the rest of the
project." A writing skill owning the render splits the build concern across two
commands and muddies NFR-8. (2) **Can't serve the real path.** A render whose
output ships in a release must run at release time in CI / a build step, and CI
cannot invoke a slash command — so an in-session `--render` does not serve the
release path it was designed for. (3) **Runtime tension.** Recording a render
toolchain (or scaffolding a render *script*) inside the template pushes toward
prescribing a runtime, which NFR-6 forbids; a "runtime-neutral script" is close
to self-contradictory. The as-built in-session model is the *do-nothing* option
and is explicitly **not** the KISS choice — it is the most coupled one.

Greenfield, the real question is a clean ownership cut: **what minimal thing does
`/write-documentation` hand toward delivery, and where does the render/build
live?** Options, first-level:

- **Option A — Recipe-only handoff (writing here, build at `/deployment-plan`).**
  `/write-documentation` owns the markdown sources, a **target-agnostic,
  product-type-aware *delivery recipe*** in its manifest (what well-delivered
  docs look like *for this kind of product* — e.g. SaaS → in-app / HTML help;
  downloadable zip → README + quick-start; library → API reference site — without
  naming a renderer or toolchain), and the release-readiness ledger. It owns **no**
  render execution, render script, `--render`, or toolchain. `/deployment-plan`
  owns building the deliverable: it reads the recipe + ledger and **codes the
  actual render targets against whatever runtime exists at build time** (Claude
  knows the environment then). This is the **rich-but-KISS** option — rich because
  the recipe is product-type-aware and feeds a real release gate; KISS because
  `/write-documentation` emits only markdown + a recipe + a ledger, prescribes no
  runtime, and the ownership cut is clean. It is also the most *true to this
  project's opinionated methods*: rendering becomes a session coding task done
  when needed, not pre-baked machinery — consistent with "AI is session work, not
  a heuristic engine."

- **Option B — Scaffold a committed multi-target build artifact (the
  open-questions story).** `/write-documentation` emits a committed,
  parameterized-by-target render script that the release process runs; `--render`,
  if kept, is a local-preview wrapper over the same script. Keeps the render
  *definition* with `/write-documentation`. Tension: it still makes a writing
  skill responsible for a build artifact, and a template-shipped script in *some*
  language soft-prescribes a runtime (NFR-6) — the runtime-neutrality bar is hard
  to clear with a concrete scaffold.

- **Option C — Markdown-only floor (no delivery opinion).** `/write-documentation`
  authors markdown + the ledger and says nothing about delivery shape;
  `/deployment-plan` owns every delivery decision unaided. The true minimum.
  Loses the product-type → delivery-shape guidance that is a genuine value-add and
  the reason a *writing* skill is well-placed to recommend delivery form.

(An option we are *not* carrying forward: keeping render execution inside
`/write-documentation` at all — the as-built in-session `--render`. It is folded
into the problem statement above as the rejected do-nothing baseline. A
downstream sibling possibility, surfaced for awareness only: the render
*toolchain* could be `/bootstrap`/environment territory since it is a per-project
tool, with `/deployment-plan` invoking it — relevant only if A or B lands and is
a Block 2 detail, not a first-level choice here.)

**Recommendation.** Adopt **Option A**. Reshape the manifest's "Render
configuration" section into a target-agnostic, product-type-aware **"Delivery
recipe"**; remove the `--render` argument and Step S2.5 from
`/write-documentation`; flip the root `CLAUDE.md` invariant from "render
single-owned by `/write-documentation`" to "`/deployment-plan` owns doc
render/build, consuming `/write-documentation`'s delivery recipe + ledger"; and
update FR-14 / ARCHITECTURE / `design-decisions.md` to the recipe-handoff model.
Downstream implications, named not specced (per first-level-primary; raise as a
Block 2 design item or next addendum if load-bearing): Block 2's
`/deployment-plan` work grows from "*reference* the render" to "**own** the
doc-build capability" (a larger sibling edit than the build plan assumed); the
release-readiness ledger becomes the primary `/write-documentation` →
`/deployment-plan` handoff signal and should be sharpened as such; and the
"runtime-neutral render" question dissolves (the template prescribes nothing; the
build is coded at deploy time).

> AUDIT NOTE — JAH:
> _Accepted with caveats: We must be tolerant of an unknown product type; the skill, when run, will know more about the product type but may need to research realtime and/or have conversational discovery with the developer rather than prescriptive of only specific product types._

### Recommendations — resolve before or during Block 2

#### R1. Phase 2.3 tracking docs still describe an unbuilt, differently-named, spec-deferred command

**Sources.** [`PROJECT_PLAN.md`](../PROJECT_PLAN.md) Phase 2.3 (L468–487);
[`CLAUDE_CODE_PROMPTS.md`](../CLAUDE_CODE_PROMPTS.md) Prompt 2.3 (L689–728).

**Problem.** Both still call the command `/write-user-documentation`, state the
spec is "deferred to Phase 2.3 design session," and Prompt 2.3's scope asks the
coding session to decide the command's shape (inputs, single-vs-staged, outputs)
from scratch — all stale. The command is built as `/write-documentation`, its
design is settled and its doctrine embedded, and neither doc reflects the rename,
the Block 1 (core skill, done) / Block 2 (sibling wiring) / dogfood / close
staging, or the render decision pending in B1.

**Recommendation.** Update Phase 2.3 and Prompt 2.3 to the built name
`/write-documentation`; mark the spec landed, pointing at the shipped command
file and this checkpoint rather than re-deriving shape; record the Block 1 /
Block 2 / dogfood / close staging; and align the exit criteria with B1's render
decision. (Both files are in Stage 2's land-path edit scope.)

> AUDIT NOTE — JAH:
> _DECISION: Already run prompts and plans don't get altered. They get a deviation block, eg., "changes since this prompt ran". However, we do want to update forward looking references and all other documents like design decisions, and future prompts and plans to maintain coherence._

### Notes — acceptable now, revisit later

#### N1. The open-questions render story has a factual error and will need rewriting to the landed decision

**Sources.** [`open-questions.md`](../open-questions.md) § Deferred User Stories
(2026-06-24 render story); ds-zip-tax
[`scripts/build-docs.py`](../../../ds-zip-tax-for-odoo/scripts/build-docs.py).

**Problem.** The story claims ds-zip-tax "hand-wrote a render-to-HTML script
alongside its `pandoc --pdf-engine=typst` PDF build." It did not — `build-docs.py`
is single-target PDF only, authored by `/deployment-plan` in Phase 7 (before
`/write-documentation` existed, which is exactly why that POC conflates writing
and build). The multi-target need (HTML for online tutorials, PDF in a zip,
others) is real and Jamie-raised but is not demonstrated by the POC, and the
cited evidence overstates it.

**Recommendation.** Correct the false HTML-script claim, and once B1 lands,
migrate the story to its resolved home (`design-decisions.md` if a decision, or
rewrite to the chosen architecture). Because `open-questions.md` is outside
Stage 2's edit scope, this surfaces as a TODO at landing, not a landed edit.

> AUDIT NOTE — JAH:
> _Approved. Also, I think the assertion that this project already has a runtime (python to build docs) is actually false as well. I said that in error. The sibling project ds-zip-tax-for-odoo has the runtime that was established before this command existed, and I assumed this project would inherit such a runtime at which time this command is finalized._

## What the design got right (preserve)

- **README / CONTRIBUTING section-ownership model** — `/write-documentation`
  owns user-facing sections by offer; `/bootstrap` (Developer setup) and
  `/deployment-plan` (Deployment) keep theirs; `/wind-down` routes rather than
  clobbers. Extends the existing ownership model cleanly; unaffected by B1.
- **Movement-aware currency tuple** — `documented-through { movement, phase }`,
  reusing `/onboard`'s movement detection read-only, with phase numbers compared
  only within a movement. Solves the per-movement phase-reset trap correctly.
- **Numbered manifest, newest-governs**, `AWAITING-APPROVAL → ACTIVE →
  SUPERSEDED` — consistent with the PRD / checkpoint idioms already in the
  template; per-file frontmatter as source of truth, no CLAUDE.md status comment.
- **Release-readiness ledger** as a signal a release process consumes (not runs)
  — survives B1 and, under Option A, becomes the central handoff to
  `/deployment-plan`'s release gate.
- **Conformance-gap discipline** — surface doc/code discrepancies, never edit
  product code (rule 8 / NFR-8). Correct boundary.
- **The embedded documentation-craft doctrine** (Diátaxis modes, per-audience
  voice profiles, CLI-reference / readability / anti-pattern rules) — the
  load-bearing content of the skill; untouched by this review.

## Next steps

1. Jamie marks each finding inline (AUDIT NOTE blocks above).
2. Re-run `/design-review`. Stage 2 walks each disposition, appends rows to the
   Disposition log, and asks whether to land the doc or open another addendum
   round.
3. After landing, Block 2 (sibling wiring) proceeds under the render-ownership
   decision from B1, followed by the `/refresh-from-repository` source-mode
   dogfood and the `/write-documentation` dogfood.

## Disposition log

*Stage 2 fills this section. Empty until Stage 2's first walk. Rows accumulate
append-only across rounds — later-round dispositions on the same finding ID
supersede earlier ones for the purposes of Stage 2's landing application.*

| Round | Finding | Disposition | Reason |
|-------|---------|-------------|--------|
| Original | B1 | Accepted with Caveats | Option A — recipe handoff; build/render owned by `/deployment-plan`. Caveat: the delivery recipe stays product-type-**open** — the skill researches the product type in real time and/or discovers it conversationally with the developer, never limited to a fixed type enum. |
| Original | R1 | Decision | Already-run prompts/plans are not rewritten — they get a deviation block ("changes since this prompt ran"). Forward-looking references and other docs (design-decisions, future prompts/plans) get coherence updates. |
| Original | N1 | Accepted | Correct the false ds-zip-tax HTML-script claim and migrate the story once B1 lands (TODO — `open-questions.md` out of Stage 2 scope). Added correction: this project has **no** Python runtime (that is ds-zip-tax's); the checkpoint Context's contrary line and the same assumption in the story are errors to fix. |

## Sign-off Summary

*Filled when Stage 2 lands the doc. Jamie does not edit this section — sign-offs
go inline above.*

| ID | Final disposition |
|----|-------------------|
| B1 | Accepted with Caveats — Option A (writing + product-type-open delivery recipe here; render/build owned by `/deployment-plan`) |
| R1 | Decision — already-run prompts/plans get deviation notes, not rewrites; forward-looking docs + new prompts written for coherence |
| N1 | Accepted — correct the open-questions render story (false HTML-script claim + false runtime assumption) and migrate it |

## Follow-up actions landed

*Lists every doc edit and every forward-looking prompt that landed during the
landing pass.*

- **B1** — edited `docs/REQUIREMENTS.md` FR-14 and `docs/ARCHITECTURE.md`
  (manifest shape + "No language runtime" note) to the recipe-handoff model:
  `/write-documentation` writes markdown + a delivery recipe + the ledger and
  owns no renderer; `/deployment-plan` owns the build.
- **B1** — the application work that falls outside `/design-review`'s edit
  scope (the command file, the root `CLAUDE.md` invariant, `design-decisions.md`,
  `open-questions.md`) was authored as **Prompt 2.4 / Phase 2.4** rather than
  left as loose TODOs — reshape the command, flip the invariant, rewrite the
  design-decisions note, correct the open-questions story.
- **R1** — `docs/PROJECT_PLAN.md` Phase 2.3 gained a `Status (2026-06-25)`
  deviation note and `docs/CLAUDE_CODE_PROMPTS.md` Prompt 2.3 a revision-footer
  line (bodies preserved); authored **Phases/Prompts 2.5, 2.6, 2.7** (ecosystem
  wiring → refresh dogfood → write-doc dogfood + close) so the remaining
  write-documentation work is a clean, runnable prompt chain.
- **N1** — the `open-questions.md` corrections (false HTML-script claim; false
  "this project has a Python runtime" assumption, which also appears in this
  checkpoint's Context as a point-in-time error) are folded into Prompt 2.4,
  since `open-questions.md` is outside Stage 2's edit scope.
