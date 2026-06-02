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

**MUST DO before responding to the first user message of any
session.** Read these two files end-to-end — their contents are
*not* loaded into context by default. Skipping is the #1 cause of
rule drift (KISS violations, commit-message bloat, no
simpler-alternative on additions).

- [`rules/coding-session-rules.md`](rules/coding-session-rules.md)
  — the 9 standing rules, including rule-7 commit handoff format
  and rule-4 simpler-alternative self-check.
- [`rules/design-philosophy-rules.md`](rules/design-philosophy-rules.md)
  — KISS, progressive disclosure.
  *iPhone, not Android. Macintosh, not PC.*

**Read when relevant to the current task:**

- [`rules/project-rules.md`](rules/project-rules.md) — MVP scope,
  out-of-scope list, dependency justification.
- [`rules/testing-rules.md`](rules/testing-rules.md) — test
  discipline. This project has no traditional test suite; the
  "testing" sections describe re-reading command files, mental
  dry-run, and the live-test walkthrough.
- [`rules/environment-rules.md`](rules/environment-rules.md) —
  cross-platform conventions, shells, scratch files. The
  project-specific environment is inside the
  `<!-- ONBOARD-FILL: environment -->` block.
- [`rules/multi-agent-rules.md`](rules/multi-agent-rules.md) —
  subagent usage (explore-plus-plan mode).

If Jamie says "rule 4" or "this is a rule 1 issue" mid-session,
that's a drift signal pointing at
[`rules/coding-session-rules.md`](rules/coding-session-rules.md).
Acknowledge, correct course, move on.

## Project-specific context

The "what is this template" reference is [`README.md`](README.md).
Architectural shape (source-of-truth root + `cc-template/`
distributable, universal-content duplication, source-only docs at
root) is in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md). The
testable contract is [`docs/REQUIREMENTS.md`](docs/REQUIREMENTS.md).
The Load-bearing invariants section below complements REQUIREMENTS
with "don't break this" warnings keyed to the conventions that
stage detection and recurring lifecycles depend on.

## Load-bearing invariants — do not break

Changing one of these requires auditing the whole chain.

### Configuration ritual (status comments)

- **Status comment names, exact spelling**: `ONBOARD-STATUS`,
  `BOOTSTRAP-STATUS`, `DEPLOYMENT-PLAN-STATUS`. Base values:
  `UNCONFIGURED`, `COMPLETE <YYYY-MM-DD>`. Status-specific extras:
  `BOOTSTRAP-STATUS` also takes `INSTRUCTIONS-WRITTEN <YYYY-MM-DD>`
  (README and rules written, tools not yet PATH-validated);
  `DEPLOYMENT-PLAN-STATUS` also takes `DEFERRED <YYYY-MM-DD>`. Each
  command owns exactly one comment.
- **Recurring commands do NOT own status comments in `CLAUDE.md`.**
  Per-file frontmatter status is the source of truth instead:
  `AWAITING-DECISIONS` / `LANDED` on
  `docs/design/design-review-checkpoint-NNN.md`, and
  `AWAITING-DISPOSITIONS` / `LANDED` on
  `docs/test-plans/phase-NNN-exit.md`. Don't add
  `DESIGN-REVIEW-STATUS` or `EXIT-TEST-PLAN-STATUS` banners —
  recurring artifacts conflate badly with the three-status
  configuration ritual.
- **`/bootstrap` has two modes** keyed off `BOOTSTRAP-STATUS`:
  write mode (`UNCONFIGURED` → writes instructions → flips to
  `INSTRUCTIONS-WRITTEN`); verify mode (`INSTRUCTIONS-WRITTEN` →
  runs the README's verify commands → flips to `COMPLETE`). Don't
  collapse them.

### `/design-review` lifecycle

- **Two stages, iterative across rounds** (mirrors
  `/exit-test-plan`'s addendum lifecycle). Auto-detected from
  `docs/design/design-review-checkpoint-*.md` state. Stage 1 has
  two branches: **initial** (no checkpoint or newest is `LANDED`)
  writes a new checkpoint with `AWAITING-DECISIONS`; **addendum**
  (newest is `AWAITING-DECISIONS`, latest round all marked,
  Disposition log already has rows for that round) appends
  `## Addendum N — date` with re-opened findings (suffix `-AN`)
  plus any new findings. Stage 2 (latest round marked, Disposition
  log does NOT yet have rows for that round) walks each marked
  finding, appends rows to the Disposition log, then asks Jamie:
  land or open another round. Frontmatter stays
  `AWAITING-DECISIONS` for the whole loop; Jamie never manually
  flips it. No limit on addendum count.
- **Stage 1 is findings-only (rule 4).** Both Stage 1 branches
  edit only the checkpoint file (initial also writes
  `docs/design/REVIEWS.md`); neither edits REQUIREMENTS /
  ARCHITECTURE / PROJECT_PLAN / CLAUDE_CODE_PROMPTS. Stage 2 is
  the doc-editing session, but only on the "land" path — "open
  another round" edits only the checkpoint file (Disposition log
  rows). Stage 2 land-path edit scope: REQUIREMENTS / ARCHITECTURE
  / PROJECT_PLAN / CLAUDE_CODE_PROMPTS + checkpoint file +
  REVIEWS.md. Anything else (rules, design-intake docs, CLAUDE.md,
  README) surfaces as TODO, never an edit.
- **One finding = one decision.** Each finding has exactly one
  `AUDIT NOTE — JAH:` block recording exactly one decision. If a
  Recommendation has N independently-decidable items, split into
  N findings (suffix IDs `B1a`/`B1b`/`B1c` when they share a
  narrative root, distinct top-level IDs otherwise).
- **First-level decisions primary.** When a finding presents
  Options A/B/C, analyze first-level options deeply; do NOT fully
  spec downstream sub-options that depend on the first-level pick
  (surface downstream implications in 1-2 sentences each). If a
  downstream decision becomes load-bearing later, raise it as a
  separate finding in the next addendum.
- **Disposition log is the round-walked marker.** Stage detection
  uses the `## Disposition log` table's `Round` column to
  distinguish "Stage 2 hasn't walked latest round" from "Stage 2
  walked; Jamie chose open-another-round." Round values are
  `Original` (Round 1) or `Addendum N`. Rows are append-only;
  later-round dispositions on the same root finding ID supersede
  earlier rows for landing application (earlier rows remain
  historical).
- **AUDIT NOTE placeholder is load-bearing.** Stage detection
  parses the exact string
  `> _[UNMARKED — replace this line with your decision per the legend above]_`
  to distinguish marked from unmarked findings. The placeholder
  applies to every AUDIT NOTE in every round; stage detection
  only checks the latest round. Changing this text breaks stage
  detection — audit `.claude/commands/design-review.md` Steps 0,
  S1.5, S1.A together.
- **Five-shape disposition legend.** A marked AUDIT NOTE is one of
  five shapes: `Accepted`, `Accepted with caveats`, `Defer
  Approved`, `DECISION: <choice>`, or `REJECTED: <reframing
  prose>`. REJECTED means no listed recommendation was
  satisfactory; Stage 2 records it verbatim in the Disposition log
  and treats it as an automatic open-another-round trigger, with
  the rejection prose as the next addendum's reframe input.
  Changing the legend means auditing `design-review.md` Steps
  S1.5, S2.1, S2.4, S2.5, S1.A together (both the root and
  `cc-template/` copies). Step 0 is unaffected — REJECTED is a
  non-placeholder marking like the other four.
- **`/design-review` commit handoffs only on artifact boundaries.**
  Stage 1 *initial* (new checkpoint) and Stage 2 *landing*
  (`LANDED`) surface a rule-7 handoff. Stage 1 *addendum* and
  Stage 2 *open another round* are mid-iteration — no handoff;
  wind-down stages pending changes at session close.

### `/exit-test-plan` lifecycle

- **Two stages, iterative** — auto-detected from
  `docs/test-plans/phase-*-exit.md` state for the target phase.
  Stage 1 has initial and addendum branches; Stage 2 dispositions
  the newest run log. The document can grow with multiple §6.N
  polish addendums before landing. `LANDED` requires every run
  log (§4 plus every §6.N) to be PASS / SKIP with no outstanding
  Fix-now items unverified. Frontmatter stays
  `AWAITING-DISPOSITIONS` for the whole loop; Jamie never
  manually flips it.
- **`[PENDING]` placeholder is load-bearing.** Stage detection
  parses the literal `[PENDING]` at the end of each TC row.
  Applies to every run log (§4 and every §6.N). Changing this
  text breaks stage detection — audit
  `.claude/commands/exit-test-plan.md` Steps 0, S1.5, S1.A
  together.
- **§5 grows append-only across rounds.** §5.1's table includes a
  `Round` column (`Original` for §4 findings, `Addendum N` for
  §6.N findings). Stage 2 on a later round appends to §5.1 and
  §5.2; it does not modify earlier content. §5.3 is written once
  on the first Stage 2 and never modified.
- **Stage 1 is plan-only (rule 4 + rule 8).** Either branch
  writes only the test plan file — no other doc edits, no
  inventing project affordances (new CLI commands, seeders,
  test-mode routes) to make a step easier. Stage 2 edit scope:
  test plan file + `docs/design-decisions.md` +
  `docs/open-questions.md` + `TODO.txt`. Spec gaps queue for
  `/design-review` rather than promoting directly to REQUIREMENTS
  / ARCHITECTURE.
- **`/exit-test-plan` commit handoffs only on artifact boundaries.**
  Stage 1 initial (new plan) and Stage 2 landing (`LANDED`)
  surface a rule-7 handoff. Stage 1 addendum and Stage 2 "open
  another round" are mid-iteration — no handoff; wind-down
  stages pending changes at session close.

### Cross-cutting

- **Filename conventions, zero-padded 3-digit N**:
  `docs/design/design-review-checkpoint-NNN.md` and
  `docs/test-plans/phase-NNN-exit.md` (e.g. `001`, `010`, `100`).
  Pad width fixed at 3 so files sort lexicographically through
  999. `/design-review`, `/exit-test-plan`, `/wind-down`, and any
  future scanner depend on this glob shape.
- **README section names, exact spelling**: `Developer setup`
  (stub by `/onboard`, replaced by `/bootstrap`) and `Deployment`
  (stub by `/onboard`, replaced by `/deployment-plan`). Don't
  rename without updating all three commands.
- **Marker blocks in rules files**:
  `<!-- ONBOARD-FILL: environment -->` in
  `rules/environment-rules.md` (`/bootstrap` fills) and
  `<!-- ONBOARD-FILL: project-scope -->` in
  `rules/project-rules.md` (`/onboard` fills).
- **File ownership doesn't overlap.** `/design-review` owns
  `docs/design/design-review-checkpoint-*.md` and
  `docs/design/REVIEWS.md` exclusively. `/exit-test-plan` owns
  `docs/test-plans/phase-*-exit.md` exclusively. Other commands
  surface and warn but never edit. Each command's "What this
  command does NOT do" section lists what others own — when
  adding a responsibility, audit the others.

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
