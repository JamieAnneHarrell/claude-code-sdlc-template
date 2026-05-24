# CLAUDE.md

> 🛠️ **You are working ON cc-template, not in a copied project.**
> This file is the authoritative session-start guide for
> template-improvement work. Read it cold.

## What this template is

`cc-template` is the seed Jamie copies into a new directory to start
a new project. The template ships **three configuration commands**
plus **two recurring commands**.

The configuration ritual is a sequence of three slash commands:

1. **`/onboard`** — *what are we building?* Reads design docs from
   `docs/design/`, produces REQUIREMENTS / ARCHITECTURE /
   PROJECT_PLAN / CLAUDE_CODE_PROMPTS, and writes README and rules
   stubs.
2. **`/bootstrap`** — *how do I start coding?* Reads the onboarded
   project, plans the dev environment, fills the README "Developer
   setup" section and the `<!-- ONBOARD-FILL: environment -->`
   block in `rules/environment-rules.md`. Hard prerequisite to
   Phase 0.
3. **`/deployment-plan`** — *how does this ship?* Optional and
   deferrable. Produces `docs/DEPLOYMENT.md` and the README
   "Deployment" section.

The split exists because the three questions get answered with
different amounts of information at different times. Don't collapse
them back together.

The recurring commands are:

4. **`/design-review`** — *are we still building the right thing
   the right way?* Two-stage and iterative. Stage 1 has two
   branches: the **initial branch** writes a sign-off-ready
   checkpoint document at `docs/design/design-review-checkpoint-NNN.md`
   with severity-tiered findings (Blockers / Recommendations /
   Notes), one `AUDIT NOTE — JAH:` block per finding (one finding
   = one decision), and an empty `## Disposition log` placeholder;
   the **addendum branch** (after a research / clarification
   session between rounds) appends a new `## Addendum N — date`
   section with re-opened findings (suffix `-AN`) plus any new
   findings, in the same shape. Stage 2 walks the latest round's
   marked AUDIT NOTE blocks, appends rows to the Disposition log,
   then asks Jamie whether to land the doc or open another
   addendum round; on the land path it applies the accumulated
   revisions to REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN /
   CLAUDE_CODE_PROMPTS. Mirrors `/exit-test-plan`'s addendum
   lifecycle. Runs at high-risk transitions over the life of the
   project, not as part of configuration. See
   `.claude/commands/design-review.md`.

5. **`/exit-test-plan`** — *did the phase actually meet its exit
   criteria as a human can verify them?* Two-stage and iterative.
   Stage 1 has two branches: the **initial branch** reads the
   current phase's exit criteria + prompt + implementation files
   and writes a manual walkthrough at
   `docs/test-plans/phase-NNN-exit.md` (§1 scope, §2 prep, §3 test
   cases as Steps / Expected / Fail signals, §4 run log with
   `[PENDING]` placeholders, §5 empty); the **addendum branch**
   (after a polish coding session lands fixes) appends a new §6.N
   polish addendum with first-class TCs in the same shape as §3,
   plus its own run log and comments. Stage 2 reads the newest
   filled run log + trailing notes, walks each finding, performs
   root-cause analysis where warranted, appends rows to §5, and
   asks Jamie whether to land the document or open another polish
   round. Mirrors decisions to `docs/design-decisions.md` /
   `docs/open-questions.md` / `TODO.txt`. Runs at every phase exit
   that warrants a manual check, not as part of configuration.
   See `.claude/commands/exit-test-plan.md`.

Both recurring commands are deliberate exceptions to the "don't
add a fourth command" guideline — each addresses a real drift
risk that was first felt on a live project and lifted into the
template only after the pattern proved itself.

## Load-bearing invariants — do not break

If you change one of these, audit the whole chain first.

- **Status comment names, exact spelling**: `ONBOARD-STATUS`,
  `BOOTSTRAP-STATUS`, `DEPLOYMENT-PLAN-STATUS`. Base values:
  `UNCONFIGURED`, `COMPLETE <YYYY-MM-DD>`. Two extra
  status-specific values exist: `BOOTSTRAP-STATUS` also takes
  `INSTRUCTIONS-WRITTEN <YYYY-MM-DD>` (README and rules written,
  developer has not yet validated that required tools are on
  PATH); `DEPLOYMENT-PLAN-STATUS` also takes
  `DEFERRED <YYYY-MM-DD>`. Each command owns exactly one comment.
- **Recurring commands do NOT own status comments in `CLAUDE.md`.**
  Per-file frontmatter status is the source of truth for each:
  `AWAITING-DECISIONS` / `LANDED` on
  `docs/design/design-review-checkpoint-NNN.md`, and
  `AWAITING-DISPOSITIONS` / `LANDED` on
  `docs/test-plans/phase-NNN-exit.md`. Don't add a
  `DESIGN-REVIEW-STATUS` or `EXIT-TEST-PLAN-STATUS` banner — the
  three-status banner is the configuration ritual; recurring
  artifacts conflate badly with it.
- **`/bootstrap` has two modes** keyed off the current
  `BOOTSTRAP-STATUS` value: write mode (from `UNCONFIGURED`) writes
  instructions and flips to `INSTRUCTIONS-WRITTEN`; verify mode
  (from `INSTRUCTIONS-WRITTEN`) runs the README's verify commands
  and flips to `COMPLETE`. Don't collapse them.
- **`/design-review` has two stages** auto-detected from the state
  of `docs/design/design-review-checkpoint-*.md`, and the full
  lifecycle is **iterative** across rounds (mirrors
  `/exit-test-plan`'s addendum lifecycle). Stage 1 has two
  branches: the **initial branch** (no checkpoint, or newest is
  `LANDED`) writes a new checkpoint with `AWAITING-DECISIONS`; the
  **addendum branch** (newest is `AWAITING-DECISIONS`, latest
  round all marked, Disposition log already has rows for that
  round) appends a new `## Addendum N — date` section with
  re-opened findings (suffix `-AN`) plus any new findings. Stage 2
  (newest is `AWAITING-DECISIONS`, latest round all marked,
  Disposition log does NOT yet have rows for that round) walks
  each marked finding, appends rows to the Disposition log, then
  asks Jamie whether to land the doc or open another addendum
  round. Frontmatter stays `AWAITING-DECISIONS` for the whole
  iterative loop; Jamie never manually flips it. There is no
  limit on the number of addendums.
- **Stage 1 is findings-only (rule 4).** Both Stage 1 branches
  edit only the checkpoint file (initial also writes
  `docs/design/REVIEWS.md`); neither edits REQUIREMENTS /
  ARCHITECTURE / PROJECT_PLAN / CLAUDE_CODE_PROMPTS. Stage 2 IS
  the doc-editing session, and only on the "land" path — the
  "open another round" path edits only the checkpoint file
  (Disposition log rows added). Stage 2's land-path edit scope is
  exactly REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN /
  CLAUDE_CODE_PROMPTS plus the checkpoint file itself plus
  REVIEWS.md. Anything outside that scope (rules files,
  design-intake docs, CLAUDE.md, README.md) surfaces as a TODO,
  never an edit.
- **One finding = one decision.** Composition rule in Step S1.4:
  every finding gets exactly one `AUDIT NOTE — JAH:` block, and
  that block records exactly one decision. If a finding's
  Recommendation spans N independently-decidable items, split into
  N findings (suffixed IDs `B1a`/`B1b`/`B1c` when they share a
  narrative root, distinct top-level IDs otherwise). Bundling
  multiple recommendations under one AUDIT NOTE defeats Jamie's
  ability to sign each off independently.
- **First-level decisions primary.** Composition rule in Step S1.4:
  when a finding presents Option A / B / C, analyze the
  first-level options deeply but do NOT fully spec downstream
  sub-options that depend on which first-level option is picked.
  Surface downstream implications in 1-2 sentences each. If a
  downstream decision becomes load-bearing after the first-level
  pick, surface it as a separate finding in the next addendum.
- **Disposition log is the round-walked marker.** Stage detection
  uses the `## Disposition log` table's `Round` column to
  distinguish "Stage 2 hasn't walked the latest round yet" from
  "Stage 2 walked it; Jamie picked open-another-round." Round
  values are `Original` (for Round 1 Findings) or `Addendum N`
  (for the Nth addendum). Rows append-only across rounds —
  later-round dispositions on the same root finding ID supersede
  earlier ones for landing application; earlier rows remain as
  historical record.
- **Checkpoint filename convention**:
  `docs/design/design-review-checkpoint-NNN.md` with **zero-padded
  3-digit N** (`001`, `002`, ..., `010`, ..., `100`). The pad width
  is fixed at 3 so files sort lexicographically as numeric order
  through 999. `/design-review`, `/wind-down`, and any future
  command that scans checkpoints depend on this glob shape; don't
  change it without auditing all consumers.
- **AUDIT NOTE placeholder line is load-bearing.** Stage detection
  parses for the exact string
  `> _[UNMARKED — replace this line with your decision per the legend above]_`
  to distinguish marked from unmarked findings. The placeholder
  applies to **every** AUDIT NOTE block in the document — Round 1
  Findings and every Addendum N — and they're all first-class
  finding markups of identical shape. Stage detection only checks
  blocks in the **latest round**; earlier rounds are historical
  record once their dispositions landed in the Disposition log.
  If the placeholder text changes, stage detection breaks — audit
  `.claude/commands/design-review.md` Step 0, Step S1.5, and Step
  S1.A together.
- **Commit handoffs only on artifact boundaries.** Stage 1
  *initial* branch (new checkpoint created) and Stage 2 *landing*
  (doc flipped to `LANDED`) surface a rule-7 commit handoff.
  Stage 1 *addendum* branch and Stage 2 *open another round* are
  mid-iteration and do NOT surface commit handoffs — wind-down
  stages any pending changes at session close. Mirrors
  `/exit-test-plan`'s commit-handoff discipline.
- **`/exit-test-plan` has two stages** auto-detected from the
  state of `docs/test-plans/phase-*-exit.md` for the target
  phase. Stage 1 has two branches (initial plan and addendum
  authoring); Stage 2 dispositions the newest run log. The full
  lifecycle is **iterative**: the document can grow with multiple
  §6.N polish addendums before it lands, and `LANDED` requires
  every run log (§4 plus every §6.N) to be PASS / SKIP with no
  outstanding Fix-now items left unverified. Frontmatter stays
  at `AWAITING-DISPOSITIONS` for the whole iterative loop;
  Jamie never manually flips it.
- **Test plan filename convention**:
  `docs/test-plans/phase-NNN-exit.md` with **zero-padded 3-digit N**
  (`phase-001-exit.md`, ..., `phase-010-exit.md`, ...,
  `phase-100-exit.md`). Same pad-width discipline as
  design-review-checkpoint-NNN. `/exit-test-plan`, `/wind-down`,
  and any future command that scans test plans depend on this
  glob shape.
- **`[PENDING]` placeholder in any run log is load-bearing.**
  Stage detection parses for the literal uppercase bracketed
  string `[PENDING]` at the end of each TC row. The placeholder
  applies to **every** run log in the document — §4 and every
  §6.N — and they're all first-class run logs of identical shape.
  If the placeholder text changes, stage detection breaks —
  audit `.claude/commands/exit-test-plan.md` Step 0, Step S1.5,
  and Step S1.A together.
- **§5 grows append-only across rounds.** §5.1's table includes
  a `Round` column (`Original` for §4 findings, `Addendum N` for
  §6.N findings). Stage 2 on a later round appends rows to
  §5.1 and sub-sections to §5.2; it does not modify earlier
  content. §5.3 (process note) is written once on the first
  Stage 2 and never modified.
- **Stage 1 of `/exit-test-plan` is plan-only (rule 4 + rule 8).**
  Whichever branch fires (initial or addendum), Stage 1 writes
  only the test plan file. It does not edit any other doc and
  does not invent project affordances (new CLI commands, new
  seeders, test-mode routes) to make a test step easier. Stage 2
  IS the disposition-applying session, and its edit scope is
  exactly the test plan file + `docs/design-decisions.md` +
  `docs/open-questions.md` + `TODO.txt`. Spec gaps queue for
  `/design-review` rather than promoting directly to
  REQUIREMENTS / ARCHITECTURE.
- **Commit handoffs only on artifact boundaries.** Stage 1
  initial branch (new plan created) and Stage 2 landing (doc
  flipped to `LANDED`) surface a rule-7 commit handoff. Stage 1
  addendum branch and Stage 2 "open another round" are
  mid-iteration and do NOT surface commit handoffs — wind-down
  stages any pending changes at session close.
- **README section names, exact spelling**: `Developer setup` and
  `Deployment`. `/onboard` writes both as stubs; `/bootstrap`
  replaces the first; `/deployment-plan` replaces the second.
  Don't rename without updating all three commands.
- **Marker blocks in rules files**:
  `<!-- ONBOARD-FILL: environment -->` in
  `rules/environment-rules.md` (filled by `/bootstrap`) and
  `<!-- ONBOARD-FILL: project-scope -->` in
  `rules/project-rules.md` (filled by `/onboard`).
- **File ownership doesn't overlap.** Each command's "What this
  command does NOT do" section lists what the others own. If you
  add a responsibility to one command, audit the others.
  `/design-review` owns `docs/design/design-review-checkpoint-*.md`
  and `docs/design/REVIEWS.md` exclusively. `/exit-test-plan` owns
  `docs/test-plans/phase-*-exit.md` exclusively. The other
  commands surface and warn but never edit those files.

## Design principles for template changes

- **KISS / progressive disclosure** (see
  [`rules/design-philosophy-rules.md`](rules/design-philosophy-rules.md)).
  *iPhone, not Android. Macintosh, not PC.*
- **Don't add another command** without strong justification.
  The template already ships three configuration commands and
  two recurring ones; each recurring command earned its place by
  first proving itself on a live project. New commands should
  clear the same bar.
- **Tailored questions per stack** — don't ask universal questions
  when the design doc already pins the stack.
- **Failures refuse explicitly** with a pointer to the right
  command. No silent fallbacks.

## Standard rules still apply
Read all of the rules/*.md files first.

When working on the template, also follow the standard rules in
`rules/`. The most important during template-improvement work:

- [`rules/coding-session-rules.md`](rules/coding-session-rules.md) —
  the 9 rules, including rule-7 commit handoff format.
- [`rules/design-philosophy-rules.md`](rules/design-philosophy-rules.md) —
  KISS and progressive disclosure.

## Testing template changes

In increasing order of confidence:

1. **Re-read the six command files end-to-end** after a change.
   Check the chain is unbroken: status names match, file ownership
   doesn't overlap, prereqs reference what the previous command
   actually produces, the design-review checkpoint convention
   (filename, frontmatter status, AUDIT NOTE placeholder text,
   Disposition log Round column shape) is consistent across
   `/onboard`, `/design-review`, and `/wind-down`, and the
   test-plan convention (filename, frontmatter status, `[PENDING]`
   placeholder) is consistent across `/exit-test-plan` and
   `/wind-down`.
2. **Mental dry-run** against a hypothetical project (Python CLI,
   PHP/LAMP, Node/web). Verify the prompts and outputs are coherent.
3. **Live test**: copy `cc-template/` to a sandbox dir, drop a
   known-good design doc into `docs/design/`, then run the full
   chain in order: `/onboard` → `/design-review` (stage 1
   initial) → mark up the checkpoint inline (some markings
   asking for further investigation) → `/design-review` (stage 2,
   recommends "open Addendum 1") → research session → re-run
   `/design-review` (stage 1 addendum branch) → mark up
   Addendum 1 (all closing cleanly this round) → `/design-review`
   (stage 2 lands the doc) → `/bootstrap` → Phase 0 work →
   `/exit-test-plan` (stage 1 initial) → run the plan and fill
   §4 → `/exit-test-plan` (stage 2 dispositions) → polish coding
   session lands fixes → `/exit-test-plan` (stage 1 addendum
   branch) → run §6.1 and fill its run log → `/exit-test-plan`
   (stage 2 lands) → between-phase `/design-review` pair →
   `/deployment-plan` (full path and deferred path) →
   `/wind-down`. Inspect outputs against each command file's
   spec. Pay special attention to: zero-padded NNN sorting past
   `checkpoint-009.md` and past `phase-009-exit.md`; the
   wind-down warning when a checkpoint is `AWAITING-DECISIONS`
   (any of its four shapes — unmarked latest round, walked
   pending land-or-addendum, between-rounds research session,
   addendum mid-mark) or a test plan is mid-polish-loop, and
   TODO.txt's first entry doesn't reflect it; design-review
   Stage 2's refusal when any latest-round finding is unmarked;
   design-review Stage 1 addendum branch's refusal when no
   marking actually asks for further investigation;
   exit-test-plan Stage 2's refusal when any run log row (§4 or
   §6.N) is still `[PENDING]`; the exit-test-plan addendum
   branch's refusal when no Fix-now items are ready for
   verification.

## Migrating projects from older template versions

If a project was onboarded under a previous template version (e.g.
the old single-`DS-TEMPLATE-STATUS` system), the migration steps
are documented in
`C:\Users\jamie\.claude\plans\good-morning-claude-the-zippy-wind.md`
section C. Reuse that checklist; don't re-derive.
