---
checkpoint: 004
date: 2026-06-03
reviewer: Claude (Opus 4.8) with Jamie
status: LANDED 2026-06-04
trigger: Refresh seed/drift contract — /refresh-from-repository not safe to first-run because consumers edit rules before /onboard or refresh, and current command may silently discard those edits; revisits checkpoint 002 seed/drift dispositions and whether /onboard should self-check/refresh first
---

# Design Review — Checkpoint 004

## Context

This checkpoint opened (2026-06-03) to patch the **seed/drift path**
of the just-built `/refresh-from-repository`
([`cc-template/.claude/commands/refresh-from-repository.md`](../../cc-template/.claude/commands/refresh-from-repository.md))
before its first (dogfood) source-mode run. An inline fix had been
started and pulled back when the divergences from checkpoint 002's
signed-off contract multiplied past "small patch"; the four coupled
findings are captured in [`docs/open-questions.md`](../open-questions.md)
"First-refresh seed must treat consumer rule-edits as an expected
path; collapse the redundant force-skills-only lever."

Working the patch (2026-06-04) surfaced that the whole bug class we
were chasing — wrong-side baseline in the re-seed branch, the
pre-marker migration seeding from downstream, the question of *when*
a baseline gets established — are all **artifacts of the storage
mechanism itself**: per-block content hashes plus a baseline kept in
a sidecar state file (checkpoint 002's Option D hybrid). Two
observations from Jamie collapsed the problem: (1) the memory of
"this block was deleted / customized on purpose" can live **in the
rules file itself** as a marker state (a tombstone or a `forked`
flag), established by asking once — no sidecar baseline; (2) the
*merge* is already the executing LLM session's job (002 Steps 5a /
5c), so the per-block hash is only doing **classification**, which a
two-way compare plus an ask-once question can do without any stored
baseline at all. Following that thread to its end removes the state
file, the per-block hashes, the `git hash-object` recipe, the
`last-synced` reference, and the baseline — leaving markers + the
command file's own version stamp + the LLM.

So the decision this checkpoint actually has to make has risen one
level: **not "how do we fix the seed path of the hash/baseline
mechanism" but "do we keep that mechanism at all, or replace it with
a stateless marker-state + ask-once model."** B1 below is that
choice. The earlier path is preserved as Option B so the decision and
the investigation are both on the record; the investigation summary
section documents how we got here. If Option A lands, the
hash/baseline mechanism becomes a documented **abandoned approach**
(N3) so a future session does not re-derive it.

Docs read: the command spec end-to-end; the four-pronged user story;
checkpoint 002 (B1/B1-A1, R1/R1-A1, R2a–R2d, R3, R4-A1, R6) and the
twelve refresh-related `design-decisions.md` entries; checkpoint 003
(R9 onboarding checklist, R10 pre-marker migration); `cc-template/README.md`;
the Known Limitation "`/onboard` does not preserve CC-TEMPLATE-BLOCK
markers"; `REQUIREMENTS.md` FR-13 / NFR-4 / NFR-9; `PROJECT_PLAN.md`
Phase 2.1. Verified on-disk: `cc-template/rules/*.md` are wrapped in
CC-TEMPLATE-BLOCK markers; `multi-agent-rules.md` carries exactly one
marked block (`briefing-rule`), mode content intentionally unmarked.

**Prior checkpoint this one revises.** Checkpoint 002 (LANDED
2026-05-26) pinned the mechanism now under reconsideration — Option D
hybrid, the CC-TEMPLATE-BLOCK markers, the four-section state file,
three-way semantics in source mode. The **marker syntax and the
coarse-grained wrapping (002 R1-A1 / R6) survive in both options**;
what B1 decides is whether the per-block hash + baseline + state-file
machinery survives or is replaced.

**Gate this review unblocks.** The blocked dogfood (source-mode run
against this repo's root) and Phase 2.1 close-out; and the standard
fresh-consumer path, which cannot ship until first-seed divergence is
handled safely.

**Jamie's framing (sessions 2026-06-03 / 06-04):** desired outcome is
safe first-seed behavior, not a change to the review-rules-before-
`/onboard` instruction (README may gain post-onboard "how to get new
rules later" guidance); the version stamp is the cleanest canonical
drift lever; findings may reach `/onboard` / the seed; pin FR-13 at
landing; default to defer anything that isn't a true blocker of the
dogfood or the standard path; **capture the recommended path AND the
earlier path, so the decision, the investigation, and the eventual
abandoned approach are all documented.** The provenance-by-recognition
trade in Option A (below) was reviewed and **accepted** 2026-06-04.

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

Each finding has its own AUDIT NOTE block — mark each
independently. Inline detail, caveats, and rationale are welcome
alongside the decision keyword (the keyword is what Stage 2 keys
off; the surrounding text is the audit trail).

Then re-run `/design-review`. Stage 2 will walk each marked
disposition with you, append rows to the Disposition log, and ask
whether to **land the document** or **open another addendum
round** (e.g. when a marking asks for further investigation
before the disposition is final).

**Do not fill the Sign-off Summary table at the bottom** — Stage 2
fills that when the doc lands.

## The investigation — from seed-patch to architecture reframe

Recorded so the decision in B1 is auditable and so the earlier path,
if abandoned, is not re-derived. No findings here; this is the trail.

**Where we started — the built mechanism (checkpoint 002 Option D
hybrid).** Template content wrapped in `CC-TEMPLATE-BLOCK` markers;
each block's content hashed with `git hash-object` (LF-normalized for
cross-platform determinism); a sidecar
`.claude/claude-code-sdlc-template-refresh-state.md` holding four
sections (Upstream baseline, Block hashes, Refresh logic version,
Upstream directives); reconciliation a three-way compare
(downstream-current / upstream-current / baseline-from-state-file)
matched by id. This is exactly what the command implements today.

**The seed concern that opened the review (four prongs).** (1)
Consumer rule-edits before first sync are the *expected* case, not an
edge — the README has them review/edit rules before `/onboard`. (2)
The re-seed branch (Step 2 "no state file but markers exist") seeds
the baseline from downstream-current, so a pre-existing edit freezes
into the baseline and the *next* run misclassifies it as "downstream
untouched, upstream changed" and silently reverts it. (3) The
pre-marker migration (Step 6) has the same downstream-sourced-baseline
defect. (4) The `force-skills-only` / Upstream-directives lever is
redundant with the version stamp, has no reachable use case, and
nothing ingests it — so it can only ever be empty.

**The seed-fix attempts (what would rescue Option D).** First,
baseline-source correctness: source the baseline from upstream-at-sync
everywhere it is seeded, never downstream-current. Second, *when* the
baseline is first established — at first refresh, at `/onboard`, or
shipped pre-computed inside `cc-template/` (Jamie's "ship the
baseline" idea, which makes the distribution moment the canonical sync
point and dissolves the wrong-side-baseline bug for fresh consumers,
at the cost of a maintenance discipline keeping the shipped hashes
fresh). Third, first-seed divergence UX — surface interactively vs
record-and-defer.

**The pivot.** "Ship the baseline" prompted "then why hashes at all?"
The hash table is only a *classifier* (did this block change, on which
side) — and the merge is already the LLM's job. The deletion-vs-never-
existed case (the one with no semantic signal) needs *some* memory,
but that memory can be a **tombstone written into the file** after
asking once — and the same ask-once-and-record trick generalizes to
the *edit* case (`forked` marker state). With memory living in the
files and the LLM doing two-way compare + surgical merge, the baseline
isn't needed for classification either; and with no baseline, the
`last-synced` reference has no job; and with drift handled by the
command file's own version stamp, there is **no sidecar state file at
all.**

**Where it led — the model in Option A.** Markers carry state
(template-owned / `forked` / `removed`); refresh is two-way +
ask-once-and-record; the LLM merges; the version stamp is the sole
drift lever; nothing is stored outside the (already-committed) rules
files and the command file. The single thing this is weaker at than
Option D is provenance: it asks the consumer once instead of inferring
from a baseline. Jamie accepted that trade.

## Findings

### Blockers — fix before the dogfood + the standard fresh-consumer path

One blocker: the architecture choice. Everything else is downstream of
it or independent (R1).

#### B1. Reconciliation architecture — adopt marker-state + ask-once (recommended), or retain the checkpoint-002 hash/baseline/state-file path

**Sources.** The built command [Steps 1–7](../../cc-template/.claude/commands/refresh-from-repository.md);
checkpoint 002 B1/B1-A1 (Option D), R2a/R2d, R3 caveat, R4-A1
(four-section state file); `docs/design-decisions.md` "reconciliation
mechanism", "content hashing uses git hash-object", "consumer-boundary
partition", "Source-repo refresh syncs local cc-template/ → root",
"Phase 2.1 builds in cc-template only…"; `docs/open-questions.md`
four-prong story; FR-13; NFR-4; the investigation section above.

**Problem.** The review opened to patch the seed path of the built
mechanism. The patch revealed that the entire bug class
(baseline-source, re-seed, pre-marker migration) exists *because* a
per-block hash + baseline are stored in a sidecar state file. Two
observations dissolve the class — record memory in the files
themselves (tombstone / `forked` markers, established by asking once),
and let the LLM do the two-way compare + surgical merge it already
owns. So the first-level decision is the architecture, not the seed
patch: keep the hash/baseline/state-file machine and fix its seed
path, or replace it with the stateless marker-state + ask-once model.

**Recommendation.** Pick one at markup via `DECISION:` (or `Accepted`
for the recommended Option A). The seed-fix sub-decisions that only
matter under Option B (baseline source, where the baseline is
established, surface-vs-silent UX) are documented in the investigation
section and **re-open as their own findings in Addendum 1 only if
Option B is chosen** — they are not specced as live findings here.

- **Option A (Recommended) — Stateless marker-state + ask-once
  reconciliation.**
  - **Markers carry state in the file:** template-owned (normal),
    `forked` (consumer-owned, set by ask-once), `removed` (tombstone
    where a block was deleted or moved out). HTML-comment form → free
    at session-read time (the CLAUDE.md preprocessor strips it),
    visible to refresh's raw read.
  - **Refresh is two-way + ask-once-and-record**, matched by id:
    matches upstream → skip; differs and unmarked → ask once (*mine* →
    `forked` / *take upstream* → apply / *merge* → LLM hand-merge);
    `forked` → leave, optionally note upstream now differs and offer;
    `removed` → respect, never re-add; absent with no tombstone → ask
    once (*never had it* → add / *removed it* → write tombstone); new
    upstream block → add. The LLM performs the compare and the
    surgical red/green merge (already its job per 002 Steps 5a / 5c).
  - **Drift:** the command file's own version stamp is the sole lever
    (R1) — upstream higher → pull commands skills-only, re-invoke.
  - **No state file, no per-block hashes, no `git hash-object` recipe,
    no `last-synced`, no baseline.** Source and public modes converge
    — the only difference is reading upstream-current from disk vs
    network; the "fetch baseline" branch is gone, and public mode no
    longer needs git *history*. First-seed divergence stops being a
    special case: the first refresh is a one-time guided
    yours/take/merge walk over diverged blocks, then quiet — no seed
    path, no baseline to source from the wrong side.
  - **Honest trade (rule 4):** provenance is established by asking the
    consumer once (recorded in-file) rather than inferred from a
    baseline — it relies on the consumer recognizing their own edit,
    possibly months later, with "take upstream" the git-recoverable
    safe default. This is the *only* place Option A is weaker than
    Option B; everything else is strictly simpler. **Trade accepted by
    Jamie 2026-06-04.**
  - **Dissolves/supersedes:** 002 B1-A1 (source-mode three-way — goal
    preserved via marker-state), R4-A1 (the state file), the
    git-hash-object content-hashing decision, R2d's hash-match
    auto-upgrade-offer (drop, or relocate-detect semantically),
    "state file is dogfood-generated and committed." *Cost:* a larger
    command rewrite, an FR-13 rewrite, several superseded
    design-decisions entries, and the abandoned-approach record for
    Option B (N3).

- **Option B — Retain checkpoint-002's Option D hybrid and fix its
  seed path.** Keep markers + per-block hashes + file-level baseline +
  four-section state file + three-way merge, exactly as built and
  signed off. To make it safe, resolve the seed-fix sub-decisions from
  the investigation section: (i) source the baseline from
  upstream-at-sync, never downstream-current (the silent-revert root
  cause, in both the re-seed branch and Step 6); (ii) decide where the
  baseline is first established (first refresh / `/onboard` / shipped
  pre-computed in `cc-template/`); (iii) decide first-seed divergence
  UX (surface vs silent). *Retains* deterministic provenance — the one
  thing Option A trades away. *Debt:* the correctness-and-maintenance
  surface this review exists to chase — keeping seeded/shipped hashes
  correct, cross-platform LF/`hash-object` discipline, committing the
  state file so the baseline survives a clean clone, and the recurring
  drift risk across all of those. Choosing B is choosing the richer
  mechanism for the determinism it preserves; per rule 4, Option A is
  the simpler alternative.

> AUDIT NOTE — JAH:
> _[DECISION: (A)_

### Recommendations — resolve before or during the gate

#### R1. Version stamp is the sole drift lever; remove `force-skills-only` / Upstream directives

**Sources.** Command [Step 1.3](../../cc-template/.claude/commands/refresh-from-repository.md)
(reads `force-skills-only-from`), [Step 2](../../cc-template/.claude/commands/refresh-from-repository.md)
(`## Upstream directives`), [Step 7.1](../../cc-template/.claude/commands/refresh-from-repository.md)
(clear-if-consumed); the `Refresh logic version` stamp in the command
header; checkpoint 002 R3 caveat + R4-A1; `docs/design-decisions.md`
"consumer-boundary partition" vs the 002 R3 disposition row (they
disagree on whether the directive lives in the state file or the
command file); `docs/open-questions.md` prong 4; FR-13; NFR-4.

**Problem.** The directive duplicates the version-stamp drift signal
(a version bump already stages every downstream behind it), its one
notionally-distinct capability has no reachable use case, nothing
ingests it into the downstream-owned file, and the docs disagree on
where it lives. Independent of B1: it is dead weight in either
architecture.

**Recommendation.** Make the command file's version stamp the **sole
drift lever**; remove the `force-skills-only-from` read (Step 1.3),
the `## Upstream directives` section, and the Step 7.1 clear. Per
Jamie's standing Q3 answer, **edit FR-13 at landing** — noting the
edit's shape is contingent on B1: under **Option A** FR-13 is
rewritten to the marker-state model (no state file at all, the lever
removal subsumed); under **Option B** the state file drops from four
sections to three. The lever-removal decision itself is firm
regardless. Bump the command's Refresh-logic-version stamp.

> AUDIT NOTE — JAH:
> _Accepted._

### Notes — acceptable now, revisit later

#### N1. `/onboard` preserving the `briefing-rule` marker (more load-bearing under Option A)

**Sources.** [`cc-template/.claude/commands/onboard.md` lines 329–343](../../cc-template/.claude/commands/onboard.md#L329-L343);
[`cc-template/rules/multi-agent-rules.md` lines 17–29](../../cc-template/rules/multi-agent-rules.md#L17-L29);
`docs/open-questions.md` Known Limitation "`/onboard` does not preserve
CC-TEMPLATE-BLOCK markers."

**Problem.** Verified narrow: `multi-agent-rules.md` has exactly one
marked block (`briefing-rule`); the mode content `/onboard` rewrites
wholesale is unmarked, and `project-rules.md`'s marked blocks sit
above the divider `/onboard` doesn't rewrite. The only risk is that
`/onboard`'s "replace the placeholder" instruction doesn't say
*preserve the `briefing-rule` markers verbatim*. Under **Option A**
markers are *memory-bearing* (they hold `forked`/`removed` state), so
preservation matters more — but the ask-once migration still
self-heals a dropped marker by re-inserting and asking.

**Recommendation.** Defer. If B1 lands Option A, queue a TODO to make
`/onboard.md` preserve the `briefing-rule` block verbatim (tighter
than relying on self-heal); under Option B this stays the existing
Known Limitation with pre-marker-migration self-heal.

> AUDIT NOTE — JAH:
> _DECISION: Fix this now in this iteration, no need for a later TODO._

#### N2. Dogfood behavior under Option A

**Sources.** Command [Step 6](../../cc-template/.claude/commands/refresh-from-repository.md);
`docs/design-decisions.md` "Phase 2.1 builds in cc-template only…";
`docs/open-questions.md` prong 3; `TODO.txt` watch note
(root explore-plus-plan; only `briefing-rule` should align); FR-13
source-mode + NFR-9.

**Problem.** Under **Option A** the pre-marker migration becomes
"insert markers aligning to upstream + ask-once on divergent blocks"
— there is **no state file to seed and no baseline to source, so B2's
silent-revert defect cannot occur.** Root is this repo (a consumer of
its own template), so most blocks match `cc-template/` and wrap
cleanly; wherever `cc-template/` was cleaned ahead of root (the
checkpoint 003 mandate), root differs → surfaced as "take upstream"
during the guided walk. `multi-agent-rules.md` behaves exactly as the
TODO watch note predicts: only `briefing-rule` has an id and gets
wrapped (one clean wrap or one question); the explore-plus-plan mode
content has no upstream id → stays unmarked, untouched. The committed
artifact is root with markers + content + any `forked`/`removed`
states — no state file.

**Recommendation.** Under Option A the dogfood is safe to run after
the command rewrite lands in `cc-template/`; it is the clean first
test of the ask-once migration every consumer will use. Under Option
B it is gated on the baseline-source fix first. "Inspect the diff
before committing" remains the review surface.

> AUDIT NOTE — JAH:
> _Accepted with Caveat: We will fix the briefing-rule preservation before dogfood._

#### N3. Landing scope + abandoned-approach capture

**Sources.** `CLAUDE.md` § Load-bearing invariants (Stage 2 edit scope
= REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN / CLAUDE_CODE_PROMPTS +
checkpoint + REVIEWS.md); NFR-9 (edit `cc-template/` first, copy up);
the absent root `.claude/commands/refresh-from-repository.md` (arrives
via the dogfood); `docs/open-questions.md` § Abandoned Approaches.

**Problem.** Under **Option A** the behavioral change is a substantial
**command rewrite** (markers-with-state, two-way + ask-once, drop the
state file / hashing / baseline) plus an **FR-13 rewrite** and several
**superseded `design-decisions.md` entries** (reconciliation
mechanism, content-hashing, consumer-boundary partition, "state file
is dogfood-generated"). The command file and design-decisions are
outside Stage 2's edit scope; only FR-13 and any PROJECT_PLAN phase
text are in-scope Stage 2 doc edits. The rest are TODOs against
`cc-template/`, carried onto root by the dogfood (N2).

**Recommendation.** Expect Stage 2 landing (if Option A) to produce:
the in-scope FR-13 (+ PROJECT_PLAN Phase 2.1) edits; and TODOs to
(a) rewrite `cc-template/.claude/commands/refresh-from-repository.md`,
(b) update/supersede the affected `design-decisions.md` entries,
(c) **write an Abandoned Approaches entry in `docs/open-questions.md`
for the hash/baseline/state-file mechanism** — what it was, why it was
abandoned (the seed/drift bug class + maintenance surface), and that
provenance-by-recognition was accepted as the trade — so a future
session does not re-derive it, (d) update `cc-template/README.md`
"Keeping a project up to date" to the new behavior, and (e) preserve
the 002 marker-syntax / coarse-wrapping decisions that survive.

> AUDIT NOTE — JAH:
> _Accepted._

## What the design got right (preserve)

- **The reframe was caught before the dogfood, not after.** Running
  refresh against root would have committed a state file built on the
  flawed baseline logic; routing to a checkpoint kept the 002 contract
  from being amended ad hoc and opened room to question the mechanism
  itself.
- **Markers + LLM-merge are the durable core.** The marker syntax and
  coarse-grained wrapping (002 R1-A1 / R6) survive both options; the
  executing-session-merges principle (002 5a / 5c,
  `feedback_ai_not_heuristic`) is what makes Option A possible.
- **Contract-first thinking from 002** (template marks its own
  content; respect deletion; surface, don't overwrite) carries
  straight into the marker-state model — `forked`/`removed` are just
  that contract made explicit in the file.
- **The version stamp is a clean single drift signal** — R1 simplifies
  toward what already works rather than adding machinery.

## Next steps

1. Jamie marks B1 (`DECISION: A` or `B`), R1, and the notes inline.
2. Re-run `/design-review`. Stage 2 walks each disposition. If B1 = A,
   the seed-fix sub-findings stay closed (the mechanism that needed
   them is gone) and landing produces the FR-13 edit + the TODOs in
   N3, including the abandoned-approach record. If B1 = B, Stage 2
   recommends opening Addendum 1 to spec the seed-fix sub-decisions
   (baseline source, where established, surface-vs-silent) as live
   findings.
3. After landing (Option A): rewrite the command in `cc-template/`,
   run the source-mode dogfood (N2), then pin the surviving
   marker-syntax invariants into root `CLAUDE.md` and close Phase 2.1.

## Disposition log

*Stage 2 fills this section. Empty until Stage 2's first walk.
Rows accumulate append-only across rounds — later-round
dispositions on the same finding ID supersede earlier ones for
the purposes of Stage 2's landing application.*

| Round | Finding | Disposition | Reason |
|-------|---------|-------------|--------|
| Original | B1 | Decision | Option A — stateless marker-state + ask-once reconciliation; replaces the hash/baseline/state-file mechanism. Provenance-by-recognition trade accepted. |
| Original | R1 | Accepted | Version stamp is the sole drift lever; remove `force-skills-only` / Upstream directives; FR-13 edited at landing; bump the Refresh-logic-version stamp. |
| Original | N1 | Decision | Fix `/onboard` `briefing-rule` preservation now in this iteration (not a deferred TODO); Jamie authorized an in-session `onboard.md` edit despite it being outside Stage 2's normal edit scope. |
| Original | N2 | Accepted with Caveats | Dogfood is safe under Option A; caveat: fix the `briefing-rule` preservation before the dogfood. |
| Original | N3 | Accepted | Landing produces in-scope FR-13 (+ PROJECT_PLAN Phase 2.1) edits plus the N3 TODO bundle, including the Abandoned Approaches record for the hash/baseline/state-file mechanism. |

## Sign-off Summary

*Filled when Stage 2 lands the doc. Jamie does not edit this
section — sign-offs go inline above.*

| ID | Final disposition |
|----|-------------------|
| B1 | Decision: Option A — stateless marker-state + ask-once reconciliation (replaces the hash/baseline/state-file mechanism). |
| R1 | Accepted — version stamp is the sole drift lever; `force-skills-only` / Upstream directives removed; FR-13 edited at landing. |
| N1 | Decision: `/onboard` `briefing-rule` preservation fixed this pass (in-session, authorized scope exception). |
| N2 | Accepted with Caveats — dogfood safe under Option A; fix briefing-rule preservation before dogfood (done at landing). |
| N3 | Accepted — landing produced the in-scope FR-13 / NFR-4 / PROJECT_PLAN edits plus the Phase 2.1.A TODO bundle incl. the Abandoned Approaches record. |

## Follow-up actions landed

*Filled when Stage 2 lands the doc. Lists every doc edit and
every TODO that landed during the landing pass.*

**In-scope doc edits applied this landing:**

- B1: rewrote `docs/REQUIREMENTS.md` FR-13 to the Option A stateless
  marker-state + ask-once contract (no state file / hashes /
  baseline).
- B1 / NFR-4 drift: revised `docs/REQUIREMENTS.md` NFR-4 — marker
  blocks now carry `template-owned` / `forked` / `removed` state;
  supersedes the 002 "no inline metadata" pin; removes the refresh
  state-file path + schema as load-bearing.
- R1: lever removal folded into the FR-13 rewrite (version stamp is
  the sole drift lever); command-file changes are Phase 2.1.A.
- N3 / plan coherence: added a "built then reframed" status note to
  `docs/PROJECT_PLAN.md` Phase 2.1 and a new **Phase 2.1.A**
  (Option A rewrite); added a 2026-06-04 supersession note to
  `docs/CLAUDE_CODE_PROMPTS.md` Prompt 2.1's footer (run body
  untouched) and a new **Prompt 2.1.A**.
- N1: edited `cc-template/.claude/commands/onboard.md` and its root
  mirror to preserve the `briefing-rule` `CC-TEMPLATE-BLOCK` markers
  when `/onboard` rewrites `multi-agent-rules.md` (authorized
  in-session scope exception; the consumer-facing instruction is
  self-contained, no checkpoint reference).

**TODOs queued for Phase 2.1.A (out of Stage 2's edit scope):**

- (a) Rewrite `cc-template/.claude/commands/refresh-from-repository.md`
  to Option A + the full Prompt 2.1.A rewrite work; bump the
  `Refresh-logic-version` stamp.
- (b) Update/supersede the affected `docs/design-decisions.md`
  entries (reconciliation mechanism, content-hashing,
  consumer-boundary partition, "state file is dogfood-generated").
- (c) Write the Abandoned Approaches entry in `docs/open-questions.md`
  for the hash/baseline/state-file mechanism (what it was, why
  abandoned, provenance-by-recognition accepted as the trade).
- (d) Update `cc-template/README.md` "Keeping a project up to date"
  to the Option A behavior.
- (e) Run the source-mode dogfood (N2) — after the briefing-rule fix
  (already landed); then pin the surviving marker-syntax invariants
  into root `CLAUDE.md` and close Phase 2.1 + 2.1.A.
