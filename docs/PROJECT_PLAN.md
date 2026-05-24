# Project plan

Phased plan for `claude-code-sdlc-template`. Derived from
[`docs/ARCHITECTURE.md`](ARCHITECTURE.md) and the design intake
at [`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md).

Phases close with `/exit-test-plan` (manual walkthrough against the
phase's exit criteria) when the phase ships user-observable
behavior. High-risk transitions between phases also get a
`/design-review` checkpoint — see the markers below.

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
infrastructure. No `/exit-test-plan` walkthrough warranted.

---

## Phase 1 — Onboard the source-of-truth (IN PROGRESS)

**Goal.** Run the configuration ritual on this project so the
source-of-truth has its own REQUIREMENTS / ARCHITECTURE /
PROJECT_PLAN / CLAUDE_CODE_PROMPTS, filled rules `ONBOARD-FILL`
blocks, and `ONBOARD-STATUS: COMPLETE`. Validates the configuration
chain end-to-end on a real project.

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
- `/bootstrap` run on the source — README "Developer setup"
  section written, `<!-- ONBOARD-FILL: environment -->` block
  filled, `BOOTSTRAP-STATUS: COMPLETE`.

**Exit criteria.** `ONBOARD-STATUS` and `BOOTSTRAP-STATUS` both
read `COMPLETE <date>` in root `CLAUDE.md`. The four planning docs
exist and reflect the design intake without contradiction. The
`CLAUDE.md` merge survives — no template-improvement content
(load-bearing invariants, design-principles, testing-template-
changes, migrating-older-versions) was lost. Manual phase-exit
walkthrough is light here — the deliverable is documents, not
behavior.

**Design review checkpoint:** before Phase 1.1 begins. Run
`/design-review`.

---

## Phase 1.1 — `/exit-test-plan` BLOCKED disposition (deficiency)

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
`/exit-test-plan` against an artificial blocker case during the
exit walkthrough.

---

## Phase 2.1 — `/refresh-from-repository` (roadmap)

**Goal.** Build the two-stage downstream update command that lets
consumers pull the latest commands and rules from the public
upstream without re-running `/onboard`.

**Deliverables.**
- `cc-template/.claude/commands/refresh-from-repository.md`
  written as a two-stage self-modifying command.
- Stage 1: selectively pulls latest `.claude/commands/*.md` and
  `rules/*.md` from
  `github.com/JamieAnneHarrell/claude-code-sdlc-template`.
  Replaces the command files themselves, including
  `/refresh-from-repository`.
- Stage 2 (`--merge`): runs the freshly-updated logic. Diffs the
  downstream's `CLAUDE.md` and `rules/*.md` against the upstream
  `cc-template/` subtree and surgically merges deltas while
  preserving `<!-- ONBOARD-FILL: ... -->` blocks.
- CLAUDE.md merge strategy resolved (see
  [`docs/open-questions.md`](open-questions.md)).
- Refusal logic when invoked against the source repo (pulling
  from itself is a footgun).
- Documentation: shipping README explains how downstream consumers
  use `/refresh-from-repository`; this project's
  `docs/CLAUDE_CODE_PROMPTS.md` Prompt 2.1 records the prompt that
  built it.

**Exit criteria.** A test downstream project (sandbox copy of
`cc-template/`) seeded from an earlier commit successfully pulls
the latest commands and rules via stage 1, then `--merge` lands
deltas in `CLAUDE.md` and `rules/*.md` without trampling
`ONBOARD-FILL` content. Refusal triggers correctly when run
against the source repo. Manual `/exit-test-plan` walkthrough
covers happy path + at least two failure modes (upstream
unreachable; merge conflict in `ONBOARD-FILL`-adjacent text).

**Design review checkpoint:** before Phase 2.1 begins.

## Design Review Checkpoint — pre-Phase-2.1

Run `/design-review`. Trigger: pre-Phase-2.1 scrutiny —
`/refresh-from-repository` introduces a two-stage self-modifying
command, a merge strategy for `CLAUDE.md` (open question today),
and the first piece of network-touching behavior in the template.
This is the highest-risk transition in the roadmap; a checkpoint
before scope freezes is cheap insurance.

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
consistent with the project's documentation conventions. Manual
`/exit-test-plan` walkthrough covers the happy path.

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
