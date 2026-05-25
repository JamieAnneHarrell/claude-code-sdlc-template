# CLAUDE.md

<!-- ONBOARD-STATUS: COMPLETE 2026-05-24 -->
<!-- BOOTSTRAP-STATUS: COMPLETE 2026-05-24 -->
<!-- DEPLOYMENT-PLAN-STATUS: UNCONFIGURED -->

> ✅ **Onboarded and bootstrapped.** Ready for the next phase
> prompt in `docs/CLAUDE_CODE_PROMPTS.md` (Phase 1.1 — add
> `BLOCKED` disposition to `/exit-test-plan`). `/deployment-plan`
> is deferrable — run it when distribution mechanism (git clone
> vs zip vs other) needs to be pinned.
>
> **Project:** `claude-code-sdlc-template` — a self-modifying
> project seed for new software projects driven by Claude Code.
> The distributable subdirectory `cc-template/` ships six slash
> commands (`/onboard`, `/bootstrap`, `/deployment-plan`,
> `/design-review`, `/exit-test-plan`, `/wind-down`) plus curated
> collaboration rules. Consumers copy `cc-template/` into a new
> project, drop a design doc into `docs/design/`, and run the
> configuration ritual. This repository is itself a downstream
> consumer of its own template, so improvements are exercised on
> the source before they ship.
>
> **Language / framework:** No language runtime — markdown only.
> **Multi-agent mode:** explore-plus-plan
> **Shell:** PowerShell on Windows 11 (Jamie's maintainer setup); project itself is shell-agnostic
> **Run:** N/A — markdown only &nbsp;·&nbsp; **Test:** N/A — validated by use of the template on real projects
> **Developer setup:** see `README.md`

## Reading order at session start

1. **This file** — quick orientation and reading list.
2. **`TODO.txt`** — gitignored session handoff. First entry is
   what we pick up next; the rest is the upcoming queue.
3. **`docs/PROJECT_PLAN.md`** — active phase queue.
4. **`docs/CLAUDE_CODE_PROMPTS.md`** — authoritative prompt for
   the current phase.
5. **`docs/design/`** — design intake plus any `/design-review`
   checkpoints in flight. A checkpoint with
   `AWAITING-DECISIONS` frontmatter means Stage 1 ran but Stage
   2 hasn't walked dispositions yet (or there's an addendum
   round mid-iteration).
6. **`docs/test-plans/`** — if the current phase has a test plan
   in flight, it'll be `phase-NNN-exit.md` here.
   `AWAITING-DISPOSITIONS` means the phase is somewhere in the
   iterative test-and-polish loop (original run, dispositions,
   polish coding session, optional §6.N addendums); `LANDED`
   means every run log is PASS / SKIP, every Fix-now is
   verified, and the phase exit is closed.

## Collaboration rules

**Read these every session, before any work:**

- [`rules/coding-session-rules.md`](rules/coding-session-rules.md)
  — the 9 standing rules (root-cause, trust diagnosis, rejections
  permanent, no unsolicited design, decouple data from display,
  reference rule numbers, Jamie runs commits, Jamie runs tests,
  session wind-down rewrites TODO.txt). Includes the rule-7
  commit handoff format.
- [`rules/design-philosophy-rules.md`](rules/design-philosophy-rules.md)
  — KISS, progressive disclosure, simple defaults.
  *iPhone, not Android. Macintosh, not PC.*

**Read these when relevant to the current task:**

- [`rules/project-rules.md`](rules/project-rules.md) —
  project-specific scope discipline. MVP scope statements,
  out-of-scope list, and the dependency-justification rule.
- [`rules/testing-rules.md`](rules/testing-rules.md) — test
  discipline. This project has no traditional test suite; the
  "testing" sections describe re-reading the command files, the
  mental dry-run, and the live-test walkthrough.
- [`rules/environment-rules.md`](rules/environment-rules.md) —
  cross-platform conventions, shells, where Claude scratch files
  go. Project-specific environment lives inside the
  `<!-- ONBOARD-FILL: environment -->` block and is filled by
  `/bootstrap`.
- [`rules/multi-agent-rules.md`](rules/multi-agent-rules.md) —
  how this project uses subagents (explore-plus-plan mode).

If Jamie says "rule 4" or "this is a rule 1 issue" mid-session,
that's a drift signal pointing at
[`rules/coding-session-rules.md`](rules/coding-session-rules.md).
Acknowledge, correct course, move on.

## Project-specific context

The full reference for "what is this template" lives in
[`README.md`](README.md). The architectural shape (source-of-truth
root + `cc-template/` distributable, universal-content
duplication, source-only docs at root) is in
[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md). The testable
contract — functional requirements per command, non-functional
invariants — is in [`docs/REQUIREMENTS.md`](docs/REQUIREMENTS.md).

This file ALSO documents the load-bearing invariants below, in
template-improvement language ("if you change this, audit the
whole chain first") that complements REQUIREMENTS.md's
testable-contract framing. Both are deliberate: REQUIREMENTS.md
is the contract; the section below is the don't-break-this
warning Jamie sees at session start.

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
- **`/design-review` commit handoffs only on artifact boundaries.**
  Stage 1 *initial* branch (new checkpoint created) and Stage 2
  *landing* (doc flipped to `LANDED`) surface a rule-7 commit
  handoff. Stage 1 *addendum* branch and Stage 2 *open another
  round* are mid-iteration and do NOT surface commit handoffs —
  wind-down stages any pending changes at session close. Mirrors
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
- **`/exit-test-plan` commit handoffs only on artifact boundaries.**
  Stage 1 initial branch (new plan created) and Stage 2 landing
  (doc flipped to `LANDED`) surface a rule-7 commit handoff.
  Stage 1 addendum branch and Stage 2 "open another round" are
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

## Migrating projects from older template versions

If a project was onboarded under a previous template version (e.g.
the old single-`DS-TEMPLATE-STATUS` system), the migration steps
are documented in
`C:\Users\jamie\.claude\plans\good-morning-claude-the-zippy-wind.md`
section C. Reuse that checklist; don't re-derive.
