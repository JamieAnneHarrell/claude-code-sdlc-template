# Project plan

Phased plan for `claude-code-sdlc-template`. Derived from
[`docs/ARCHITECTURE.md`](ARCHITECTURE.md) and the design intake
at [`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md).

Phases currenlty never close with `/exit-test-plan`. High-risk transitions 
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

## Phase 2 — Planned project enhancements (roadmap)

Roadmap phases that extend the template's behavior beyond
configuration and recurring lifecycles. Each ships in the dist
and gets exercised on the source first.

---

## Phase 2.1 — `/refresh-from-repository` (roadmap)

**Goal.** Build the single-stage downstream update command that
lets consumers pull latest commands and merge latest rules /
CLAUDE.md from the public upstream without re-running `/onboard`,
using block-level reconciliation against template-owned markers.

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

## Phase 2.2 — `/write-user-documentation` (roadmap)

**Goal.** Ship a command that authors end-user documentation —
used in the source-of-truth project to document itself, and used
in downstream projects by consumers to create their product
documentation.

**Deliverables.**
- `cc-template/.claude/commands/write-user-documentation.md`
  written. Spec deferred to Phase 2.2 design session.
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
