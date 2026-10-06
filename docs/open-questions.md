# Open Questions and Engineering Notes

Working surface for unresolved questions, deferred decisions,
abandoned approaches, and things worth revisiting. Update as answers
are found or decisions are made.

`/wind-down` proposes additions to this file at session end. `/onboard`
seeds it with any open questions that the design intake left
unresolved.

## How this file relates to other tracking

- **`TODO.txt`** — questions that *must* be resolved at the start of
  the next session belong there, not here. This file is for genuinely
  open / non-blocking items.
- **`design-decisions.md`** — once a question gets answered with a
  real decision, it moves there.
- **`PROJECT_PLAN.md`** — when an open question becomes scoped work,
  it becomes a phase or a phase deliverable.

## Categories

Use these section headers. Add a category section only when there's a
real entry to put under it; don't pre-create empty sections.

### Deferred User Stories
Things we've thought about and decided not to build yet. Format:
*Context* (what / why), *Proposed approach*, *Open sub-questions*.
Lift the name into PROJECT_PLAN.md when it's time to build.

### Known Limitations
Things the current implementation does NOT handle gracefully, with
the conditions that trigger the limitation and a mitigation if there
is one. Document so future debugging starts here.

### Abandoned Approaches
Things we tried, why they failed, and what replaced them. Useful when
someone (Claude or human) tries the same thing again — the file
reminds them why it didn't work.

### Open Questions
Questions we haven't yet answered, with what we know so far and what
would unblock an answer.

---

### Deferred User Stories

#### Sweep local Claude-memory items that are really project guidance into checked-in docs

*Context.* Project-level guidance (how to write / maintain this template)
was mistakenly filed into local `~/.claude/.../memory/` files, which are
per-machine and invisible to other maintainers. This session's mis-files
were cleared, but pre-existing memories carry the same issue — e.g.
`distributable-self-contained` duplicates the existing project rule "Shipped
content references no root-project artifacts"; `winddown-coherence-sweep`
and `zero-pad-counters` are project-level. Surfaced 2026-06-16.

*Proposed approach.* Audit the local memory index; for each entry decide
**project-level** (→ move into the right checked-in doc — CLAUDE.md, rules,
or ARCHITECTURE — and delete the memory) vs **genuine cross-project working
preference** (→ keep in memory). Prefer existing homes; collapse redundant
ones (`distributable-self-contained` is already covered by project-rules).

*Open sub-questions.* The line between "project-specific" and "cross-project
preference" for skill-usage habits (e.g. design-review / plan-mode working
notes) is a judgment call per item.

#### Command "does NOT do" sections duplicate the ownership map

*Context.* Each command file's "What this command does NOT do" section
re-states which files OTHER commands own ("never edits X — /design-review
owns it"). Read across all commands together, this cross-referencing is
redundant and multiplies as commands are added. Root CLAUDE.md's
ownership invariant was collapsed to "owners list + skills don't write
files they don't own"; the command files weren't. Surfaced 2026-06-15.

*Proposed approach.* Sweep every command's "does NOT do" section: keep
each command's own non-responsibilities, but replace the
per-other-command ownership restatements with a single "a skill never
writes a file it doesn't own" line pointing at the CLAUDE.md ownership
map. Mirror to `cc-template/` per NFR-9.

*Open sub-questions.* Whether each command still needs a short pointer to
the ownership map, or whether the one rule suffices.

#### Brittle hardcoded command/file counts across planning docs

*Context.* Several source-only planning docs hardcode counts that go
stale as the command/rules set grows — "six commands", "seven SDLC
commands", "six rules files". A 2026-06-15 grep found instances in
`docs/PROJECT_PLAN.md`, `docs/CLAUDE_CODE_PROMPTS.md`,
`docs/REQUIREMENTS.md`, and `docs/design-decisions.md`. Frozen artifacts
(LANDED `docs/design/design-review-checkpoint-*.md`, the
`docs/design/cc-template-product-spec.md` intake) carry the same counts
but are historical record and must not be edited.

*Proposed approach.* Sweep the non-frozen planning docs to number-free
phrasing ("the commands", "the rules files", enumerate by name when
useful). Leave frozen artifacts. The live shipped instance
(`refresh-from-repository.md` Step 7's rules-file count) and the
`REQUIREMENTS.md` FR-1 contract count are fixed in-session; this backlog
item covers the historical planning-doc records.

*Open sub-questions.* Whether definitional counts (the "10 rules", which
are literally numbered 1–10) should stay as-is — yes, they're not
brittle.

#### `/design-review` Stage 2 landing should review the project plan for future checkpoints

*Context.* When a checkpoint lands, Stage 2 could take a pass at
`docs/PROJECT_PLAN.md` to ask: "given what was just decided, are any
future high-risk transitions in the plan now worth a first-class
`/design-review` checkpoint?" If yes, insert a
`## Design Review Checkpoint — pre-Prompt-N` first-class entry into
`docs/CLAUDE_CODE_PROMPTS.md` between the relevant prompts, so design
reviews stay discoverable in the prompt flow and surface in TODO.txt at
session start. Distinct from the header-block-note form retired by
checkpoint 002 R5. Surfaced 2026-05-26 as the second thread of the
artifact-boundary routing story; the first thread (every command that
reaches a commit point invokes `/wind-down`) was decided at checkpoint
007 N7 and lands in Phase 3.

*Proposed approach.* Add a step to `/design-review`'s Stage 2 land path
that walks `docs/PROJECT_PLAN.md` for high-risk transitions and proposes
first-class checkpoint inserts when appropriate. Mirror per NFR-9.

*Open sub-questions.* Whether the "scan PROJECT_PLAN for high-risk
transitions" heuristic needs concrete criteria, or is best handled as a
session-judgment call surfaced for Jamie to disposition.

#### Regression-test automation for the distributable

*Context.* The dist is a copy-paste seed that must continue to work
end-to-end (full /onboard → /design-review → /bootstrap → ...
chain). Today regression checking is manual ("copy cc-template/ to
a sandbox dir and run the chain"). A scripted check could diff the
current dist against a tagged baseline and flag invariant-breaking
changes (status comment renames, ONBOARD-FILL marker drift,
zero-pad width changes in checkpoint/test-plan filenames).

*Proposed approach.* Phase 3 roadmap. Specifics deferred until we
have a real regression to motivate the work.

#### Seed `cc-template/TODO.txt` as the active onboarding checklist; teach the TODO-driven habit from day one

*Context.* The first thing a downstream user does is open their
freshly-copied `cc-template/` and look for "what do I do now?"
CLAUDE.md and README both name TODO.txt as the next-step surface.
Today's seeded TODO.txt has one item: "Run /onboard to configure
this project from the design doc in `docs/design/`." That makes
`/onboard` look like the literal first action, but the README
correctly says the user should review the rules first to confirm
they agree with them. That review step is silently expected, and
TODO.txt doesn't reflect it.

This compounds with the in-progress `/refresh-from-repository`
work and the CC-TEMPLATE-BLOCK marker design (the current
TODO.txt's first item). Once markers exist, the *correct*
customization sequence is: review the rules, decide which
TEMPLATE-BLOCK sections to keep verbatim vs. lift out of markers
for local edits, edit accordingly, THEN `/onboard`. None of that
is in the seeded TODO.txt today.

Two outcomes today:

1. New users skip the rules review and run `/onboard` cold; later
   discover a rule doesn't match their team with no clean
   divergence path (or, post-`/refresh-from-repository`, get
   caught off-guard by marker constraints).
2. Users don't learn the TODO-driven habit. TODO.txt is treated as
   a Claude-maintained tracking file, not a checklist humans work
   through.

*Proposed approach.* Seed `cc-template/TODO.txt` with the actual
customization-and-onboarding sequence as a numbered checklist the
user walks through:

1. Read `README.md` to understand the template shape.
2. Read each `rules/*.md` file; edit anything you disagree with
   (or, once markers ship, lift it out of its CC-TEMPLATE-BLOCK
   so `/refresh-from-repository` leaves your edits alone).
3. Drop a design doc into `docs/design/`.
4. Run `/onboard`.
5. (next session) Run `/bootstrap`.
6. (when distribution mechanism is pinned) Run `/deployment-plan`.

Update `cc-template/README.md` to describe the rule-divergence
workflow (review → edit → optionally lift from markers) and point
at TODO.txt as the working checklist. Frame TODO.txt explicitly as
"the human's checklist for this and every future session," not
just an automated handoff. Set the habit early.

*Open sub-questions.* Whether the seeded checklist should branch
by user intent ("customizing before use" vs "accepting defaults,
skip to /onboard") — risks complexity for marginal value. Whether
step 2's depth ("read every rules file") is realistic — users
often want to get to /onboard fast and only diverge later; a
softer "skim now, refine later" framing may be more honest.
Whether `/onboard` should check that TODO.txt has been walked
(e.g., does the user know they can edit rules?) or whether that's
paternalistic. Whether the seeded TODO.txt should be a sample
users can rewrite freely vs. a structured artifact whose shape
`/wind-down` preserves.

#### Align `/onboard`-owned startup surfaces with the product-visioning-first framing

*Context.* Surfaced 2026-06-27. The user-facing docs were revised to promote
`/product-visioning` as the primary way to start a new project, with
drop-a-design-doc demoted to an equal-but-secondary supported path (quick-start,
both READMEs, the `/onboard` + `/product-visioning` command-reference docs;
documentation-plan-001 Revise 1). But the surfaces `/onboard` and the seed own
still lead with the design-doc path and are now inconsistent: the
`cc-template/CLAUDE.md` unconfigured banner (step 1 "Confirm a design doc exists
in `docs/design/`"), the seeded `cc-template/TODO.txt` checklist, and
`/onboard`'s "No design doc in `docs/design/`" refusal copy. `/write-documentation`
can't touch these (rule 8 / file ownership), so the framing inversion stopped at
the doc boundary.

*Proposed approach.* Through the in-movement enhancement lane (plan →
`/design-review` → decompose), reword the `/onboard`-owned startup surfaces to
lead with `/product-visioning` and present drop-a-design-doc as the secondary
supported path: the `cc-template/CLAUDE.md` banner, the seeded `TODO.txt`
checklist, and `/onboard`'s refusal copy (accept "have a PRD or a design doc"
rather than design-doc-only). Resolve together with the "Seed `cc-template/TODO.txt`
as the active onboarding checklist" story above — its proposed step 3 still reads
"Drop a design doc," so the checklist should be reworked once, not twice. Mirror
per NFR-9.

*Open sub-questions.* Whether the banner/refusal change is purely framing or also
touches `/onboard`'s detection (it already reads a PRD as intake, so likely
framing-only). Whether this is a standalone enhancement or folds entirely into
the seed-TODO story's build.

#### Parameterize maintainer identity (name / pronouns / initials) for the public repo

*Context.* The repo is public, and shipped files hardcode the
maintainer's identity: "Jamie" by name, "she/her" by pronoun, and
"JAH" as initials (e.g. the `/design-review` AUDIT-NOTE marker
`AUDIT NOTE — JAH:`). A consumer who seeds a project from
`cc-template/` inherits all of it — their rules talk about Jamie,
their design reviews want Jamie's initials. Identity should be
placeholder-driven, with `/onboard` collecting the consumer's name,
pronouns, and initials and substituting them.

*Proposed approach.* Replace hardcoded identity in shipped files
with placeholder tokens; have `/onboard` ask for name / pronouns /
initials and fill them, the same way it already fills `ONBOARD-FILL`
blocks. Scope spans every `cc-template/` file that names the
maintainer: `rules/*.md`, `.claude/commands/*.md`, `CLAUDE.md`,
`README.md`.

*Open sub-questions.* Placeholder syntax — reuse the `ONBOARD-FILL`
marker family, or inline tokens like `{{MAINTAINER}}` / `{{PRONOUN}}`
/ `{{INITIALS}}`? The initials are load-bearing: `AUDIT NOTE — JAH:`
is the string `/design-review` stage detection keys decisions on, so
parameterizing initials means auditing `design-review.md` (both
copies) and `CLAUDE.md`'s invariant wording in lockstep. Whether
`/refresh-from-repository` re-collects identity on update or treats
filled identity as consumer-owned (probably consumer-owned, like
`ONBOARD-FILL`). The fallback if a consumer skips the questions (a
neutral default like "the maintainer" / "they" / a two-letter
placeholder). Whether this gates public-launch readiness — it is
what makes the repo genuinely reusable rather than personalized, so
likely high-priority once public launch is on the table.

#### `/onboard` captures documentation intent so design-review can review it (and docs become planned work, not only discovered)

*Context.* Surfaced 2026-06-25 while wiring Block 2 (Prompt 2.5).
`/onboard` plants `/design-review` as first-class phases, notes the
`/exit-test-plan` convention in the PROJECT_PLAN orientation, and injects
the secrets-hygiene NFR — but captures **no** documentation intent and
plants no documentation work. So design-review has no doc requirements to
scrutinize, and `/write-documentation` discovers the doc set cold from
specs + code. The Prompt 2.5 orientation note (a one-line pointer to
`/write-documentation` in the PROJECT_PLAN orientation paragraph) is the
minimal treatment; this story is the larger ownership question it defers.

*Proposed approach (to weigh, not decided).* Capture documentation
*intent and audiences* — not structure — at PRD/onboard time, so
REQUIREMENTS or PRODUCT_VISION anchors it and design-review can check
"did we plan for docs." Possibly a movement-close documentation step in
the plan. Must reconcile with `/write-documentation`'s deliberate
infer-don't-prescribe design ("iPhone, not Android") — the anchor is
*who needs docs*, not *what the docs are*.

*Open sub-questions.* Where intent lives — a PRD section, a
PRODUCT_VISION audience cue (positioning already implies audiences), or a
REQUIREMENTS NFR? Is `/product-visioning` the truer owner of "who are the
audiences," since it owns positioning? Does onboard plant a documentation
phase, or does it stay currency-driven/opt-in? Should `/design-review`
gain a "documentation planned?" checklist item? How does any of this
avoid re-prescribing a doc structure?

#### Regression-test automation — declassified from Phase 3

*Context.* A scripted check that diffs the current `cc-template/` dist against a
tagged baseline and flags invariant-breaking changes (status-comment renames,
`ONBOARD-FILL` marker drift, zero-pad-width changes in `checkpoint-NNN` /
`phase-NNN-exit` filenames, stage-detection placeholder drift). Was Phase 3 /
Prompt 3; checkpoint 006 R5 declassified it — it does not belong to this movement
and isn't motivated by a real regression yet. FR-16 now frames it as an
unscheduled candidate. Surfaced 2026-06-26.

*Proposed approach.* A small source-only script (`scripts/regression-check.ps1`
or equivalent) + a baseline git tag (e.g. `dist-baseline-v0.1`) + a root-`CLAUDE.md`
note on when to run it (before tagging a dist release). Detect / report /
exit-nonzero; never auto-fix. Pull into a movement when a real regression
motivates the work.

*Open sub-questions.* CI vs local-only before a dist release? Cross-platform
(PowerShell vs bash vs a both-OK tool)?

#### Vocabulary sweep: relabel the "Phase 2.x" entries as steps and align the filename families

*Context.* Checkpoint 007 R5 settled the Movement → Phase → Step vocabulary
(see `design-decisions.md` "Movement → Phase → Step vocabulary"); the definition
ships in `rules/project-rules.md` in Phase 3. What remains is the sweep: entries
written before Phase 3 are labelled "Phase 2.x" / "Prompt 2.x" for what are
Steps 2.x of Phase 2, and the skills, rules and live planning docs still use the
old words in places. Rename only — the numbers are kept. Exit testing is already
phase-grained (`/exit-test-plan` runs at the end of a phase), so its framing
survives unchanged. Surfaced 2026-06-27; scoped down 2026-10-05.

*Proposed approach.* When the drift hurts, sweep the live planning docs and the
shipped skills (`/product-visioning`, `/design-review`, `/exit-test-plan`,
`/wind-down`, `/write-documentation`, `/onboard`) onto the settled terms, and
decide which filename families (`project-plan-NNN`, `phase-NNN-exit`,
`claude-code-prompts-NNN`) inherit the new words — in particular whether
`phase-NNN-exit.md`'s `NNN` becomes a sequential counter independent of the
phase number. Frozen artifacts (LANDED checkpoints, archived plans, the design
intake) keep their original wording as historical record.

*Open sub-questions.* Whether "Product plan" vs "Project plan" is part of the
same settlement, and whether the PRD / product-vision layer needs a named level
above Movement. Whether the sweep is a tactical batch or its own in-movement
enhancement.

#### Per-movement research / intake holding area so `docs/design/` reads as current truth, not provisional exploration

*Context.* Surfaced 2026-06-29 dogfooding `/product-visioning` in an empty
project for the first time. A visioning session generates a lot of exploration
and discussion — some gets baked into the PRD, some is discarded, some is lifted
into `PRODUCT_VISION.md`. Today that provisional material has no home distinct
from current design truth: `docs/design/` mixes the live artifacts (the active
`PRD-<slug>-NNN.md`, the `design-review-checkpoint-NNN.md` set, the `REVIEWS.md`
index) with provisional / aspirational input (the original design intake —
`cc-template-product-spec.md` here — and any visioning research / notes). A
reader scanning `docs/design/` can't tell what is current from what was
superseded or rejected, and rejected paths sitting in old intake can taint
future vision (and could be re-surfaced as live options by `/design-review`'s
S1.1 whole-file read). Provenance ("where we came from") is worth keeping — it
just shouldn't sit alongside, or be mistaken for, current scope.

*Proposed approach (to weigh, not decided).* Introduce a holding area for
per-movement research / intake / onboarding docs — provisional material lands
there as provenance, separate from the current design artifacts in
`docs/design/`. First-level decision is whether to add such an area at all and
where it lives (e.g. a single `docs/design/intake/` vs. a per-movement
subfolder), keeping current truth — the active PRD, the checkpoints — as the
only thing a `docs/design/` scan surfaces.

*Open sub-questions.* One intake folder vs. a per-movement subfolder. Whether
`/product-visioning` (today it outputs only the PRD) gains ownership of writing
the holding area, or `/onboard` files the source design doc there on decompose.
How it relates to the existing `cc-template-product-spec.md` intake and
`docs/design/README.md`. Whether rejected paths should be explicitly quarantined
so `/design-review`'s Abandoned-Approaches / whole-file read doesn't re-surface
them as live options. (The internal-manifest question was closed as "do nothing"
at checkpoint 007 N3; a holding area would be one more purpose-named folder.)

---

### Abandoned Approaches

#### `/refresh-from-repository` reconciliation via per-block hash + baseline + sidecar state file (Option D)

*What it was.* The first-built mechanism for
`/refresh-from-repository` (checkpoint 002, built 2026-06-03; never
shipped). Template-owned content was wrapped in `CC-TEMPLATE-BLOCK`
markers (that part survives); each block's content was hashed with
`git hash-object` (LF-normalized for cross-platform determinism); a
sidecar `.claude/claude-code-sdlc-template-refresh-state.md` held four
sections (Upstream baseline, Block hashes, Refresh logic version,
Upstream directives). Reconciliation was a **three-way** compare —
downstream-current / upstream-current / baseline-from-state-file —
matched by id, with the per-block hash as the change-classifier and the
baseline distinguishing "block the consumer deleted" from "block that
never existed yet." Source mode used a subtree content-hash of the
local `cc-template/` as the baseline.

*Why it was abandoned (checkpoint 004 B1, 2026-06-04).* Patching the
seed path surfaced that the whole bug class lived in the storage
mechanism itself. The baseline was seeded from the wrong side
(downstream-current) in both the re-seed branch and the pre-marker
migration, so a pre-existing consumer edit would freeze into the
baseline and the *next* run would misclassify it as "downstream
untouched, upstream changed" and silently revert it. Two observations
dissolved the class: (1) the memory of "this block was deleted /
customized on purpose" can live **in the rules file itself** as a
marker state (a tombstone or a `forked` flag), set by asking once — no
sidecar baseline needed; (2) the *merge* is already the executing
session's job, so the per-block hash was only doing **classification**,
which a two-way compare plus a one-time question does without any stored
baseline. Following that thread removed the state file, the per-block
hashes, the `git hash-object` recipe, the `last-synced` reference, and
the baseline — leaving markers + the command's version stamp + the LLM.

*What replaced it.* Option A — stateless marker-state + ask-once. See
`design-decisions.md` "`/refresh-from-repository` reconciliation:
Option A". The marker syntax and coarse-grained wrapping from checkpoint
002 carried forward; the hash/baseline/state-file machinery did not.

*The accepted trade (so it is not re-litigated).* Option A is weaker
than Option D in exactly one place: **provenance**. Option D could infer
"who changed this block" from a stored baseline; Option A asks the
consumer once and records the answer in the marker, relying on the
consumer recognizing their own edit (possibly months later), with "take
upstream" the git-recoverable safe default. Jamie accepted this trade
2026-06-04. The determinism Option D preserved was **not** judged worth
its correctness-and-maintenance surface: keeping seeded/shipped hashes
correct, cross-platform LF/`hash-object` discipline, committing the
state file so the baseline survives a clean clone, and the recurring
drift risk across all of it. **Do not re-propose a stored-baseline /
per-block-hash mechanism** without new information that changes this
calculus (rule 3).

*Sibling mechanisms also considered and rejected* (at checkpoint 002,
when Option D was chosen; none revived by Option A). **Per-block-hash
alone** and **file-level-baseline alone** — each leaves
deleted-vs-never-existed ambiguous (Option A resolves that by asking
once). **git-3-way against tagged upstream releases** — forces
release-tag discipline on a one-maintainer upstream and exposes git
conflict-marker syntax to consumers who read these `.md` files every
session; Option A keeps all conflict resolution inside the running
session with no markers in the files.

---

### Open Questions

#### How the source-only `docs/user/` + `docs/maintainer/` doc set reaches adopters

*Context.* Surfaced 2026-06-25 dogfooding `/write-documentation` on the template.
The command authors its canonical set under `docs/user/` + `docs/maintainer/`, but
those paths are source-only (FR-12) and never copy into `cc-template/`, so adopters
who seed a project never receive it. The surface that *does* travel —
`cc-template/README.md` — is rewritten by `/onboard` the moment a project is
seeded. Two consequences: (1) the detailed docs have no delivery path to adopters
yet, and (2) the shipping README can't relative-link into the published doc set
without the links breaking on seed, so deep-doc routing for adopters is unresolved.

*What we know so far.* `/write-documentation` owns content + a product-type-open
delivery recipe; `/deployment-plan` owns the render/build and delivery. The recipe
in `docs/documentation-plans/documentation-plan-001.md` records the target forms
and these questions. No `docs/DEPLOYMENT.md` exists yet (`/deployment-plan` is
UNCONFIGURED).

*What would unblock an answer.* A `/deployment-plan` pass that decides how (and
which subset of) the published doc set reaches adopters — a rendered/hosted form, a
shipped subset mirrored into `cc-template/`, or an upstream-URL pointer — and how
that interacts with `/onboard` rewriting `cc-template/README.md`.

#### Portability audit — what makes *this* project work better than a freshly-seeded one?

*Context.* Consumers run the template in their own environments,
seeded only from `cc-template/` via `/onboard`. But this project also
benefits from things that **don't** ship: root `CLAUDE.md`'s
Load-bearing invariants and project context (source-only), the global
`~/.claude/CLAUDE.md` (Jamie's TODO.txt workflow, design philosophy,
migration pointers — machine-scoped), and project memory under
`~/.claude/projects/.../memory/`. Some of that is load-bearing for how
well this project runs. A seeded project gets none of it. The
question: which of those behaviors are doing real work, and which
should move into a *shipped* surface (`project-rules.md` or a skill)
so consumers inherit them?

*What we know so far.* `/onboard` seeds from `cc-template/`; root
`CLAUDE.md` invariants + project context are source-only and never
copied. Global `~/.claude/CLAUDE.md` (TODO.txt workflow,
progressive-disclosure philosophy) applies on Jamie's machine to
*every* project, so a consumer on another machine never sees it.
Project memory is local auto-memory, not shipped. The memories that
shape behavior (e.g. the repo-wide wind-down sweep, the
distributable-self-contained habit) are candidates for promotion to
shipped rules — some are already being promoted (the
self-containment habit lands as a project rule in
`rules/project-rules.md`; the repo-wide sweep is its own story
above).

*Nested sub-question — should `project-rules.md` be an always-loaded
read?* Today the session-start directive at the top of `CLAUDE.md`
mandates only `coding-session-rules.md` and `design-philosophy-rules.md`
end-to-end; `project-rules.md` is "read when relevant." If
project-scope discipline is load-bearing enough to belong in every
session's working memory, it should join the always-loaded set.
Trade-off: more mandatory reading at session start vs. relying on
just-in-time relevance. Resolve as part of the same audit.

*What would unblock an answer.* A deliberate pass that diffs root
`CLAUDE.md` + global/project memory against what a freshly-seeded
project receives, classifying each behavior as (a) correctly
source-only / machine-local, (b) should-be-promoted to a shipped
rule, or (c) should-be-promoted into a skill. The promotion
candidates then become their own stories.


