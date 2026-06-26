# Project plan

Phased plan for `claude-code-sdlc-template`. Derived from
[`docs/ARCHITECTURE.md`](ARCHITECTURE.md) and the design intake
at [`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md).

Phases currently never close with `/exit-test-plan`. High-risk transitions 
between phases get a `/design-review` checkpoint — see the markers below.

Phase 0 was scoped before this plan was written, so its prompt is
recorded in [`docs/CLAUDE_CODE_PROMPTS.md`](CLAUDE_CODE_PROMPTS.md)
as historical record only. Phase 1 (onboarding) closes with this
file's existence. Subsequent phases extend the template's behavior.

---

## Phase 0 — Source/dist restructure (COMPLETE)

**Goal.** Restructure the repository into source-of-truth root +
`cc-template/` distributable so changes are git-tracked and the
copy-paste workflow remains intact.

**Deliverables.**
- Repo restructured into source-of-truth root + `cc-template/`
  distributable.
- All six commands (`/onboard`, `/bootstrap`, `/deployment-plan`,
  `/design-review`, `/exit-test-plan`, `/wind-down`) present at
  both locations.
- Rules and design philosophy duplicated where universal.
- Root `CLAUDE.md` and root `README.md` written as
  template-improvement guides.
- `docs/design-decisions.md` records source/dist split rationale
  and the rejected alternatives.

**Exit criteria.** Complete as of the source/dist restructure.
This phase ships no user-observable behavior — it's
infrastructure. No `/exit-test-plan` walkthrough.

---

## Phase 1 — Onboard the source-of-truth (COMPLETE)

**Goal.** Run the configuration ritual on this project so the
source-of-truth has its own REQUIREMENTS / ARCHITECTURE /
PROJECT_PLAN / CLAUDE_CODE_PROMPTS, filled rules `ONBOARD-FILL`
blocks, and `ONBOARD-STATUS: COMPLETE`. Validates the configuration
chain end-to-end on a real project.

**Checkpoint sequencing.** `/onboard` →
`/design-review` (post-onboarding sanity check, gates `/bootstrap`)
→ `/bootstrap` → Phase 1 closes → Phase 1.1 begins. The
post-onboarding `/design-review` fires *between* `/onboard` and
`/bootstrap`, not after.

**Deliverables.**
- `docs/REQUIREMENTS.md`, `docs/ARCHITECTURE.md`,
  `docs/PROJECT_PLAN.md`, `docs/CLAUDE_CODE_PROMPTS.md` written by
  `/onboard`.
- `<!-- ONBOARD-FILL: project-scope -->` block in
  `rules/project-rules.md` filled.
- `rules/multi-agent-rules.md` rewritten to explore-plus-plan
  mode.
- README stub written (Developer setup + Deployment sections as
  pointers).
- Root `CLAUDE.md` rewritten to onboarded shape; template-
  improvement content from `CLAUDE.md.bak` merged back in as a
  follow-up step.
- Dual `LICENSE`: CC BY-NC-ND 4.0 at root; MIT at
  `cc-template/LICENSE`.
- Post-onboarding `/design-review` checkpoint LANDED (gates
  `/bootstrap`).
- `/bootstrap` run on the source — README "Developer setup"
  section written, `<!-- ONBOARD-FILL: environment -->` block
  filled, `BOOTSTRAP-STATUS: COMPLETE`.

**Exit criteria (pre-bootstrap).** `ONBOARD-STATUS: COMPLETE <date>`
in root `CLAUDE.md`. The four planning docs exist and reflect the
design intake without contradiction. The `CLAUDE.md` merge
survives — no template-improvement content (load-bearing
invariants, design-principles, testing-template-changes,
migrating-older-versions) was lost. The post-onboarding
`/design-review` checkpoint has flipped to `LANDED <date>` with
its findings dispositioned across the planning docs.

**Exit criteria (post-bootstrap).** `BOOTSTRAP-STATUS: COMPLETE <date>`
in root `CLAUDE.md`. README "Developer setup" section is the
verified-working shape produced by `/bootstrap`'s verify mode.
Manual phase-exit walkthrough is light here — the deliverable is
documents, not behavior.

---

## Phase 1.1 — `/exit-test-plan` BLOCKED disposition (COMPLETE)

**Goal.** Add a `BLOCKED` disposition to `/exit-test-plan` so test
items not runnable due to a blocker discovered during a test run
can be recorded as blocked at the time, with re-testing scheduled
for the next addendum.

**Deliverables.**
- `.claude/commands/exit-test-plan.md` updated to describe the
  `BLOCKED` disposition: when it applies, how it's recorded in §4
  / §6.N run logs, how Stage 2 dispositions a BLOCKED row
  (whether it queues a cleanup coding pass, a `/design-review`
  checkpoint, or additional phases).
- `cc-template/.claude/commands/exit-test-plan.md` mirrored.
- `rules/testing-rules.md` updated if the `BLOCKED` disposition
  changes the §3 / §4 / §5 contract the rules file documents.
- Documentation in `docs/CLAUDE_CODE_PROMPTS.md` Prompt 1.1
  footer noting the change.

**Exit criteria.** Both copies of `exit-test-plan.md` describe
`BLOCKED` identically. `/exit-test-plan` Stage 1 (initial or
addendum) places `[BLOCKED]` as a valid run log placeholder in
the same shape as `[PENDING]`. Stage 2 walks blocked rows and
queues the appropriate next-step. Validated by re-running
`/exit-test-plan` against an artificial blocker case in another
project during the exit walkthrough.
---

## Phase 1.2 — Rules and CLAUDE.md cleanup pass (COMPLETE)

**Goal.** Optimize universal rules and CLAUDE.md content for
context-window cost without removing any load-bearing invariant.
Driven by checkpoint 001 findings R4a (rules context-size), R4b
(rule 7 commit-message brevity), R4c (cc-template/CLAUDE.md
placeholder qualifiers), and R2 (cc-template/CLAUDE.md
Reading-order entry for `docs/design/`).

**Deliverables.**
- Audit of current line counts across root `CLAUDE.md`,
  `cc-template/CLAUDE.md`, and all six rules files
  (`coding-session-rules.md`, `design-philosophy-rules.md`,
  `project-rules.md`, `multi-agent-rules.md`,
  `environment-rules.md`, `testing-rules.md`) with
  load-bearing-vs-trimmable classification per section.
- Targeted reduction (≥25%) of rules + CLAUDE.md content while
  preserving every named load-bearing invariant and every rule
  that downstream consumers rely on. Edits mirror to both root
  and `cc-template/`.
- Rule 7 rework so commit-message brevity is foregrounded: "subject
  + zero or one body sentence" is the default; body bullets cite
  doc IDs (FR-N, NFR-N, ARCHITECTURE §N, Prompt N) over restating
  context; multi-paragraph bodies are the rare case.
- `cc-template/CLAUDE.md` placeholder reworded per OQ option A:
  drop the "(configured projects)" parenthetical from the
  Reading-order header; rephrase the "filled in by `/bootstrap`"
  / "filled in by `/onboard`" qualifiers so they read correctly
  pre- and post-onboard.
- `cc-template/CLAUDE.md` Reading-order list gets a new
  `docs/design/` entry mirroring root `CLAUDE.md` item 5
  ("design intake plus any `/design-review` checkpoints in
  flight"), so downstream consumers' generated CLAUDE.md
  directs them to in-flight checkpoints.
- Rules-overlap pass for `design-philosophy-rules.md` vs global
  `~/.claude/CLAUDE.md`: decide per section whether to keep
  self-contained (downstream consumers don't have Jamie's
  globals) or trim with explicit "see global" notes.
- `docs/open-questions.md` placeholder-qualifiers entry marked
  resolved (or moved to `design-decisions.md`).

**Exit criteria.** Aggregate line count across rules + CLAUDE.md
(root and dist) is ≥25% lower than the pre-cleanup baseline. Every
load-bearing invariant named in CLAUDE.md is still present and
sourced. A re-read of rule 7 produces commit-message handoffs
that hit the new brevity bar. The dist's `cc-template/CLAUDE.md`
reads correctly both pre-onboard (placeholder state) and
post-onboard (rewritten state). The new `docs/design/`
Reading-order entry is present in `cc-template/CLAUDE.md`.
---

## Phase 1.3 — root↔cc-template bring-forward (COMPLETE)

**Goal.** Align this project's own root copies (`rules/*.md`,
`CLAUDE.md`, `.claude/commands/*.md`) with the cleaned-up
`cc-template/` versions. The pre-Phase-2.1 cleanup bundle (checkpoint
003 landings — architectural spine, version-freshness rule,
skills-own-rituals, REJECTED legend, triage-then-batch, etc.) landed
only in `cc-template/`; the root copies drifted behind. Aligning root
*before* Phase 2.1 means the `CC-TEMPLATE-BLOCK` markers Phase 2.1
wraps are applied to already-clean content, and this project's own
sessions run under the cleaned-up rules.

**Deliverables.**
- Root `rules/*.md`: universal (template-owned) content brought
  forward from `cc-template/rules/`; each file's `ONBOARD-FILL` block
  content preserved.
- Root `CLAUDE.md`: template-owned sections (Collaboration rules,
  Reading order — per the checkpoint 002 R3 partition recorded in
  `docs/design-decisions.md` "consumer-boundary partition") brought
  forward from `cc-template/CLAUDE.md`; project-specific sections
  (banner, Project-specific context, Load-bearing invariants)
  preserved. Surgical, not a copy — root `CLAUDE.md` is not a verbatim
  copy of the dist placeholder.
- Root `.claude/commands/*.md`: hard-copied from `cc-template/` so all
  six are byte-identical, after a pre-diff confirms root carries no
  content that never landed in `cc-template/`.

**Exit criteria.** The three fully-universal rules files
(`coding-session-rules.md`, `design-philosophy-rules.md`,
`testing-rules.md`) are identical root↔`cc-template/`. The three with
`ONBOARD-FILL` blocks (`project-rules.md`, `environment-rules.md`,
`multi-agent-rules.md`) match `cc-template/` on universal content with
root's filled blocks intact. All six `.claude/commands/*.md` are
byte-identical root↔`cc-template/` (verified by diff). Root
`CLAUDE.md` Collaboration-rules + Reading-order reflect the
`cc-template/` improvements (rule summaries stripped, rules-read
directive foregrounded) with banner / invariants / context preserved.
No `CC-TEMPLATE-BLOCK` markers introduced — that is Phase 2.1's job.

---

## Phase 2 — Planned project enhancements (roadmap)

Roadmap phases that extend the template's behavior beyond
configuration and recurring lifecycles. Each ships in the dist
and gets exercised on the source first.

---

## Phase 2.1 — `/refresh-from-repository` (SUPERSEDED by Phase 2.1.A)

**Goal.** Build the single-stage downstream update command that
lets consumers pull latest commands and merge latest rules /
CLAUDE.md from the public upstream without re-running `/onboard`,
using block-level reconciliation against template-owned markers.

**Status (2026-06-04): built, then reframed — superseded by Phase
2.1.A.** The command was built in `cc-template/` (2026-06-03) per
the Option D deliverables below. Before close-out, a **second
design review** ([checkpoint 004](design/design-review-checkpoint-004.md),
LANDED 2026-06-04) reframed the reconciliation architecture from
Option D (per-block hash + file-level baseline + state file) to
**Option A** (stateless marker-state + ask-once; see FR-13). The
Option-D deliverables below are retained as the **as-built record**
— now a documented abandoned approach — and are **not** the current
contract. The Option A rewrite is **Phase 2.1.A** below.

**Deliverables.**
- `cc-template/.claude/commands/refresh-from-repository.md`
  written per the design contract in
  [`docs/design-decisions.md`](design-decisions.md) (`Refresh-
  from-repository is single-stage with skills-first staging on
  detected drift`, `Template marks its own content; reconciliation
  tolerates divergence`, `Source-repo refresh syncs local
  cc-template/ → root`, `Refresh progressive-disclosure flags`,
  `Vendored-template lock-in as a documented use case`) and the
  checkpoint 002 dispositions.
- Template marker syntax (pinned by checkpoint 002):
  `<!-- CC-TEMPLATE-BLOCK: <id> --> ... <!-- /CC-TEMPLATE-BLOCK -->`
  with `<id>` a stable human-readable kebab-case identifier,
  no inline metadata. Load-bearing per NFR-4.
- Reconciliation algorithm: **Option D (hybrid)** per checkpoint
  002 — per-block content hash for inline-edit detection +
  file-level baseline reference for deletion-vs-never-existed
  disambiguation. Three-way comparison (downstream-current /
  upstream-current / baseline-from-state-file).
- Refresh state file `.claude/claude-code-sdlc-template-refresh-state.md`
  created per checkpoint 002 R4-A1 — single machine-managed
  markdown file with sections: Upstream baseline, Block hashes,
  Refresh logic version, Upstream directives. Refresh
  reads/writes; consumers don't hand-edit.
- Template markers added to refresh-managed regions of rules
  files and CLAUDE.md. **Coarse-grained wrapping** per
  checkpoint 002 R6: one TEMPLATE-BLOCK per top-level rule
  section (not per paragraph); compact kebab-case ids (file-
  scoped, file prefix implicit). **CLAUDE.md merge partition**
  per checkpoint 002 R3: Collaboration-rules + Reading-order
  sections wrap in TEMPLATE-BLOCK; banner / project-specific
  context / Load-bearing invariants stay consumer-owned (free
  regions, refresh never touches).
- Auto-detected drift → skills-first staging logic implemented;
  manual `--refresh-skills-only` flag exposed; upstream-directive
  in state file can also force `--refresh-skills-only` on next
  downstream run (per checkpoint 002 R3 caveat).
- `--no-claudemd` flag exposed.
- Source-mode behavior: detect `cc-template/` subdir at cwd and
  treat it as upstream; same code path serves the source-of-truth
  repo's root↔dist sync and the vendored-template-lock-in use
  case. **Preserves three-way reconciliation semantics** per
  checkpoint 002 B1-A1 — baseline reference for source-mode comes
  from the state file (subtree content-hash or source-repo HEAD
  at last sync). On first source-mode invocation in a repo,
  content-inspect the `cc-template/` subdir to confirm it's this
  template; cache determination in CLAUDE.md (per checkpoint 002
  N1).
- Inline-edit conflict UX per checkpoint 002 R2b: when refresh
  detects an inline-edit divergence in a TEMPLATE-BLOCK, surface
  inline to the running session, one block at a time, with three
  resolution choices (accept upstream / keep downstream /
  hand-merge). No sidecar conflict files; no git-style conflict
  markers.
- Cross-file migration upgrade-offer per checkpoint 002 R2d: when
  a consumer-deleted block's hash matches a known template block
  in one of the six template-managed rules files
  (`coding-session-rules.md`, `design-philosophy-rules.md`,
  `multi-agent-rules.md`, `project-rules.md`,
  `environment-rules.md`, `testing-rules.md`), refresh detects
  the migration and offers to apply upstream's update.
- **Pre-marker migration** per checkpoint 003 R10: when invoked
  in a downstream project that has rules files but no
  `CC-TEMPLATE-BLOCK` markers (consumer onboarded pre-Phase-2.1),
  the command performs a one-time migration — insert
  coarse-grained markers at top-level rule-section boundaries,
  seed the refresh state file with per-block hashes from current
  content, and surface block-by-block divergences via the standard
  inline-edit conflict UX. Per checkpoint 003 N5 (HTML comments
  are stripped before Claude context-load), the fine-vs-coarse-
  grained block-boundary decision is relaxed.
- Semantic conflict surfacing across template/personalized regions
  is the executing session's job (per
  `docs/design-decisions.md` "Template marks its own content").
- Documentation: shipping `cc-template/README.md` explains how
  consumers use the command (default invocation, flags, vendored-
  lock-in pattern); this project's
  `docs/CLAUDE_CODE_PROMPTS.md` Prompt 2.1 records the build
  prompt; root `CLAUDE.md` Load-bearing invariants section
  updated *after* markers exist on-disk (pinning their syntax and
  the state-file path as load-bearing).

**Exit criteria.** Sandbox downstream project (copy of an earlier
`cc-template/` commit) runs refresh cleanly: command files
wholesale-replaced; rules and CLAUDE.md block-merged against
upstream with `ONBOARD-FILL` content preserved; consumer-edited
template blocks surface as divergences rather than silent
overwrites; consumer-deleted blocks are not re-added.
Source-mode invocation at this project's root syncs `cc-template/`
contents up correctly. Drift auto-detection stages skills-first
when the locally-loaded refresh is older than upstream's current
refresh.

**Design review checkpoint:** before Phase 2.1 begins.

## Design Review Checkpoint — pre-Phase-2.1 (LANDED 2026-05-26)

Ran `/design-review`. Trigger: pre-Phase-2.1 scrutiny —
`/refresh-from-repository` introduces a self-modifying command,
a merge strategy for `CLAUDE.md`, and the first piece of
network-touching behavior in the template. Highest-risk
transition in the roadmap. Landed as
[checkpoint 002](design/design-review-checkpoint-002.md) across
Round 1 + 1 addendum — pinned Option D (hybrid) algorithm,
CC-TEMPLATE-BLOCK marker syntax,
`.claude/claude-code-sdlc-template-refresh-state.md` state file,
coarse-grained TEMPLATE-BLOCK wrapping (R6), CLAUDE.md merge
partition (R3), and the inline-edit conflict UX (R2b).

---

## Phase 2.1.A — `/refresh-from-repository` Option A rewrite (COMPLETE)

**Goal.** Rewrite the built `/refresh-from-repository` from the
Option D hash/baseline/state-file mechanism to the **stateless
marker-state + ask-once** model adopted by checkpoint 004 (B1
Option A), dissolving the seed/drift bug class so the command ships
safely for fresh consumers.

**Status (2026-06-04): COMPLETE.** Block 1 (the `cc-template/`
command rewrite + dist-side docs — deliverables 1–6 below) landed
2026-06-04, plus two in-build decisions the checkpoint left open (a
fetch→review→apply security gate; public-mode fetch via shallow
clone). Block 2 (the source-mode dogfood to root + the root
ARCHITECTURE / REQUIREMENTS NFR-4 marker-encoding pins + phase close
— deliverables 7–8) landed 2026-06-04: the dogfood ran the Step 6
pre-marker migration cleanly (30 template-owned blocks inserted in
sync, `briefing-rule` kept as `state=forked`, `collaboration-rules`
took upstream), a second refresh verified quiet, and the marker
encoding is pinned in ARCHITECTURE + NFR-4. Deviation: the planned
CLAUDE.md marker pin was **not** applied — Jamie ruled the byte-level
encoding belongs with the owning skill + design docs, not duplicated
into CLAUDE.md invariants; CLAUDE.md left clean.

**Deliverables.**
- Rewrite `cc-template/.claude/commands/refresh-from-repository.md`
  to the marker-state model: markers carry `template-owned` /
  `forked` / `removed` state; two-way compare + ask-once-and-record;
  the executing session merges; **drop** the state file, per-block
  hashes, `git hash-object` recipe, `last-synced` reference, and
  baseline. Bump the `Refresh-logic-version` stamp.
- Remove the `force-skills-only-from` read, the `## Upstream
  directives` state-file section, and the Step 7.1 clear (checkpoint
  004 R1) — the version stamp is the sole drift lever.
- Settle the `forked` / `removed` marker-state encoding — NFR-4
  pins the state vocabulary and removes the state file; this build
  pins the exact syntax.
- Supersede the affected `docs/design-decisions.md` entries
  (reconciliation mechanism, content-hashing, consumer-boundary
  partition, "state file is dogfood-generated").
- Write the hash/baseline/state-file mechanism into
  `docs/open-questions.md` § Abandoned Approaches.
- Update `cc-template/README.md` "Keeping a project up to date" to
  the Option A behavior.
- Mirror the command to root `.claude/commands/` via the source-mode
  dogfood (checkpoint 004 N2), after the briefing-rule fix is in.
- `/onboard` `briefing-rule` preservation (checkpoint 004 N1) —
  **landed at checkpoint 004 close, ahead of this phase.**
- Authoritative build prompt: **Prompt 2.1.A** in
  `docs/CLAUDE_CODE_PROMPTS.md`.

**Exit criteria.** Source-mode dogfood at this repo's root runs the
ask-once migration cleanly (markers inserted aligning to upstream;
divergent blocks surface as guided yours/take/merge; no state file
written); the surviving checkpoint 002 marker-syntax /
coarse-wrapping / CLAUDE.md-partition decisions are intact; FR-13
(Option A) is satisfied by the rewritten command. Then pin the
marker encoding into the design docs (ARCHITECTURE + REQUIREMENTS
NFR-4) and close Phase 2.1 + 2.1.A.

**Design review checkpoint:**
[checkpoint 004](design/design-review-checkpoint-004.md) (LANDED
2026-06-04, Round 1) — reframed Option D → Option A.

---

## Phase 2.2 — `/product-visioning` strategic-planning layer (COMPLETE)

**Goal.** Add the strategic-planning front-end the tactical SDLC loop
lacked: an interactive `/product-visioning` that authors the next PRD,
with `/onboard` reshaped into the PRD decomposer (first PRD or a later
movement) and `/design-review` reverted to pure review. Built
out-of-band ahead of the roadmap because a downstream project needed the
movement workflow first.

**Status (2026-06-24): COMPLETE.** The command set, the PRD /
`PRODUCT_VISION` artifact model, and supporting doc edits landed in
`cc-template/` and the source-only docs (2026-06-22). The source-mode
root-sync dogfood ran 2026-06-24: `/refresh-from-repository` added
`product-visioning.md`, updated `onboard` / `wind-down` /
`refresh-from-repository` at root, and took-upstream on the `CLAUDE.md`
`reading-order` convention-check nudge; the forked `briefing-rule` block
was un-forked back to template-owned. The working-tree line-ending
re-checkout was verified complete 2026-06-24 (index blobs all-LF, working
tree all-CRLF, status clean); the optional confirming refresh was not
needed.

**Deliverables.**
- `cc-template/.claude/commands/product-visioning.md` authored
  (interactive session → `DRAFT` `docs/design/PRD-<slug>-NNN.md`; no
  decompose, no `PRODUCT_VISION` write, no PRD-status flips).
- `cc-template/.claude/commands/onboard.md` reshaped into a dual-mode
  decomposer (first PRD = full setup + adopt the intake as
  `PRD-<slug>-001`; later movement = apply vision revisions, archive the
  prior plan, author the new `PROJECT_PLAN` + prompts, flip PRD status).
- `cc-template/.claude/commands/wind-down.md`: DRAFT-PRD safety net,
  movement-complete suggestion menu, archival/decompose re-attributed
  `/design-review` → `/onboard`.
- `cc-template/.claude/commands/refresh-from-repository.md`: one-time
  product-vision migration step removed; Step numbering reconciled (0–7).
- `cc-template/.claude/commands/design-review.md` reverted to pure
  review (Jamie owns it via git).
- Artifact model: `docs/PRODUCT_VISION.md` (onboard-owned) and
  `docs/design/PRD-<slug>-NNN.md` (product-visioning authors `DRAFT`,
  onboard flips status), sharing the `docs/project-plans/` movement
  counter.
- Supporting docs swept: root + `cc-template/` `CLAUDE.md`, ARCHITECTURE,
  REQUIREMENTS (FR-3, FR-17), both READMEs, design-decisions,
  open-questions.

**Exit criteria.** Source-mode dogfood at this repo's root syncs the
reshaped command set (adds `product-visioning.md`; updates onboard /
wind-down / refresh / design-review) and block-merges the `CLAUDE.md`
`reading-order` nudge ("take upstream"); a second refresh verifies quiet;
root and `cc-template/` command files reach parity. Until the dogfood
runs, root is intentionally one step behind `cc-template/`.

---

## Phase 2.3 — `/write-user-documentation` (roadmap)

**Goal.** Ship a command that authors end-user documentation —
used in the source-of-truth project to document itself, and used
in downstream projects by consumers to create their product
documentation.

**Status (2026-06-25): Block 1 landed; renamed and reshaped.** The
command shipped as `/write-documentation` (not
`/write-user-documentation`) with its full spec and embedded craft
doctrine — the Phase 2.3 design session is settled, not deferred.
Checkpoint 005 (2026-06-25) landed the render/build ownership
decision: `/write-documentation` owns *writing* + a product-type-open
delivery recipe + the release-readiness ledger; `/deployment-plan`
owns rendering/building the delivered docs (Option A). The remaining
work is Phases 2.4–2.7 (apply checkpoint 005 → ecosystem wiring →
refresh dogfood → write-doc dogfood + close); Block 2's
`/deployment-plan` piece grew from *referencing* a render to *owning*
the doc build. The deliverables and exit criteria below are the
Block 1 record and are not rewritten (checkpoint 005 R1).

**Deliverables.**
- `cc-template/.claude/commands/write-user-documentation.md`
  written. Spec deferred to Phase 2.3 design session.
- Mirrored at `.claude/commands/write-user-documentation.md`.
- This project's own user documentation produced by running the
  new command on itself (eats own dog food).

**Exit criteria.** Command runs against a freshly-seeded downstream
project (sandbox) and produces a user-facing README and any
supplementary docs the consumer requested. Output style is
consistent with the project's documentation conventions. Sandbox
re-read against the dist command spec is the validation
(`/exit-test-plan` is not used in this project per NFR-6).

---

## Phase 2.4 — Apply checkpoint 005 (render/build ownership) (COMPLETE)

**Status (2026-06-25): COMPLETE.** All four deliverables landed in
`cc-template/` + the supporting docs; the root command copy stays the
old shape until Phase 2.6 propagates it.

**Goal.** Reshape `/write-documentation` and the load-bearing
invariants to Option A: writing + a product-type-open delivery recipe
here, render/build owned by `/deployment-plan`.

**Deliverables.**
- `cc-template/.claude/commands/write-documentation.md` reshaped:
  `--render` + Step S2.5 removed; manifest "Render configuration" →
  product-type-open "Delivery recipe"; "single render owner" claim
  dropped; build pointed at `/deployment-plan`; ledger kept as the
  handoff signal.
- Root `CLAUDE.md` `/write-documentation` invariant flipped to
  "`/deployment-plan` owns the doc render/build, consuming the
  delivery recipe + ledger."
- `docs/design-decisions.md` "render under review" note rewritten to
  the landed Option A decision (no tombstone).
- `docs/open-questions.md` render story corrected (false ds-zip-tax
  HTML-script claim; false "this project has a runtime" assumption)
  and migrated.

**Exit criteria.** The command names no renderer/toolchain and has a
product-type-open delivery recipe; the root invariant, design
decisions, and open questions all reflect Option A; nothing prescribes
a runtime. The root command copy is intentionally still the old shape
until Phase 2.6 propagates it.

---

## Phase 2.5 — Block 2: ecosystem wiring (COMPLETE)

**Status (2026-06-25): COMPLETE.** The three siblings wired in
`cc-template/`. Beyond plan: added a `/write-documentation` pointer to
onboard's PROJECT_PLAN orientation paragraph and queued a Deferred User
Story on `/onboard` capturing documentation intent (see
`open-questions.md`).

**Goal.** Wire the three sibling commands to the doc-ownership model.

**Deliverables (in `cc-template/.claude/commands/`).**
- `deployment-plan.md` — owns the doc render/build: reads the delivery
  recipe + ledger, codes the render targets against the environment
  runtime at release time, adds a pre-release doc-gate (docs CURRENT +
  no OPEN conformance gaps + no unfilled required visuals), ships the
  built docs.
- `wind-down.md` — coherence sweep respects `/write-documentation`'s
  README / CONTRIBUTING section ownership and surfaces stale docs /
  OPEN ledger items at session close.
- `onboard.md` — one-line stale-docs reminder on movement decomposition.

**Exit criteria.** Each sibling edited in `cc-template/`; ownership
stays non-overlapping (NFR-8); `/deployment-plan` owns the build with
no parallel render defined elsewhere.

---

## Phase 2.6 — Dogfood `/refresh-from-repository` (source mode)

**Goal.** Propagate the Phase 2.4 / 2.5 command changes from
`cc-template/` to root.

**Deliverables.** A source-mode refresh run (review-before-apply);
root copies of `write-documentation.md` + the three siblings updated;
a second refresh verified quiet.

**Exit criteria.** Root commands match `cc-template/`; the second
refresh is quiet. This is the step that makes the reshaped
`/write-documentation` runnable at root — it must precede Phase 2.7.

---

## Phase 2.7 — Dogfood `/write-documentation` + close

**Goal.** Run `/write-documentation` on this template, produce
`docs/published/`, and close the write-documentation work.

**Deliverables.** `docs/published/` for the template itself (numbered
manifest + audience-facing markdown + delivery recipe + ledger);
write-documentation work marked closed in the tracking docs.

**Exit criteria.** The manifest is `ACTIVE` and the markdown reads as
audience-facing, with a product-type-open delivery recipe recorded.
No build/render is produced — this project has no runtime and
`/deployment-plan` is `UNCONFIGURED`; markdown is the deliverable. A
sandbox dry-read confirms a consumer could use the output without the
internal docs.

---

## Phase 3 — Regression-test automation (roadmap)

**Goal.** Scripted check that diffs the current `cc-template/`
dist against a tagged baseline and flags invariant-breaking
changes — status comment renames, `ONBOARD-FILL` marker drift,
zero-pad-width changes in `checkpoint-NNN` / `phase-NNN-exit`
filenames, stage-detection placeholder text drift.

**Deliverables.**
- A small source-only script (`scripts/regression-check.ps1` or
  equivalent) that runs diff against a tagged baseline.
- A baseline tag in git history (e.g. `dist-baseline-v0.1`).
- Documentation in root `CLAUDE.md` Testing section: how to run
  the check before tagging a new dist release.

**Exit criteria.** Running the check against the current dist
exits 0; intentionally breaking an invariant (e.g. renaming
`ONBOARD-STATUS` to `ONBOARD_STATUS`) causes the check to exit
non-zero and report the violation. Specifics still TBD until a
real regression motivates the work.

---

## Out of scope (post-MVP, deferred indefinitely)

- Markdown linting / style enforcement automation.
- A separate landing page or product website.
- Translations of the template into non-English locales.
- Bundling the template as an installable package (npm/pip/etc.).
- A push-model update mechanism from upstream to known downstream
  projects.
