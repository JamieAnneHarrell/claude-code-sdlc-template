---
checkpoint: 002
date: 2026-05-26
reviewer: Claude (Opus 4.7) with Jamie
status: LANDED 2026-05-26
trigger: Pre-Phase-2.1 sanity check — /refresh-from-repository design contract and consumer-edit reconciliation
---

# Design Review — Checkpoint 002

## Context

This checkpoint scrutinizes the `/refresh-from-repository` design
contract before Phase 2.1's build session opens. Five
`docs/design-decisions.md` entries landed 2026-05-26 setting the
contract: single-stage with skills-first drift staging; template
marks its own content; source-mode sync; progressive-disclosure
flags; vendored-template lock-in. Four reconciliation-algorithm
options sit open in `docs/open-questions.md`, and three additional
user stories in the same file are candidates for disposition here.

`/refresh-from-repository` is the highest-risk transition in the
roadmap: it's the first network-touching command in the template,
the first self-modifying command, and it has to merge content that
downstream consumers may have edited — including the rules that
ship in `cc-template/rules/`. A consumer-edit case the contract
doesn't handle becomes a silent regression that erodes trust in
the upgrade path. The review focuses there.

Prior checkpoint 001 (LANDED 2026-05-24) introduced Phase 1.2
(cleanup) and the Phase 2 parent header; nothing in 001's
dispositions conflicts with this review's scope. The plan-mode →
`/wind-down` user story in `docs/open-questions.md` is *out of
scope* for this checkpoint per Jamie's direction; it routes to a
separate cleanup session scheduled before Phase 2.1 starts (TODO
surfaced at landing).

Jamie's framing for this review:

- The reconciliation-algorithm finding (B1) presents options
  neutrally; she picks via `DECISION:` at markup time.
- The four consumer-edit scenarios — wholesale rule deletion,
  partial edit inside marker, rules-file reordering, cross-file
  migration — each get an independent decision (R2a / R2b / R2c /
  R2d) because the contract today names a matrix exists but
  doesn't spell out each scenario.
- Findings should be sparing in the Blocker tier (only true
  show-stoppers for Phase 2.1 to begin); optimizations and
  observations go in Recommendations or Notes.

The gate this review unblocks is Phase 2.1's build session:
authoring `cc-template/.claude/commands/refresh-from-repository.md`,
pinning the template-marker syntax (becomes load-bearing per
NFR-4), and resolving the algorithm choice.

## How to mark up this review

For each finding below, replace the placeholder line under
`AUDIT NOTE — JAH:` with one of:

- `Accepted`
- `Accepted with caveats: <your caveats>`
- `Defer Approved`
- `DECISION: <explicit choice>`

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

## Findings

### Blockers — fix before Phase 2.1 build session opens

#### B1. Reconciliation algorithm not picked — four options still open

**Sources.** `docs/open-questions.md` "Reconciliation algorithm
for `/refresh-from-repository`" (lines 49–83); `docs/design-
decisions.md` "Template marks its own content; reconciliation
tolerates divergence" scope note (lines 416–421: "exact
reconciliation algorithm ... is deferred to the Phase 2.1 build
session and the pre-Phase-2.1 `/design-review` will sanity-check
the choice"); `docs/REQUIREMENTS.md` FR-13; `docs/PROJECT_PLAN.md`
Phase 2.1 deliverables (line 198: "Reconciliation algorithm
chosen from the options").

**Problem.** Phase 2.1's build session cannot meaningfully begin
without an algorithm decision because the algorithm determines
(a) what metadata each template marker carries (hash? baseline
copy? nothing?), (b) where baseline content lives (inline in the
marker? top-of-file stamp? git history?), (c) what upstream-side
discipline is required (release tags? plain commits?), and
(d) the comparison logic the command implements. Authoring the
command spec without picking forecloses or pre-commits these
sub-decisions implicitly.

**Recommendation.** Pick one of the four options below at markup
via `DECISION:`. Each option's tradeoffs are summarized; downstream
implications (what marker syntax it forces, what baseline
storage it requires) are named in 1–2 sentences each.

- **Option A — Per-block content hash recorded in the marker.**
  Each template block carries `hash=<sha>` in its open marker.
  Refresh recomputes the live hash and compares to the recorded
  one to detect "consumer edited inline." Self-contained per
  file; no out-of-band state; no upstream-side discipline beyond
  computing-and-stamping hashes when content changes.
  *Downstream:* marker syntax must include a `hash=` field;
  refresh has to update the hash on every successful merge.
  Doesn't on its own distinguish "block-new-since-baseline" from
  "block-the-user-removed" — needs to pair with either (a) a
  remembered baseline list of block IDs the consumer last
  sync'd, or (b) just-trust-deletion-as-deliberate (which
  conflicts with the Android-deletes-"iPhone, not Android"
  case in R2a).
- **Option B — File-level commit-stamp baseline.** A
  `last-synced-at: <upstream-commit>` stamp at the top of each
  refresh-managed file. Refresh fetches that upstream commit's
  version of the file as the baseline, then runs a three-way
  comparison: downstream-current, upstream-current, baseline.
  Distinguishes inline edits, intentional deletions, and
  upstream-only additions cleanly. *Downstream:* needs a place
  to persist the commit hash (top-of-file HTML comment is the
  obvious choice); requires the public upstream repo to keep
  history accessible (already true via GitHub); requires
  fetching arbitrary historical content (modest network cost
  per refresh).
- **Option C — Git 3-way merge against tagged upstream releases.**
  Upstream tags releases; downstream records the tag it sync'd
  from in a top-of-file stamp; refresh runs `git merge-file`
  between downstream, current upstream, and the baseline-tag
  content. *Downstream:* leverages git's well-understood merge
  engine; depends on upstream maintaining release-tag
  discipline (a non-trivial process commitment for a
  one-maintainer project); inline-conflict surfacing inherits
  git's conflict-marker style, which is unfamiliar to consumers
  reading a markdown file.
- **Option D — Hybrid: per-block hash PLUS file-level commit
  stamp.** Hash for cheap inline-edit detection; commit stamp
  provides the baseline that resolves "deletion vs
  never-existed" cleanly. The two signals are complementary; both
  primitives are independently useful. *Downstream:* marker
  syntax carries a `hash=` field AND each refresh-managed file
  has a top-of-file `last-synced-at:` stamp. Larger surface area
  to maintain but no single signal is load-bearing alone — if
  one drifts, the other still catches most cases.

> AUDIT NOTE — JAH:
> _DECISION: OPTION D - Hybrid. CAVEAT: How does this work with a local cc-template project such as a maintainer version or one that intentionally choses to pin a version by having a cc-template copied into their local repo? Need to resolve._

### Recommendations — resolve before Phase 2.1 implementation

#### R1. Template-marker syntax — minimum metadata and ONBOARD-FILL coexistence rules

**Sources.** `docs/REQUIREMENTS.md` NFR-4 (template-marker syntax
load-bearing once shipped); `docs/design-decisions.md` "Template
marks its own content" (lines 372–407, names markers but not the
exact syntax); `docs/PROJECT_PLAN.md` Phase 2.1 deliverables
("Template marker syntax pinned (open/close strings, embedded
metadata fields). Becomes load-bearing per NFR-4 once shipped");
existing `<!-- ONBOARD-FILL: project-scope -->` / `<!-- /ONBOARD-
FILL -->` pair in `rules/project-rules.md` (the inverse primitive
for consumer-owned content).

**Problem.** Phase 2.1's scope item 2 says "pin the template-
marker syntax." The contract today names markers exist, but the
exact syntax — open/close strings, embedded metadata fields,
unique block identifiers — is open. Two sub-decisions are
load-bearing for the consumer-edit scenarios (R2a–R2d) and
they're independent of B1's algorithm pick:

1. **Block identifiers.** Each template-marker block needs a
   stable unique identifier so refresh can match upstream blocks
   to downstream blocks under reordering (R2c). Without IDs,
   matching is positional and reordering surfaces as conflict on
   every refresh. *Downstream of B1:* if B1 picks per-block hash
   (A or D), the hash needs somewhere to attach — block IDs are
   the natural anchor; if B1 picks file-level commit-stamp (B) or
   git-3-way (C), block IDs are still useful for human-readable
   conflict reports.
2. **ONBOARD-FILL coexistence.** `<!-- ONBOARD-FILL: <name> -->` /
   `<!-- /ONBOARD-FILL -->` already exists. The new template-
   marker syntax must be visually distinguishable at a glance
   (so consumers don't mistake one for the other) and must NOT
   nest or overlap with ONBOARD-FILL blocks (a single line can't
   be both consumer-owned and template-owned). Worth pinning the
   rule in the design-decisions entry.

**Recommendation.** In Phase 2.1, before writing the command
spec, pin:

- An open/close marker pair distinguishable from ONBOARD-FILL by
  the keyword (e.g. `<!-- TEMPLATE-BLOCK: <id> ... -->` /
  `<!-- /TEMPLATE-BLOCK -->`, or another naming that doesn't
  reuse "FILL" or "ONBOARD"). Exact strings TBD by the build
  session.
- A unique `id` field on the open marker (kebab-case,
  human-readable, scoped per-file — uniqueness within a file is
  enough; cross-file references aren't a use case).
- Whatever B1's algorithm needs for metadata (hash, baseline
  reference, both, or neither — derived from the B1 pick).
- An invariant in the design-decisions entry: TEMPLATE-BLOCK and
  ONBOARD-FILL blocks may not nest or overlap; a refresh-managed
  file is partitioned into TEMPLATE-BLOCK regions, ONBOARD-FILL
  regions, and free regions (where free regions are consumer-
  personalized, never touched by refresh).

Add the exact marker syntax to root `CLAUDE.md` Load-bearing
invariants section at Phase 2.1 landing (per Prompt 2.1 scope
item 10).

> AUDIT NOTE — JAH:
> _DECISION: CC-TEMPLATE-BLOCK: <id> where ID is a block hash_

#### R2a. Consumer-edit scenario — wholesale rule deletion (Android case)

**Sources.** `docs/design-decisions.md` "Template marks its own
content" consumer-contract bullet ("To opt out: move the block
out of the template region into your own personalized-rules
section, **or delete the block entirely**. Refresh respects the
absence as your decision."); the canonical example from the same
file's "Why" section (lines 393–401: "Android project deletes
'iPhone, not Android' ... refresh has no signal it was
deliberate, silently re-adds the rule").

**Problem.** The "delete block, refresh respects absence" rule
covers the case where the consumer deletes the *contents* of a
marker block but leaves the marker pair. What about the case
where the consumer deletes the entire marker pair AND its
contents — i.e. the marker for that block is gone from the file
entirely? Refresh has no marker to anchor against. Two
interpretations:

1. **Anchored deletion respects** — refresh detects "upstream has
   block `<id>`, downstream has no marker with `<id>`," concludes
   "consumer deleted deliberately," and skips re-adding.
2. **Anchored deletion confused** — without a baseline list of
   block IDs the consumer last sync'd (or a baseline file
   per B1's pick), refresh can't distinguish "block deleted
   deliberately" from "block-never-existed-yet-because-the-
   consumer-is-on-an-older-version." It might re-add the
   "iPhone, not Android" rule that the Android consumer
   deliberately deleted.

The resolution depends on B1: per-block hash alone can't tell;
file-level commit-stamp baseline can; git-3-way can.

**Recommendation.** Pin in the design-decisions entry: deletion-
detection requires comparing downstream against a baseline (not
against upstream-current alone). Whichever B1 option lands must
provide that baseline. If B1 picks A (per-block hash alone),
either pair it with a remembered block-ID list at the top of each
refresh-managed file (file-local baseline), or accept that
wholesale-deletion is *not* a supported consumer-edit case and
document that limitation in the contract. The "Android deletes
iPhone-rule" example must work cleanly or the contract is
misleading.

> AUDIT NOTE — JAH:
> _DECISION: B1 landed with BLOCK ID and FILE HASH (Option D). That should provide sufficient markers for the contract. If any list of commit stamps needs to be stored, don't polute user readable files with those. Chose a header or footer within the skill itself. Unrecognized local block IDs could be a marker to suggest a --refresh-skills-only pass to ensure we know of all markers._

#### R2b. Consumer-edit scenario — partial edit inside marker (UX of conflict surfacing)

**Sources.** `docs/design-decisions.md` "Template marks its own
content" consumer-contract bullet ("Inline edits are detected and
surfaced, not silently overwritten") and "Semantic conflict
detection is the executing session's job" paragraph (lines
408–414); `docs/PROJECT_PLAN.md` Phase 2.1 deliverable line 211
("Semantic conflict surfacing across template/personalized
regions is the executing session's job").

**Problem.** The contract says inline edits are surfaced, not
overwritten. It doesn't say *how*: pause and ask the consumer
inline? Write a sidecar diff file the consumer reviews after?
Use git-style conflict markers in the file? Each has different
implications for the consumer's experience and for refresh's
atomicity. The "executing session's job" framing covers semantic
conflicts (upstream's new rule contradicts consumer's
personalization) but doesn't specify the syntactic-conflict UX
(consumer softened a sentence in a TEMPLATE-BLOCK).

**Recommendation.** Pin: when refresh detects an inline-edit
divergence in a TEMPLATE-BLOCK, it surfaces the conflict to the
running Claude session inline (one block at a time), showing
downstream-current vs upstream-current side-by-side, and asks the
consumer to choose: accept upstream (lose the edit), keep
downstream (skip this block this refresh), or hand-merge
(consumer edits the resolved block in the session). Don't write
sidecar files (consumer has to find and clean up); don't use
git-style conflict markers (consumers reading a `.md` file
shouldn't have to learn git merge syntax). The session is the
right venue because the consumer is already in conversation; the
existing Stage-2-style "walk one at a time" pattern from
`/design-review` is a working precedent.

> AUDIT NOTE — JAH:
> _Accepted._

#### R2c. Consumer-edit scenario — rules-file section reordering

**Sources.** `docs/design-decisions.md` "Template marks its own
content" (silent on reordering); R1 above (block identifiers).

**Problem.** A consumer reorders sections in
`coding-session-rules.md` (e.g. moves rule 7 above rule 4 because
their team treats commit discipline as more foundational). All
TEMPLATE-BLOCK markers stay intact. Two ways refresh could handle
this:

1. **Match by block ID.** Refresh matches upstream blocks to
   downstream blocks by `id` field, not by file position. Reorder
   is preserved across refreshes; consumer order survives.
2. **Match by position.** Refresh expects blocks in upstream
   order. Reorder surfaces as conflict on every block, or
   refresh silently restores upstream order.

Downstream of R1 — if R1 mandates block IDs, option 1 is
natural; if R1 omits IDs, option 2 is forced.

**Recommendation.** Pair with R1: block IDs are required, and
refresh matches by ID not by position. Reordering is a respected
form of consumer personalization. Document this explicitly in
the design-decisions entry — it's the kind of behavior consumers
won't predict and will be pleasantly surprised by, but only if
the contract names it.

> AUDIT NOTE — JAH:
> _Accepted._

#### R2d. Consumer-edit scenario — cross-file rule migration

**Sources.** `docs/design-decisions.md` "Template marks its own
content" consumer-contract bullet ("To opt out: move the block
out of the template region into your own personalized-rules
section"); contract silence on cross-*file* moves.

**Problem.** "Move out of template region" today means within the
same file (consumer cuts the block out of the TEMPLATE-BLOCK
region and pastes it into a free region below). What about moving
*across* files — consumer takes rule 5 out of
`rules/coding-session-rules.md` and pastes a modified version
into a new `rules/our-team-rules.md`? Refresh sees the block
missing from coding-session-rules.md → respects the absence (per
R2a's resolution). Upstream later updates rule 5's text. Refresh
has no way to know the consumer's `our-team-rules.md` contains a
migrated version and might want the update propagated, or might
not. Two reasonable interpretations:

1. **No cross-file tracking.** Refresh treats each file
   independently. Cross-file moves = deletion in the source file
   + new content in the destination file, period. Consumer
   accepts that upstream improvements to migrated rules don't
   propagate automatically. (This is the same tradeoff personalized
   rules make today — see the "Personalized rules are yours
   forever" contract bullet.)
2. **Migration tracking.** A consumer can annotate the migrated
   block in their new file with `<!-- TEMPLATE-MIGRATED-FROM:
   <original-id> -->` to signal "this was originally template
   block X." Refresh can then surface updates to block X with a
   prompt: "upstream changed rule 5; your migrated version is in
   our-team-rules.md — want to review the upstream change?"

Option 2 is more capable but adds a primitive (TEMPLATE-MIGRATED-
FROM) and an interaction; option 1 is KISS and matches the
"personalized rules don't get upstream improvements" precedent.

**Recommendation.** Option 1 — treat cross-file moves as plain
deletion plus consumer-owned content. Document the limitation in
the design-decisions entry's consumer contract: "Moving a
TEMPLATE-BLOCK out of its file is the same as deleting it as far
as refresh is concerned — upstream improvements won't propagate."
Defer option 2 to evidence-of-need; consumer migration is rare
and the limitation is consistent with the existing "personalized
rules are yours forever" stance.

> AUDIT NOTE — JAH:
> _Accepted with Caveat: if it is wholesale moved within one of our own known files and the hash matches we should be able to detect the rule that hash matched and offer to upgrade it with a newer version._

#### R3. CLAUDE.md merge boundary — what's template-owned post-onboard?

**Sources.** `docs/design-decisions.md` "Refresh progressive-
disclosure flags" (`--no-claudemd` exists for heavy customizers);
`docs/PROJECT_PLAN.md` Phase 2.1 deliverable line 209 ("Template
markers added to refresh-managed regions of rules files and
CLAUDE.md (post-onboard wrapping is part of this phase's work)");
`cc-template/CLAUDE.md` structure (Reading-order, Collaboration-
rules pointers, banner) vs an onboarded downstream CLAUDE.md
(adds project-name, banner customization, Project-specific
context, sometimes Load-bearing invariants).

**Problem.** `cc-template/CLAUDE.md` (pre-onboard placeholder)
and a downstream's `CLAUDE.md` post-onboard are structurally
different. `/onboard` rewrites the banner, fills sections, may
add a "Project-specific context" section. Phase 2.1's scope says
"template markers added to refresh-managed regions of CLAUDE.md
(post-onboard wrapping is part of this phase's work)" — but the
*partition* of which blocks are template-owned vs consumer-owned
post-onboard isn't specified. A few obvious candidates:

- **Template-owned:** Collaboration-rules section (the
  "MUST-DO before responding to the first user message" block,
  the rules-file pointer list, the rule-drift signal paragraph).
  Reading-order section (item names and brief descriptions).
- **Consumer-owned:** Banner content (project name, language,
  multi-agent mode, shell, run/test commands, developer setup
  pointer). Project-specific context section. Anything below the
  Load-bearing invariants section that's project-specific.
- **Mixed / ambiguous:** Load-bearing invariants section itself
  (template-defined invariants like ONBOARD-FILL, status
  comments, filename conventions — but consumers may add project-
  specific invariants).

Without a partition, Phase 2.1 either over-wraps (refresh
overwrites the banner) or under-wraps (consumer loses upstream
improvements to the Collaboration-rules block).

**Recommendation.** Phase 2.1's CLAUDE.md wrapping work pins the
partition explicitly in the design-decisions entry. A reasonable
default: Collaboration-rules section and Reading-order section
are template-owned (wrapped in TEMPLATE-BLOCK); banner, Project-
specific context, and the Load-bearing invariants section are
consumer-owned (no wrapping — free regions, refresh never
touches). The Load-bearing-invariants caveat: upstream may want
to push invariant updates (e.g. adding a new ONBOARD-FILL
marker), and downstream may have added project-specific entries —
this is the case where `--no-claudemd` exists as the escape
hatch, and the contract should say so. Don't try to merge
Load-bearing invariants; it's the messiest section to partition
and the `--no-claudemd` flag covers the consumer who needs
upstream's invariant updates.

> AUDIT NOTE — JAH:
> _Accepted with caveats: An upstream revision that wants to add invariants should be able to force a --refresh-skills-only. A contract needs to be made such that a downstream looks at the upstream before running and determines if refresh is necessary before running. This allows a future version of the code to solve mergin issues differently than we guess will work now in this first pass._

#### R4. Drift-detection version-stamp mechanism

**Sources.** `docs/design-decisions.md` "/refresh-from-repository
is single-stage with skills-first staging on detected drift"
("the command fetches upstream's current `refresh-from-
repository.md`, compares the version-stamp / marker-convention
metadata to the locally-loaded copy, and on drift pulls only
`.claude/commands/*.md`"); `docs/REQUIREMENTS.md` FR-13
(auto-detects "merge-logic drift").

**Problem.** Drift detection is a load-bearing safety mechanism
(it's what prevents stale refresh logic from misinterpreting new
upstream marker syntax). The design-decisions entry says it
compares "version-stamp / marker-convention metadata" but doesn't
say where that metadata lives in the command file. Two
candidates:

1. **A version-stamp comment at the top of
   `refresh-from-repository.md`.** E.g.
   `<!-- REFRESH-LOGIC-VERSION: 2 -->`. Refresh reads this from
   the locally-loaded copy and the fetched upstream copy; if
   integer values differ, drift. Simple to maintain; one source
   of truth.
2. **Marker-convention metadata inferred from the spec body.**
   Refresh would have to parse the spec to figure out the
   current marker convention, which is fragile and couples the
   drift check to the spec's prose shape.

Without pinning the mechanism, Phase 2.1 either invents one
ad-hoc (likely option 1, but possibly without thinking through
when to bump the version) or defers, in which case drift
detection becomes a Phase 2.1 build-time scramble.

**Recommendation.** Pick option 1 and pin in the design-decisions
entry: `<!-- REFRESH-LOGIC-VERSION: N -->` at the top of
`refresh-from-repository.md`, integer monotonically incremented
whenever the marker convention or the merge algorithm shape
changes in a way that older refresh logic would misinterpret
(spec polish that doesn't change semantics doesn't require a
bump). Refresh's first action on invocation is to fetch upstream's
copy, compare the integer, and stage skills-first if upstream's
is higher. Document the bump-trigger criteria in the command spec
so future maintainers (including future-Claude editing the spec)
know when to increment.

> AUDIT NOTE — JAH:
> _Acceoted with Caveat: Couldn't metadata be stored in a .claude-code-sdlc-template-version.md file with the commit number in the commands directory? what is the KISS option here that we're missing?_

#### R5. "Before running this prompt" header-block pattern — disposition

**Sources.** `docs/open-questions.md` "'Before running this
prompt' header-block pattern in CLAUDE_CODE_PROMPTS.md" (lines
143–187); `.claude/commands/onboard.md` step
`docs/CLAUDE_CODE_PROMPTS.md` ("Design-review checkpoints between
prompts" — plants the header block on each forward-looking
prompt); `docs/CLAUDE_CODE_PROMPTS.md` Prompt 2.2 + Prompt 3 (still
carry the pattern); Prompt 2.1 had its header block stripped in
the 2026-05-26 cleanup; Prompts 1.1 and 1.2 carry historical-state
header blocks; this project's convention is "don't edit landed
prompts retroactively."

**Problem.** The header block was meant as a between-phase
checkpoint signal but ended up conflating two things: (a) a *gate
assertion* the prompt has no way to verify ("the pre-Phase-X
review has landed") — false-positives are noticed only after the
consumer starts the work; (b) a *next-session reminder* that
arguably belongs in TODO.txt at pickup time, not embedded in the
prompt body. The first-class `## Design Review Checkpoint —
pre-Phase-X` entry in PROJECT_PLAN.md already carries the
authoritative between-phase signal; the header block is
redundant noise at best, misleading at worst.

**Recommendation.** Option A — retire the header-block-note form
entirely from `/onboard`'s spec. Rationale: KISS + progressive
disclosure. PROJECT_PLAN.md first-class checkpoint entries are
the discoverable signal (they live in the phase-queue file that
session-start reading-order points to); reproducing the same
signal inside CLAUDE_CODE_PROMPTS.md duplicates the source of
truth and dates fast when phases reshuffle. The 2026-05-26
Prompt-2.1 cleanup already retired one instance of the pattern
without negative consequences. Apply to `/onboard`'s spec so
future-onboarded downstream projects don't reproduce the
pattern. Existing landed prompts (1.1, 1.2) keep their header
blocks as historical record (per the "don't edit landed prompts
retroactively" convention); forward-looking Prompt 2.2 and
Prompt 3 lose theirs as part of this disposition's landing
edits.

Downstream implications named per composition rule: if option A
lands, `/onboard.md` spec is edited; CLAUDE_CODE_PROMPTS.md
Prompts 2.2 and 3 lose their header blocks; the OQ entry moves
to `docs/design-decisions.md` as a resolved decision. Option B
(rewrite as gate-check) and Option C (hybrid) are surfaced for
visibility but not recommended.

> AUDIT NOTE — JAH:
> _Accepted._

### Notes — acceptable now, revisit later

#### N1. Source-mode auto-detection — false-positive risk

**Sources.** `docs/design-decisions.md` "Source-repo refresh
syncs local cc-template/ → root" ("Detection is trivial: the
source-of-truth repo is the only one with a cc-template/ subdir
at cwd"); `docs/REQUIREMENTS.md` FR-13.

**Problem.** Source-mode is triggered by the presence of a
`cc-template/` subdirectory at cwd. The framing assumes this only
happens in the source-of-truth repo and in vendored-lock-in
setups (both intentional). False-positive scenario: a consumer
who, for unrelated reasons (training material, a folder named
after another tool that happens to share the name, a paused
migration), has a `cc-template/` subdir in their downstream
project. Refresh would treat that subdir as upstream and merge
its contents to root. The error mode is recoverable (git revert)
but disorienting.

**Recommendation.** Accept the risk for v1; it's vanishingly
rare in practice (the directory name is specific enough that
incidental collisions are unlikely). On the first source-mode
invocation in a given repo, refresh could surface a one-line
confirmation ("Detected `cc-template/` subdir at cwd; treating
as upstream. Y/N?") so an accidental collision is caught before
files move. Don't add a flag to disable auto-detection — that
would foreground a corner case the consumer doesn't need to
think about (per progressive disclosure).

> AUDIT NOTE — JAH:
> _DECISION: We're smart enough to know if a cc-template subdirectory IS or IS NOT an instance of our own code. First time we see it we can check the contents to determine if it's us or a random naming collision and save that informatino to CLAUDE.md_

## What the design got right (preserve)

- **Inverse primitives — TEMPLATE-BLOCK + ONBOARD-FILL.**
  Template-owned and consumer-owned content each have their own
  marker convention; refresh respects the boundary in both
  directions. The design-decisions entry explicitly names the
  consumer contract (don't edit inside, move-out to opt-out,
  inline edits surface) before specifying the implementation —
  contract-first thinking that prevents the failure modes of the
  rejected proposals (LOCAL markers consumers ignore; diff-only
  without a baseline that confuses "new" with "deleted").
- **Skills-first drift staging is auto-detected, not flagged.**
  The escape-hatch flag (`--refresh-skills-only`) exists as a
  manual override for power users; the default path detects
  drift and stages automatically. This matches the
  `feedback_progressive_disclosure_escape_hatches` memory:
  consumers shouldn't have to know to reach for the flag.
- **Single-stage default (revised from earlier two-stage).** The
  earlier `pip install --upgrade pip` two-stage framing assumed a
  cached-in-memory command runtime that doesn't apply to Claude
  Code commands (read fresh from disk each invocation). The
  2026-05-26 revision corrects the framing and lands on a KISS
  single-stage default with auto-detected staging when needed.
- **Source-mode unification with vendored-lock-in.** Same code
  path serves three relationships (root↔dist, vendored↔root,
  public↔downstream). One mental model rather than three;
  avoids a fourth configuration-ritual command for the
  source-of-truth's manual sync problem.
- **"Executing session is the semantic conflict detector."** The
  contract explicitly names semantic-conflict surfacing as the
  Claude session's job, not a heuristic engine the command spec
  builds. Matches `feedback_ai_not_heuristic`: AI is session
  work, not a runtime heuristic.

## Next steps

1. Jamie marks each finding inline (AUDIT NOTE blocks above).
   B1 expects a `DECISION: <A/B/C/D>` at minimum.
2. Re-run `/design-review`. Stage 2 walks each disposition,
   appends rows to the Disposition log, and asks whether to land
   the doc or open Addendum 1 (e.g. if B1 needs further research
   on a specific algorithm option before the disposition is
   final).
3. After landing, the plan-mode → `/wind-down` user story session
   (per Jamie's note in S1.3 Q4) runs as a separate cleanup
   session — surfaced as a TODO at Stage 2 landing.
4. Phase 2.1 build session opens with Prompt 2.1 in
   `docs/CLAUDE_CODE_PROMPTS.md`, now armed with concrete
   dispositions for algorithm, marker syntax, consumer-edit
   matrix, CLAUDE.md merge boundary, drift-detection mechanism,
   and the header-block disposition (applied to `/onboard`'s
   spec and to Prompts 2.2 / 3).

## Disposition log

*Stage 2 fills this section. Empty until Stage 2's first walk.
Rows accumulate append-only across rounds — later-round
dispositions on the same finding ID supersede earlier ones for
the purposes of Stage 2's landing application.*

| Round | Finding | Disposition | Reason |
|-------|---------|-------------|--------|
| Original | B1 | Decision | Option D (hybrid: per-block hash + file-level commit stamp); investigation directive — source-mode interaction (maintainer / vendored-pin local cc-template) with hybrid baseline needs resolution. |
| Original | R1 | Decision | Marker `<!-- CC-TEMPLATE-BLOCK: <id> --> ... <!-- /CC-TEMPLATE-BLOCK -->`; `<id>` is a stable human-readable kebab-case identifier; per-block hash carried as a separate `hash=` field on the open marker. |
| Original | R2a | Decision | Baseline metadata lives within the skill itself (not in user-facing files); unrecognized local block IDs surface as a `--refresh-skills-only` suggestion; specific storage mechanism deferred to Addendum 1 alongside B1 + R4. |
| Original | R2b | Accepted | Inline-edit conflicts surface inline to the running session, one block at a time: accept upstream / keep downstream / hand-merge. |
| Original | R2c | Accepted | Refresh matches blocks by id, not by position; consumer reordering preserved across refreshes. |
| Original | R2d | Accepted with Caveats | Default = no cross-file tracking (deletion + new content). Caveat: when a moved block's hash matches a known template block in one of the template-managed rules files this project ships (today's six: coding-session-rules / design-philosophy-rules / multi-agent-rules / project-rules / environment-rules / testing-rules), refresh detects the migration and offers to apply upstream's update. Scope explicitly excludes arbitrary `.md` files the consumer adds. |
| Original | R3 | Accepted with Caveats | Partition pinned (Collaboration-rules + Reading-order = template-owned; banner / project-specific / Load-bearing invariants = consumer-owned). Caveat: downstream fetches upstream's `refresh-from-repository.md` and checks whether refresh is necessary before running; upstream can force `--refresh-skills-only` via a signal in that file. Mechanism overlaps with R4 — resolved together in Addendum 1. |
| Original | R4 | Decision | Drift detection is required; storage mechanism deferred to Addendum 1 discovery — candidates include a sidecar version file (e.g. `.claude-code-sdlc-template-version.md`), a version number in `cc-template/README.md`, a pre-commit-generated incrementing id per skill, or a cleaner KISS option TBD. |
| Original | R5 | Accepted | Retire the "Before running this prompt" header-block-note form from `/onboard`'s spec; remove from forward-looking Prompts 2.2 and 3 at landing; landed Prompts 1.1 and 1.2 keep historical-state blocks per the "don't edit landed prompts retroactively" convention. |
| Original | N1 | Decision | First source-mode invocation content-inspects the `cc-template/` subdir to determine whether it's our own template; cache the determination in CLAUDE.md so subsequent invocations don't re-check. |
| Addendum 1 | B1-A1 | Accepted | Source-mode preserves three-way semantics; baseline reference for source-mode comes from R4-A1 storage (subtree content-hash or source-repo HEAD at sync time). Same algorithm in public and source modes; only the "fetch baseline" step branches. |
| Addendum 1 | R1-A1 | Accepted | Marker carries id only, no inline metadata: `<!-- CC-TEMPLATE-BLOCK: <id> --> ... <!-- /CC-TEMPLATE-BLOCK -->`. Per-block hashes live in R4-A1 storage. |
| Addendum 1 | R4-A1 | Decision | Candidate 6 with filename `.claude/claude-code-sdlc-template-refresh-state.md` (namespace-disambiguated to avoid collision with other tools that may use `refresh-state.md`). Single file carries upstream baseline + per-block hash map + drift-version stamp + upstream directives. |
| Addendum 1 | R6 | Accepted | Three coupled prongs land: coarse-grained wrapping (one TEMPLATE-BLOCK per top-level rule), per-block hashes in R4-A1 storage (not inline in markers), compact kebab-case ids. Bounds added line count to ~5-7% of post-Phase-1.2 baseline. |

## Sign-off Summary

*Filled when Stage 2 lands the doc. Jamie does not edit this
section — sign-offs go inline above.*

| ID | Final disposition |
|----|-------------------|
| B1 / B1-A1 | Decision: Option D (hybrid: per-block hash + file-level baseline reference); source-mode preserves three-way semantics. |
| R1 / R1-A1 | Decision: marker `<!-- CC-TEMPLATE-BLOCK: <id> --> ... <!-- /CC-TEMPLATE-BLOCK -->`, id-only (kebab-case, file-scoped); per-block hashes live in state file. |
| R2a | Decision: baseline metadata lives in `.claude/claude-code-sdlc-template-refresh-state.md` (not in user-facing files); unrecognized local block IDs suggest `--refresh-skills-only`. |
| R2b | Accepted: inline-edit conflict surfaces inline, one block at a time, three resolution choices (accept upstream / keep downstream / hand-merge). |
| R2c | Accepted: refresh matches blocks by id, not by position; consumer reordering preserved. |
| R2d | Accepted with Caveats: default = no cross-file tracking; caveat — when a moved block's hash matches a known template block in one of the six template-managed rules files, refresh detects and offers to apply upstream's update. |
| R3 | Accepted with Caveats: CLAUDE.md partition pinned (Collaboration-rules + Reading-order template-owned; banner + project-specific + Load-bearing invariants consumer-owned); caveat — downstream pre-flight checks upstream's refresh state before running; upstream can force `--refresh-skills-only` via a directive in the state file. |
| R4 / R4-A1 | Decision: storage = `.claude/claude-code-sdlc-template-refresh-state.md` (Candidate 6, namespace-disambiguated); single file carries Upstream baseline + Block hashes + Refresh logic version + Upstream directives sections. |
| R5 | Accepted: retire "Before running this prompt" header-block-note form from `/onboard`'s spec; remove from forward-looking Prompts 2.2 and 3 at landing; landed prompts keep historical-state blocks. |
| R6 | Accepted: coarse-grained wrapping (one TEMPLATE-BLOCK per top-level rule) + per-block hashes in state file (not inline in markers) + compact kebab-case ids. Bounds added line count to ~5-7% of post-Phase-1.2 baseline. |
| N1 | Decision: first source-mode invocation content-inspects `cc-template/` subdir to confirm it's this template; cache determination in CLAUDE.md. |

## Follow-up actions landed

**Doc edits applied (Stage 2 edit scope):**

- B1 / B1-A1 / R1 / R1-A1 / R2b / R4 / R4-A1 / N1: edited `docs/REQUIREMENTS.md` FR-13 — added Option D algorithm, pinned CC-TEMPLATE-BLOCK marker syntax, state file location, inline-edit UX, source-mode content-inspection caching, source-mode three-way semantics.
- R1 / R1-A1 / R4 / R4-A1: edited `docs/REQUIREMENTS.md` NFR-4 — pinned the template-marker syntax (was "TBD by Phase 2.1") and added the state file path + schema as load-bearing.
- B1 / B1-A1 / R1 / R1-A1 / R2b / R2d / R3 / R4 / R4-A1 / R6 / N1: edited `docs/PROJECT_PLAN.md` Phase 2.1 deliverables — pinned algorithm, marker syntax, state file, R6 granularity, R3 partition, source-mode three-way + content-inspection, inline-edit conflict UX, cross-file migration upgrade-offer.
- edited `docs/PROJECT_PLAN.md` "Design Review Checkpoint — pre-Phase-2.1" header — flipped to `LANDED 2026-05-26 (checkpoint 002)` with one-line summary of what landed.
- B1 / B1-A1 / R1 / R1-A1 / R2b / R2d / R3 / R4 / R4-A1 / R6 / N1: edited `docs/CLAUDE_CODE_PROMPTS.md` Prompt 2.1 — Read-first adds checkpoint 002; Scope rewritten to reflect all pinned decisions; deviation footer entry added.
- R5: edited `docs/CLAUDE_CODE_PROMPTS.md` Prompt 2.2 — removed "Before running this prompt" header block; deviation footer entry added.
- R5: edited `docs/CLAUDE_CODE_PROMPTS.md` Prompt 3 — removed "Before running this prompt" header block; deviation footer entry added.

**TODOs surfaced (out of Stage 2 edit scope; queued for separate sessions):**

- **T1.** Update `docs/design-decisions.md` refresh entries to reflect checkpoint 002 dispositions: Option D algorithm specifics (B1 / B1-A1); CC-TEMPLATE-BLOCK marker syntax (R1 / R1-A1); state file format and rationale (R4 / R4-A1, R2a); R6 coarse-grained granularity strategy; R2d cross-file hash-match upgrade-offer; R3 pre-flight check + upstream force-skills-only; N1 source-mode content-inspection caching.
- **T2.** Update `.claude/commands/onboard.md` to retire the "Before running this prompt" header-block-note instruction from its `docs/CLAUDE_CODE_PROMPTS.md` authoring step (R5 spec edit). Mirror to `cc-template/.claude/commands/onboard.md`.
- **T3.** Move resolved `docs/open-questions.md` entries to `docs/design-decisions.md` as closed: "Reconciliation algorithm for `/refresh-from-repository`" (closed by B1 / B1-A1) and "'Before running this prompt' header-block pattern in CLAUDE_CODE_PROMPTS.md" (closed by R5).
- **T4.** Plan-mode → `/wind-down` user-story session (carryover from prior TODO; queued before Phase 2.1 build session).
- **T5.** Add a new `docs/open-questions.md` "Deferred User Stories" entry: closing a `/design-review` (Stage 2 landing) should also review the overall project plan and, if a high-risk transition is approaching, insert a future `/design-review` checkpoint into `docs/CLAUDE_CODE_PROMPTS.md` as a *first-class entry* between prompts (the `## Design Review Checkpoint — pre-Prompt-N` shape already used for pre-Phase-2.1). Keeps design reviews in the prompt flow as first-class items that surface in TODO.txt at session start. Distinct from the header-block-note form retired by R5 — first-class entries are discoverable via reading-order, not embedded in prompt bodies.

---

## Addendum 1 — 2026-05-26

### Why this addendum exists

Round 1 closed with three threads asking for further investigation
before final disposition: **B1** (Option D hybrid's behavior under
source-mode and vendored-lock-in), **R4** (drift-detection
metadata storage mechanism — Jamie's caveat made the question
explicit: "what is the KISS-cleanest option here?"), and **R1**
(final marker syntax, downstream of B1 + R4). Two Round 1 caveats
— R2a's "baseline within the skill itself" and R3's "downstream
checks upstream first; upstream can force `--refresh-skills-only`"
— fold into R4's scope as mechanism details.

Addendum 1 re-opens those three findings with concrete options
analysis and adds one new finding (R6) on a concern Jamie surfaced
during the disposition walk: would marker insertion in rules
files create significant context-window bloat that erodes the
gains from Phase 1.2's cleanup pass?

Scope stays inside the existing `/refresh-from-repository`
contract: mechanism resolution, not new design directions.

### Round 1 dispositions that stand (not re-opened here)

- **R2b** — Inline-edit conflict UX (one-block-at-a-time inline;
  accept upstream / keep downstream / hand-merge). Final.
- **R2c** — Block matching by id, not by position; reordering
  preserved. Final.
- **R2d** — Cross-file migration: no default tracking; caveated
  upgrade-offer when a moved block's hash matches one of the six
  template-managed rules files this project ships. Final.
- **R2a partition half** — "Don't pollute user files with baseline
  metadata; unrecognized local block IDs trigger a
  `--refresh-skills-only` suggestion." Direction final; the
  *where exactly does metadata live* part is R4-A1.
- **R3 partition half** — CLAUDE.md template/consumer partition
  pinned (Collaboration-rules + Reading-order = template-owned;
  banner / project-specific / Load-bearing invariants =
  consumer-owned). The pre-flight-check caveat is R4-A1.
- **R5** — Retire "Before running this prompt" header-block-note
  from `/onboard`'s spec; remove from forward-looking Prompts 2.2
  and 3 at landing; landed prompts keep historical-state blocks.
  Final.
- **N1** — Source-mode auto-detection via content inspection; cache
  in CLAUDE.md. Final.

### Investigation summary — metadata storage mechanism candidates

Six candidates from the Round 1 walk and Jamie's reflection.
Evaluated on KISS, progressive disclosure (markers shouldn't
intrude on the rules-reading experience), R6 context-bloat
impact, and source-mode behavior (does the same primitive work
when upstream is local on disk?).

- **Candidate 1 — `.claude-code-sdlc-template-version.md`
  sidecar at repo root.** Single file at repo root carries
  upstream commit + per-block hash map. *Pro:* single source of
  truth; rules stay marker-id-only. *Con:* new top-level file
  consumers wonder about; commits into git history every refresh.
- **Candidate 2 — Version line in `cc-template/README.md`.**
  Reuse existing file. *Pro:* no new artifacts. *Con:* one signal
  only (commit), can't hold per-block hashes; breaks when consumer
  renames the `cc-template/` directory (the FR-2 default flow).
- **Candidate 3 — Pre-commit-generated id per skill.** Source-repo
  hook bumps an integer per command file on modify. *Pro:* no
  consumer-side state. *Con:* requires hook infrastructure
  (project is markdown-only today); misses unhook-bypassed edits;
  doesn't carry per-block hashes.
- **Candidate 4 — `<!-- REFRESH-LOGIC-VERSION: N -->` in
  `refresh-from-repository.md` only** (Round 1 R4 recommendation).
  Solves drift detection only; forces per-block hashes into
  markers (worst for R6) and per-file baseline-commit into a
  separate stamp somewhere.
- **Candidate 5 — Header block inside
  `refresh-from-repository.md` itself** ("within the skill
  itself"). Skill becomes a state carrier. *Pro:* no new files;
  source-mode identical. *Con:* skill self-modifies on every
  refresh → diff noise on the command file; drift detection has
  to ignore the state block when comparing for logic changes.
  - **Variant 5b** — separate `refresh-state.md` peer file in
    `.claude/commands/`. Resolves the self-modifying concern;
    costs one extra file in `.claude/commands/`.
- **Candidate 6 — `.claude/refresh-state.md`.** Single file in
  the conventional machine-state location (`.claude/` already
  hosts commands and settings). Markdown-table format keeps
  NFR-6 markdown-only. Carries upstream baseline + per-block hash
  map + drift-version + upstream directives. *Pro:* single source
  of truth; rules stay marker-id-only (R6 solved); source-mode
  identical; consumers rarely look in `.claude/`. *Con:* slight
  format choice (markdown table vs other) but stays inside
  NFR-6.

**Synthesis.** Candidate 6 and Variant 5b both meet the criteria
cleanly (metadata outside user-facing files, source-mode
identical, single source of truth). Candidate 1 also works but
puts the file at repo root where it'd be ambient. Candidates 2,
3, 4 leave per-block hashes in markers and don't resolve R6.

### How to mark up Addendum 1

Each finding below has its own `AUDIT NOTE — JAH:` block.
Replace each placeholder line per the legend at the top of the
document. Addendum 1 dispositions supersede earlier rounds for
re-opened findings (B1-A1 supersedes B1; R1-A1 supersedes R1 on
final field decisions; R4-A1 supersedes R4 on storage-mechanism
specifics).

### Findings

#### Blockers (re-opened + new)

##### B1-A1. Source-mode + vendored-lock-in behavior under Option D [re-opened from B1]

**Sources.** Round 1 B1 (DECISION: Option D); Round 1 B1 caveat
("How does this work with a local cc-template such as a maintainer
version or one that intentionally pins a version by having a
cc-template copied into their local repo? Need to resolve.");
`docs/design-decisions.md` "Source-repo refresh syncs local
cc-template/ → root" and "Vendored-template lock-in as a documented
use case"; `docs/REQUIREMENTS.md` FR-13 source-mode clause.

**Problem.** Option D (hybrid: per-block hash + file-level commit
stamp) was specified against the public-upstream-via-GitHub model
— refresh fetches the commit's version of each file as the
baseline for three-way comparison. Source-mode (root↔dist sync or
vendored-lock-in) has no GitHub round-trip; baseline content sits
on disk in `./cc-template/`. Two readings of how the algorithm
should behave:

1. **Degenerate to copy-if-changed.** In source-mode, treat the
   local `cc-template/` as both upstream AND baseline. Refresh
   copies files where upstream-cc-template differs from
   downstream-root; no three-way comparison. Simpler, but loses
   the ability to detect "consumer edited the synced file at
   root since the last sync."
2. **Preserve three-way semantics.** The local `cc-template/` is
   upstream; baseline comes from the last successful refresh's
   stored state (whatever R4-A1's storage holds). Source-mode
   keeps the same inline-edit detection that Option D was picked
   for. Costs the R4-A1 storage mechanism in source-mode too
   (it'd be required anyway for the public-upstream case).

**Recommendation.** Reading 2 — preserve three-way semantics in
source-mode. The R4-A1 storage stores the baseline reference; in
source-mode that reference is whatever identifies the local
`cc-template/`'s state at last sync (a content-hash of the subtree
or the source repo's HEAD commit at sync time — exact choice is
downstream of R4-A1). Source-mode and public-mode share the same
algorithm; only the "fetch baseline" step branches (network for
public, disk for source). The inline-edit detection that
motivated Option D over Option B works identically in both modes.

> AUDIT NOTE — JAH:
> _Accepted._

#### Recommendations (re-opened + new)

##### R1-A1. Final marker syntax — fields and compactness [re-opened from R1]

**Sources.** Round 1 R1 (DECISION: `<!-- CC-TEMPLATE-BLOCK: <id> --> ... <!-- /CC-TEMPLATE-BLOCK -->`, kebab-case id + separate `hash=` field). Now downstream of R4-A1 (storage location) and R6 (context-bloat).

**Problem.** Round 1 R1's marker shape is settled at the
conceptual level. What's open is whether the open marker carries
the `hash=` field inline or pushes the hash into R4-A1's storage.
Inline `hash=` adds ~12-16 characters per marker per block;
pushed-to-storage keeps markers minimal
(`<!-- CC-TEMPLATE-BLOCK: rule-1 -->`).

**Recommendation.** Conditional on R4-A1: if R4-A1 picks a storage
mechanism that can hold per-block hashes (Candidates 1, 5b, or
6), pin the marker as:

```
<!-- CC-TEMPLATE-BLOCK: <id> -->
...block content...
<!-- /CC-TEMPLATE-BLOCK -->
```

— id only, no inline metadata. Storage carries the {id → hash,
file → baseline} map. If R4-A1 picks a mechanism that can't carry
hashes (Candidates 2, 3, 4), fall back to inline `hash=` and
accept R6's context-bloat impact. Decision here is essentially
"trust R4-A1's pick to enable the minimal marker shape."

> AUDIT NOTE — JAH:
> _Accepted._

##### R4-A1. Metadata storage mechanism — pick a candidate [re-opened from R4]

**Sources.** Round 1 R4 ("Accepted with Caveat: Couldn't metadata
be stored in a `.claude-code-sdlc-template-version.md` file with
the commit number in the commands directory? what is the KISS
option here that we're missing?"); Round 1 R2a caveat ("don't
pollute user readable files with those. Chose a header or footer
within the skill itself"); Round 1 R3 caveat ("upstream-driven
force `--refresh-skills-only`; downstream checks upstream before
running"); investigation summary above (six candidates).

**Problem.** The storage mechanism carries four pieces of state:
(a) last-synced upstream baseline reference, (b) per-block-id →
hash map for inline-edit detection, (c) drift-detection version
stamp for refresh logic itself, (d) optional upstream-set signal
that forces `--refresh-skills-only` on next downstream run. All
four should live in one place (single source of truth) and outside
user-facing content (R6 context-bloat).

**Recommendation.** Pick **Candidate 6** — a single
`.claude/refresh-state.md` (markdown table format to stay inside
NFR-6) carrying all four pieces of state. Format sketch:

```markdown
<!-- machine-managed; refresh-from-repository writes this file. Do not hand-edit. -->

## Upstream baseline

- last-synced-commit: <sha-or-subtree-hash-for-source-mode>
- last-synced-at: 2026-05-26

## Block hashes

| File | Block ID | Hash |
|------|----------|------|
| rules/coding-session-rules.md | rule-1-fix-root-causes | abc12345 |
| ... | ... | ... |

## Refresh logic version (drift detection)

- local-refresh-logic-version: 2

## Upstream directives

- force-skills-only-from: <upstream-commit-or-empty>
```

Rationale: `.claude/` is the conventional machine-state location;
markdown stays inside NFR-6; single file makes storage atomic per
refresh; rules files stay free of hash metadata (R6 solved);
source-mode reads/writes the same file regardless of whether
upstream is GitHub or local `cc-template/`. The pre-flight
upstream check from R3's caveat reads the "Upstream directives"
section.

Alternative: **Variant 5b** (peer `.claude/commands/refresh-state.md`)
— also acceptable; co-locates state with the skill rather than at
`.claude/` root. Choose between Candidate 6 and Variant 5b at
markup via DECISION. Candidates 1-5 (non-variant) have the
drawbacks named in the investigation summary.

> AUDIT NOTE — JAH:
> _DECISION: Candidate 6 but name the file .claude/claude-code-sdlc-template-refresh-state.md to avoid namespace collision_

##### R6. Context-bloat impact of template-marker insertion in rules files [new]

**Sources.** Phase 1.2 deviation footer (LANDED 2026-05-25 with
-11.7%, short of the 25% reduction target — "load-bearing content
density was the limit"); `CLAUDE.md` Collaboration-rules section
("MUST DO before responding to the first user message of any
session" — `coding-session-rules.md` + `design-philosophy-rules.md`
read end-to-end every session); `docs/REQUIREMENTS.md` NFR-6
(markdown-only); the six template-managed rules files this
project ships.

**Problem.** Phase 1.2 cut what it could from rules content and
came up short of target because content was already dense. Adding
template-marker wrappers around refresh-managed blocks re-inflates
the line count. Per-session this cost is paid every time rules
files are read (mandatory per CLAUDE.md). Coarse estimate: ~9
rules in `coding-session-rules.md`, ~3-4 sections in
`design-philosophy-rules.md`, ~4-6 sections each across the four
placeholder rules files. Coarse-grained wrapping (one block per
rule) adds 2 lines per block; fine-grained wrapping (per
paragraph) multiplies that 3-5x. Inline `hash=` per marker grows
each line further.

**Recommendation.** Three coupled prongs that together bound the
bloat impact:

1. **Coarse-grained wrapping** — one TEMPLATE-BLOCK per
   top-level rule section, not per paragraph. Keeps marker count
   to ~25-30 across all six rules files.
2. **Per-block hashes in R4-A1 storage, not inline in markers**
   (already covered by R1-A1's conditional). Per-marker overhead
   shrinks from ~80 chars to ~40 chars.
3. **Compact id naming** — kebab-case but short
   (`rule-1-fix-root-causes`, not
   `coding-session-rule-1-fix-root-causes-only`). Ids are
   file-scoped per R1, so file prefixes are implicit.

Combined, these keep R6's added line count under ~60 across the
six rules files (~5-7% of post-Phase-1.2 baseline) — bounded
trade-off for refresh capability. The three prongs are a single
disposition: pick all three or accept materially higher bloat
(in which case a separate finding should re-open the question).

> AUDIT NOTE — JAH:
> _Accepted._

#### Notes (re-opened + new)

*No findings in this tier in Addendum 1.*
