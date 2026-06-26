---
checkpoint: 006
date: 2026-06-26
reviewer: Claude (Opus 4.8) with Jamie
status: LANDED 2026-06-26
trigger: Ad-hoc — /write-documentation reshape (audience-named doc folders, manifest relocation, durable documentation-guidance store + interactive revise mode); plus one unrelated plan revision
---

# Design Review — Checkpoint 006

## Context

`/write-documentation` shipped (Phase 2.3) and produced this project's first
documentation set under `docs/published/` (manifest `documentation-plan-001`,
`ACTIVE`). Dogfooding it surfaced three rough edges in the *shape* of what it
writes and how it iterates — not in the render/build ownership that
[checkpoint 005](design-review-checkpoint-005.md) settled (that boundary is
preserved untouched here). This checkpoint reviews a reshape of the skill's
output layout and iteration model, decomposes it into the current movement's
plan, and tacks on one unrelated plan revision Jamie asked for.

The design input is the approved plan at
`C:\Users\jamie\.claude\plans\our-new-skill-for-ticklish-pnueli.md`, settled
across an extended design conversation this session. The findings below carry
that settled direction as their Recommendations; Jamie signs each off
independently.

Docs read: `REQUIREMENTS.md` (FR-14), `ARCHITECTURE.md` (file inventory L88–89,
filename conventions L184–206, manifest-shape L255–266, directory tree L31–60),
`PROJECT_PLAN.md` (through Phase 2.7 + Phase 3), `CLAUDE_CODE_PROMPTS.md`
(Prompt 2.7, Prompt 3), and checkpoint 005. Because this is **cleanup within
the current `initial` movement** — not a new movement — it routes through
`/design-review` (whose Stage 2 land path writes REQUIREMENTS / ARCHITECTURE /
PROJECT_PLAN / CLAUDE_CODE_PROMPTS) rather than `/product-visioning` →
`/onboard`. Root `CLAUDE.md` invariants, the four command files, the live-doc
migration, and `design-decisions.md` / `open-questions.md` are touched by the
reshape but are **outside Stage 2's edit scope** — they become the phase work
this checkpoint lands into the plan (finding R4).

This review unblocks the implementation phases (B1–B3 + R1–R4); R5 is an
unrelated, self-contained plan revision.

## How to mark up this review

For each finding below, replace the placeholder line under
`AUDIT NOTE — JAH:` with one of:

- `Accepted`
- `Accepted with caveats: <your caveats>`
- `Defer Approved`
- `DECISION: <explicit choice>`
- `REJECTED: <your reframing prose>` — none of the listed recommendations is
  satisfactory. This opens another addendum round; your rejection prose is the
  authoritative input the next addendum reframes the finding from.

Each finding has its own AUDIT NOTE block — mark each independently. Inline
detail, caveats, and rationale are welcome alongside the decision keyword (the
keyword is what Stage 2 keys off; the surrounding text is the audit trail).

Then re-run `/design-review`. Stage 2 will walk each marked disposition with
you, append rows to the Disposition log, and ask whether to **land the
document** or **open another addendum round**.

**Do not fill the Sign-off Summary table at the bottom** — Stage 2 fills that
when the doc lands.

## Findings

### Blockers — settle before implementing the reshape

#### B1. Replace `docs/published/` with audience-named doc folders

**Sources.** `ARCHITECTURE.md` L88–89 (file inventory — "`docs/published/` …
`/write-documentation` sources, images, and documentation-plan manifest"), L50
(directory tree), L255–263 (manifest shape — "Markdown sources + `images/` live
alongside under `docs/published/`"); `REQUIREMENTS.md` FR-14 L162–165; root
`CLAUDE.md` `/write-documentation` lifecycle + File-ownership blocks; approved
plan § "Audience-named folders."

**Problem.** `docs/published/` reads as "packaged for distribution," but the
skill *writes* docs — it doesn't package them. The name is also incoherent with
the repo's other doc folders, which are named by purpose (`docs/design/`,
`docs/test-plans/`) or audience, and the single bucket gives a multi-audience
product no filing signal. This was the first dogfood rough edge.

**Recommendation.** Replace the single `docs/published/` bucket with
audience-named folders directly under `docs/`. Each doc's recorded Primary
audience slugifies to `docs/<audience-slug>/<file>.md` — six canonical slugs
(`end-user`, `power-user`, `admin`, `support`, `developer`, `maintainer`) or the
project's own label slugified. The `commands/` reference nests under its
audience (`docs/user/commands/`); images live per-folder
(`docs/<audience-slug>/images/`). **No `docs/guides/` escape hatch** — every doc
takes a Primary audience (filing is not a voice decision, so the
average-across-audiences anti-pattern does not apply); the only audience-neutral
artifact is the routing README, which is the README's job. For this project:
`docs/user/` + `docs/maintainer/`. On land: revise FR-14 paths and the
ARCHITECTURE file inventory + directory tree + manifest-shape source paths; the
command-file and live-doc changes are phase work (R4).

> AUDIT NOTE — JAH:
> _Accepted._

#### B2. Relocate the documentation-plan manifest out of the doc folder

**Sources.** `ARCHITECTURE.md` L195–197 (filename conventions —
"`docs/published/documentation-plan-NNN.md`"), L255 (manifest shape);
`REQUIREMENTS.md` FR-14 L161–162; approved plan § "Manifest relocation."

**Problem.** `documentation-plan-NNN.md` sits inside `docs/published/` alongside
the actual docs, mixing the planning artifact with its subject in one directory.
The repo already isolates numbered planning artifacts from their subjects —
`docs/test-plans/phase-NNN-exit.md`, `docs/project-plans/` — and the manifest
should follow that precedent.

**Recommendation.** Move the manifest to
`docs/documentation-plans/documentation-plan-NNN.md`, mirroring `docs/test-plans/`
/ `docs/project-plans/`. Counter, frontmatter, status lifecycle, and zero-pad-3
are unchanged. On land: update the manifest path in FR-14 and the ARCHITECTURE
filename-conventions + manifest-shape + file-inventory entries; the Step-0 glob
in `write-documentation.md` and the consumer globs in `deployment-plan` /
`wind-down` / `onboard` are phase work (R4).

> AUDIT NOTE — JAH:
> _[Accepted._

#### B3. Add a durable documentation-guidance store

**Sources.** `REQUIREMENTS.md` NFR-8 (file-ownership non-overlap), FR-14, FR-17
(product-visioning ownership); the `design-decisions.md` / `open-questions.md`
durable-global analog; `onboard.md` skeleton-seeding of those two; approved plan
§ "The documentation-guidance store."

**Problem.** After Stage 2 the skill cannot retain a durable, project-level doc
directive that survives across future documentation plans, nor propagate a
feedback theme across docs already written. Thematic and strategic doc feedback
(e.g. "every how-to opens with when/why before the steps") has no home — it gets
re-litigated or copy-pasted into each new numbered manifest. Third dogfood rough
edge.

**Recommendation.** Add `docs/documentation-guidance.md` — a durable-global,
no-frontmatter, append-then-reconcile file parallel to `design-decisions.md` /
`open-questions.md`. It is **current-truth** (like `open-questions.md`):
superseded guidance is removed, and the notable *why* of a change becomes a
`design-decisions.md` entry — not a forever-log. Ownership (preserving NFR-8):
**`/onboard` creates it** from a template-shipped skeleton (as it does the other
two durable-global files); **`/wind-down` captures + retires** entries
(propose-and-confirm, read-format-first); **`/write-documentation` only reads**
it as binding doctrine and applies it. No per-entry OPEN/APPLIED status markers
— they would force `/write-documentation` to write a file `/wind-down` owns, and
are unneeded once guidance is current-truth and the sweep is idempotent. On
land: add FR-18 (the store + its ownership) and an ARCHITECTURE entry; the
skeleton file, the `onboard` seeding step, and the `wind-down` capture step are
phase work (R4).

> AUDIT NOTE — JAH:
> _Accepted._

### Recommendations — resolve before or during implementation

#### R1. Three Stage-2 modes, with an interactive "revise" mode

**Sources.** `REQUIREMENTS.md` FR-14 L163–164 ("authors or reconciles");
`write-documentation.md` Stage 2; approved plan § "Stage 2 has three non-gated
modes."

**Problem.** Stage 2 has author + reconcile today, but no path for a user-driven
thematic/strategic revision of living docs, and no statement of who owns the
docs after Stage 2 or how to iterate. The living-docs revision loop is
undescribed.

**Recommendation.** Give Stage 2 three non-gated modes — **author** (the
approved set), **reconcile** (mechanical doc-rot refresh on same-movement phase
advance), and **revise** (an interactive coding-session-in-prose: you describe
the change, the skill proposes how it lands across affected docs, applies it; no
markup, placeholders, or sign-off artifact). Like any coding session, revise
stamps the manifest's authoring log (a "Revise N" round); the durable record is
captured at `/wind-down`. Stage 1's plan-set sign-off (the DOC DECISION markup
on the manifest) is unchanged — it scopes the *set*, not doc content. Add an
"After Stage 2 — who owns the docs and how to iterate" section to the command.
On land: note this in FR-14; command-file changes are phase work (R4).

> AUDIT NOTE — JAH:
> _Accepted._

#### R2. Route strategic doc-feedback to `/product-visioning`

**Sources.** `REQUIREMENTS.md` FR-17 (product-visioning owns PRDs; never edits
`PRODUCT_VISION` elsewhere), NFR-8; rule 9 (invoke the owning skill); approved
plan § "Strategic boundary."

**Problem.** Some doc feedback is also a product/workflow decision (e.g. "make
`/product-visioning` the default entry path; demote drop-a-design-doc to a
footnote"). If the guidance store recorded such decisions directly it would
become a backdoor for product-positioning changes that should flow through the
movement machinery, and risks cross-writing `PRODUCT_VISION.md`.

**Recommendation.** When doc feedback is also a product/workflow decision, route
it to `/product-visioning` (the owning skill); the `documentation-guidance.md`
entry carries only the *doc directive* and cross-references the upstream
decision. The store never writes `PRODUCT_VISION.md`. On land: capture this
boundary in FR-18 / the ARCHITECTURE guidance-store entry; command wording is
phase work (R4).

> AUDIT NOTE — JAH:
> _Rejected. product-visioning is exclusively for planning the next movement and writing a new PRD. The example provided (demote drop-a-design-doc and promote product-visioning as the entry point to a new project with this template) was a natural product progression / feature addition that occurred mid-stream in a movement. That can happen within a current movement, much like this is happening, even if it's acknowledged as feature-creap, within a movement._

#### R3. Document the file-lifecycle patterns as a design-decision

**Sources.** `ARCHITECTURE.md` L184–206 (filename conventions / movement
counter), L78–95 (source-only content); the three lifecycle behaviors already
running in the repo; approved plan § "File-lifecycle pattern, documented."

**Problem.** The repo runs three distinct file-lifecycle patterns but nowhere
names them, so "where does this kind of file go, and does it archive on a new
movement?" has no written answer — the exact question that arose when siting the
guidance store (B3).

**Recommendation.** Record a `design-decisions.md` entry naming the three
patterns: **durable-global, reconciled in place** (`design-decisions.md`
forever; `open-questions.md` + `documentation-guidance.md` current-truth) —
never archived; **movement-scoped, archived on movement open** (`PROJECT_PLAN.md`
/ `CLAUDE_CODE_PROMPTS.md` → `docs/project-plans/`); **numbered artifacts** (PRDs
/ checkpoints / test-plans / doc-plans — newest/per-key governs, priors
`SUPERSEDED`/`LANDED`). Add a one-line ARCHITECTURE pointer (the design-decision
holds the *why*). On land: add the ARCHITECTURE pointer; the `design-decisions.md`
entry itself is outside Stage 2 scope → captured at `/wind-down` during the
phase work (R4).

> AUDIT NOTE — JAH:
> _Accepted with Caveats: this implies current documentation must be marked out of date after this update, and we must do a write-documentation reconciliation._

#### R4. Decompose the reshape into phases + prompts; fold in Phase 2.7

**Sources.** `PROJECT_PLAN.md` Phase 2.7 (L580–594) + Phase 2 roadmap;
`CLAUDE_CODE_PROMPTS.md` Prompt 2.7 (L919–950); `REQUIREMENTS.md` NFR-9 / FR-11
(edit-in-`cc-template/` discipline) + the Phase 2.6 source-mode refresh
precedent; approved plan § "Execution."

**Problem.** This reshape is cleanup within the current `initial` movement, so it
needs new steps + prompts appended to the current plan, not a `/product-visioning`
→ `/onboard` decomposition. And Phase 2.7's pending NFR-6 dry-read targets
`docs/published/` — a layout this reshape replaces — so validating the old
layout first is wasted.

**Recommendation.** Append three phases that follow the cc-template-edit →
`/refresh-from-repository` (source mode) → root-doc-edit → migration discipline:

- **Phase 2.8 — Reshape the skill spec (`cc-template/`).** Edit
  `write-documentation.md` (Step-0 glob, paths, audience-folder rule,
  guidance-store subsection, three Stage-2 modes incl. revise, after-Stage-2
  iteration section), `deployment-plan.md` / `wind-down.md` (guidance
  capture+retire) / `onboard.md` (guidance skeleton seeding); add the
  `cc-template/docs/documentation-guidance.md` skeleton.
- **Phase 2.9 — Propagate to root + root invariants.** `/refresh-from-repository`
  (source mode) lands the four skill files at root; then edit root `CLAUDE.md`
  invariants (the `/write-documentation` block, Cross-cutting filename block,
  File-ownership list).
- **Phase 2.10 — Migrate live docs + close.** `git mv` `docs/published/` →
  `docs/user/` + `docs/maintainer/` + `docs/documentation-plans/`; create
  `docs/documentation-guidance.md`; re-stamp `documentation-plan-001`; fix
  relative + root-README links; update `design-decisions.md` +
  `open-questions.md`; run the NFR-6 dry-read against the new layout (absorbing
  Phase 2.7's pending validation) and close.

Mark Phase 2.7's dogfood deliverables done (the docs were produced) and move its
NFR-6 dry-read + close into Phase 2.10. On land: write these phases into
PROJECT_PLAN, author the matching prompts in CLAUDE_CODE_PROMPTS, and update
Phase 2.7's status note. (Phase numbering and prompt bodies are authored at land;
this finding fixes the first-level decomposition, not the prompt text.)

> AUDIT NOTE — JAH:
> _Accepted with caveats: This adds new phase *steps* not a new phase. Phases are top level numnbers; phase 1, phase 2, phase 3, etc. Steps within the phase are the dot-numbers. Step 2.7, step 2.8, etc. Record this terminology as well, which should trigger a write-documentation reconciliation._

#### R5. [Unrelated to the reshape] Declassify Phase 3 (regression-test automation) into a user story

**Sources.** `PROJECT_PLAN.md` Phase 3 (L598–617) + Out of scope (L621–628);
`CLAUDE_CODE_PROMPTS.md` Prompt 3 (L954+); `REQUIREMENTS.md` FR-16 (L194–199);
Jamie's direction this session.

**Problem.** Phase 3 (regression-test automation) is roadmap scope that does not
belong to this movement and is not yet motivated by a real regression — FR-16
itself says "Specifics deferred until a real regression motivates the work."
Jamie wants it pulled out of the active plan and reclassified later. This is
unrelated to the `/write-documentation` reshape; it rides along only because the
plan + prompts are already being reworked.

**Recommendation.** Remove Phase 3 from `PROJECT_PLAN.md` and Prompt 3 from
`CLAUDE_CODE_PROMPTS.md` (both land via Stage 2), and relax FR-16's "Phase 3
deliverable" framing in REQUIREMENTS so it reads as an unscheduled candidate
rather than a planned phase (lands). Surface a TODO to add it to
`docs/open-questions.md` as a Deferred User Story ("Regression-test automation —
diff `cc-template/` against a tagged baseline; classify into a movement when a
real regression motivates it"), since `open-questions.md` is outside Stage 2's
edit scope.

> AUDIT NOTE — JAH:
> _Accepted._

### Notes — acceptable now, revisit later

#### N1. The migrated command-docs will be content-stale after the reshape

**Sources.** `docs/published/commands/*` (10 per-command docs migrating to
`docs/user/commands/`); approved plan D6 follow-up note.

**Problem.** The migrated per-command docs describe `/write-documentation` (and
its siblings) as they are *before* the reshape — once B1–B3 + R1 land, those
docs are content-stale, not just relocated.

**Recommendation.** Leave them as a relocate-and-fix-links job in Phase 2.10; do
not hand-rewrite their content during the migration. Their content refresh is a
natural **first dogfood of the new `revise` mode** in a later
`/write-documentation` run — exercise the very capability the reshape adds.

> AUDIT NOTE — JAH:
> _Accepted - same notes I had above about requiring document reconcilation._

#### N2. Deferred user story — document the file-lifecycle patterns for users

**Sources.** R3 (the design-decision records the patterns for *maintainers*);
approved plan D7; `docs/open-questions.md` Deferred User Stories.

**Problem.** R3 records the file-lifecycle taxonomy as internal rationale
(`design-decisions.md`), but the patterns are also user-facing (a consumer needs
to know which of their files archive on a movement). That user-facing write-up
is out of scope for this movement.

**Recommendation.** Queue a Deferred User Story in `docs/open-questions.md` —
*Document the file-lifecycle patterns (durable-global / movement-archived /
numbered-artifact) in the user-facing SDLC docs* — for a future
`/write-documentation` pass to pick up. No action this movement beyond recording
the story (phase work, R4 / `/wind-down`).

> AUDIT NOTE — JAH:
> _Accepted._

## What the design got right (preserve)

- The **render/build ownership boundary** from checkpoint 005
  (`/write-documentation` writes markdown + delivery recipe; `/deployment-plan`
  builds) — preserved untouched; no finding re-opens it.
- The **numbered-manifest + derived movement-aware currency** model — only its
  *location* moves (B2), not its mechanics.
- The **DOC DECISION Stage-1 plan-set sign-off** — kept as the one-time scoping
  gate (R1 changes only how *revisions* work, not initial planning).
- The **skeleton-seeded durable-global doc pattern** (`design-decisions.md` /
  `open-questions.md`) — extended for the guidance store (B3), not reinvented.

## Next steps

1. Jamie marks each finding inline (AUDIT NOTE blocks above).
2. Re-run `/design-review`. Stage 2 walks each disposition, appends rows to the
   Disposition log, and asks whether to land the doc or open another addendum
   round.
3. After landing, the reshape's Phases 2.8–2.10 (and the unrelated R5 edits) are
   in the plan; the next session picks up Phase 2.8's prompt.

## Disposition log

*Stage 2 fills this section. Empty until Stage 2's first walk. Rows accumulate
append-only across rounds — later-round dispositions on the same finding ID
supersede earlier ones for the purposes of Stage 2's landing application.*

| Round | Finding | Disposition | Reason |
|-------|---------|-------------|--------|
| Original | B1 | Accepted | Audience-named doc folders (`docs/user/`, `docs/maintainer/`); no `docs/guides/`. |
| Original | B2 | Accepted | Relocate manifest to `docs/documentation-plans/`. (Marking read as Accepted; stray leading `[` noted.) |
| Original | B3 | Accepted | Add `docs/documentation-guidance.md` (current-truth; `/onboard` creates, `/wind-down` captures, `/write-documentation` reads). |
| Original | R1 | Accepted | Three Stage-2 modes incl. interactive revise; Stage-1 plan-set sign-off unchanged. |
| Original | R2 | REJECTED | `/product-visioning` is exclusively for planning the next movement / writing a new PRD. A mid-movement feature addition (e.g. promote product-visioning as the new-project entry point) is an in-movement enhancement, not a product-visioning trigger. Re-author the strategic boundary to route via the in-movement lane. → Addendum 1 (R2-A1). |
| Original | R3 | Accepted with Caveats | Record the file-lifecycle taxonomy. Caveat: this update marks current docs out of date → requires a `/write-documentation` reconciliation. |
| Original | R4 | Accepted with Caveats | Decompose the reshape. Caveat: these are *steps* (2.8/2.9/2.10) within Phase 2, NOT new phases — phases = top-level integers, steps = dot-numbers. Record the phase/step terminology → triggers a `/write-documentation` reconciliation. |
| Original | R5 | Accepted | Declassify Phase 3 / Prompt 3; relax FR-16; queue as a Deferred User Story. |
| Original | N1 | Accepted | Migrated command-docs go content-stale → refresh via the new revise mode (recurring reconciliation directive). |
| Original | N2 | Accepted | Queue Deferred User Story: document the file-lifecycle patterns for users. |
| Addendum 1 | R2-A1 | Decision | Documentation is downstream of behavior: doc-feedback → `/write-documentation` revise mode + guidance store (reflects landed behavior); a real behavior change redirects to the in-movement lane and docs reconcile *after* it lands; documentation never opens a movement and never writes `PRODUCT_VISION.md`. Supersedes R2's Original REJECTED. |

## Sign-off Summary

*Jamie does not edit this section — sign-offs go inline above.*

| ID | Final disposition |
|----|-------------------|
| B1 | Accepted — audience-named doc folders (`docs/user/`, `docs/maintainer/`); no `docs/guides/`. |
| B2 | Accepted — manifest relocated to `docs/documentation-plans/`. |
| B3 | Accepted — durable documentation-guidance store (`/onboard` creates, `/wind-down` captures, `/write-documentation` reads). |
| R1 | Accepted — three Stage-2 modes incl. interactive revise; Stage-1 plan-set sign-off unchanged. |
| R2 / R2-A1 | Decision — documentation is downstream of behavior: doc-feedback → revise + guidance; behavior → in-movement lane (docs reconcile after it lands); never opens a movement or writes `PRODUCT_VISION.md`. (Supersedes R2's REJECTED.) |
| R3 | Accepted with Caveats — file-lifecycle taxonomy recorded; requires a `/write-documentation` reconciliation. |
| R4 | Accepted with Caveats — decompose into Steps 2.8/2.9/2.10 (steps not new phases; terminology recorded; triggers a reconciliation). |
| R5 | Accepted — Phase 3 / Prompt 3 declassified; FR-16 relaxed. |
| N1 | Accepted — migrated command-docs content-stale → revise-mode refresh in Step 2.10. |
| N2 | Accepted — Deferred User Story queued: file-lifecycle patterns for users. |

## Follow-up actions landed

**Doc edits landed this pass (the four Stage-2 target docs):**
- B1/B2/R1 → `docs/REQUIREMENTS.md` FR-14: repathed to audience folders +
  `docs/documentation-plans/`; added the three Stage-2 modes incl. revise.
- B3/R2-A1 → `docs/REQUIREMENTS.md`: new **FR-18** (documentation-guidance store
  + downstream-of-behavior strategic boundary).
- R5 → `docs/REQUIREMENTS.md` FR-16 relaxed to an unscheduled candidate.
- B1/B2/B3/R3/R4 → `docs/ARCHITECTURE.md`: file inventory + directory tree +
  filename conventions + manifest-shape + a guidance-store paragraph + a new
  **File-lifecycle patterns** subsection + the phase/step terminology note.
- R4 → `docs/PROJECT_PLAN.md`: Phase 2.7 status note; new **Phases/Steps 2.8 /
  2.9 / 2.10**. R5 → Phase 3 removed.
- R4 → `docs/CLAUDE_CODE_PROMPTS.md`: **Prompts 2.8 / 2.9 / 2.10** added. R5 →
  Prompt 3 removed.

**TODOs surfaced (outside Stage 2's edit scope — captured at `/wind-down` / done as phase work):**
- `docs/design-decisions.md` (at `/wind-down`): capture (a) the doc-architecture
  reshape decision, (b) the file-lifecycle pattern taxonomy *why* (R3), (c) the
  downstream-of-behavior documentation principle (R2-A1).
- `docs/open-questions.md`: add the **R5** regression-test Deferred User Story
  and the **N2** "file-lifecycle patterns for users" Deferred User Story.
- The command-file edits + guidance skeleton + root `CLAUDE.md` invariants +
  live-doc migration + the `/write-documentation` reconciliation are **Steps
  2.8 / 2.9 / 2.10** phase work (not landed here).

---

## Addendum 1 — 2026-06-26

### Why this addendum exists

Round 1's **R2** (route strategic doc-feedback to `/product-visioning`) was
**REJECTED**: `/product-visioning` is exclusively for planning a new movement /
writing a new PRD, and the motivating example (promote `/product-visioning` as
the new-project entry point, demote drop-a-design-doc) is a mid-movement
product/feature addition handled as an *in-movement enhancement* — exactly as
this reshape is being handled — not a `/product-visioning` trigger. This addendum
re-opens R2 as **R2-A1** with the corrected routing. No new findings and no
external research — a pure clarifying re-open.

### Round 1 dispositions that stand (not re-opened here)

- **B1, B2, B3** — Accepted (audience-named folders; manifest relocation;
  documentation-guidance store).
- **R1** — Accepted (three Stage-2 modes incl. interactive revise).
- **R3** — Accepted with Caveats (file-lifecycle taxonomy; caveat: requires a
  `/write-documentation` reconciliation). Applies at land.
- **R4** — Accepted with Caveats (decomposition; caveat: *steps* 2.8/2.9/2.10 not
  new phases; record phase/step terminology; triggers a reconciliation). Applies
  at land.
- **R5** — Accepted (declassify Phase 3 / Prompt 3; relax FR-16).
- **N1** — Accepted (migrated command-docs go content-stale → revise-mode
  refresh; reconciliation directive). **N2** — Accepted (deferred user story).

### How to mark up Addendum 1

The finding below has its own `AUDIT NOTE — JAH:` block. Replace its placeholder
line per the legend at the top of the document. Addendum 1's disposition on
R2-A1 supersedes R2's Round-1 disposition.

### Findings

#### Recommendations (re-opened)

##### R2-A1. Strategic doc-feedback routes through the in-movement enhancement lane, not `/product-visioning` [re-opened from R2]

**Sources.** R2 (original finding) + its Round-1 REJECTED disposition;
`REQUIREMENTS.md` FR-17 (`/product-visioning` authors the next movement's PRD and
nothing else); the "in-movement enhancement lane" Deferred User Story filed this
session in `docs/open-questions.md`; rule 9 (invoke the owning skill).

**Problem.** R2 recommended routing strategic doc-feedback to
`/product-visioning`. That mis-routes it: `/product-visioning` is exclusively for
planning a new movement / writing a new PRD. A product/feature addition that
arises mid-movement (the example: promote `/product-visioning` as the new-project
entry point, demote drop-a-design-doc to a footnote) is *in-movement enhancement
/ acknowledged feature-creep*, handled within the current movement — exactly as
this reshape is — not a `/product-visioning` trigger. The store-as-product-decision-backdoor
concern R2 raised still stands; only the routing target was wrong.

**Recommendation.** Re-author the documentation-guidance **strategic boundary**:
when doc feedback is also a product/feature decision, do **not** route it to
`/product-visioning`. Handle it as an **in-movement enhancement** (the same plan →
`/design-review` → decompose lane this reshape uses), reserving
`/product-visioning` for a genuinely new movement / new PRD. The
`documentation-guidance.md` entry carries only the *doc directive* and
cross-references where the product/feature decision was made (a checkpoint/step,
or a PRD only if it rises to a new movement). The store still never writes
`PRODUCT_VISION.md`. *Downstream:* on accept, this version (not R2's) lands into
FR-18 / the ARCHITECTURE guidance-store entry; the "in-movement vs. new movement"
judgment line is the in-movement-lane story's to make crisp.

> AUDIT NOTE — JAH:
> DECISION: Documentation is downstream of behavior — it reflects what has
> already landed, and never drives a behavior change or opens a movement.
> Routing for doc-feedback:
> (1) A documentation request (reframe / re-emphasis / restructure) is a
>     documentation job: `/write-documentation` revise mode applies it across the
>     affected docs and records the directive in `documentation-guidance.md`;
>     rationale lands as a `design-decisions.md` entry at `/wind-down`. It
>     documents behavior that already exists — no `/design-review`, no
>     `/product-visioning`. (The "promote product-visioning as the entry point"
>     example is exactly this: both entry paths already exist in the skills, so
>     it is a pure emphasis reframe.)
> (2) If a request arriving as "doc feedback" actually asks for a behavior /
>     workflow change, that is not documentation: `/write-documentation` never
>     edits behavior (rule 8) — it is redirected to the behavior process (an
>     in-movement enhancement: plan → `/design-review` → decompose into steps +
>     prompts). Documentation then reconciles *after* that behavior lands
>     (reconcile / revise mode), never before.
> (3) New movements are never introduced by documentation; they arise naturally
>     at the end of a finished movement via the normal `/product-visioning` → PRD
>     → `/onboard` flow.
> The store never writes `PRODUCT_VISION.md`. On land this replaces R2's
> recommendation in FR-18 / the ARCHITECTURE guidance-store entry, and writes the
> downstream-of-behavior principle + the rule-8 redirect into the revise-mode spec.
