# Requirements

Functional and non-functional requirements for the
`claude-code-sdlc-template` project. Lifted from
[`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md)
during `/onboard`. Each item is a single testable statement.

## Functional requirements

**FR-1: Ship six slash commands.** The distributable
(`cc-template/.claude/commands/`) ships six commands as markdown
files: `/onboard`, `/bootstrap`, `/deployment-plan`,
`/design-review`, `/exit-test-plan`, `/wind-down`. The same six
commands live at the source root (`.claude/commands/`) for the
source-of-truth project's own self-consumption.

**FR-2: Distributable is a copyable subdirectory.** Consumers copy
the `cc-template/` subdirectory of this repo into a new project
directory and rename the destination to their project name. There
is no nested `cc-template/cc-template/` after the copy — the
consumer's new project directory IS the rename of the copied
`cc-template/`.

**FR-3: `/onboard` produces the planning docs.** Given one or more
design docs in `docs/design/`, `/onboard` writes
`docs/REQUIREMENTS.md`, `docs/ARCHITECTURE.md`,
`docs/PROJECT_PLAN.md`, and `docs/CLAUDE_CODE_PROMPTS.md`. It also
fills the `<!-- ONBOARD-FILL: project-scope -->` block in
`rules/project-rules.md`, rewrites `rules/multi-agent-rules.md`
to the chosen mode, writes a README stub, and flips
`<!-- ONBOARD-STATUS: UNCONFIGURED -->` to
`<!-- ONBOARD-STATUS: COMPLETE <date> -->`.

**FR-4: `/bootstrap` has two modes keyed off `BOOTSTRAP-STATUS`.**
Write mode (from `UNCONFIGURED`) writes the README "Developer
setup" section and the `<!-- ONBOARD-FILL: environment -->` block
in `rules/environment-rules.md`, then flips status to
`INSTRUCTIONS-WRITTEN <date>`. Verify mode (from
`INSTRUCTIONS-WRITTEN`) runs the README's verify commands and
flips status to `COMPLETE <date>`.

**FR-5: `/deployment-plan` is deferrable.** Optional command;
produces `docs/DEPLOYMENT.md` and the README "Deployment" section.
Status comment supports `DEFERRED <date>` in addition to the
standard values.

**FR-6: `/design-review` is two-stage and iterative.** Stage 1 has
two branches: the initial branch writes a new
`docs/design/design-review-checkpoint-NNN.md` with severity-tiered
findings (Blockers / Recommendations / Notes); the addendum
branch appends `## Addendum N — <date>` sections with re-opened
findings (suffix `-AN`) plus any new findings. Stage 2 walks the
latest round's marked findings, appends rows to the Disposition
log, then asks Jamie to land the doc or open another addendum
round.

**FR-7: `/exit-test-plan` is two-stage and iterative.** Stage 1
has two branches: the initial branch writes
`docs/test-plans/phase-NNN-exit.md` with test cases (Steps /
Expected / Fail signals), §4 run log with `[PENDING]`
placeholders, and an empty §5; the addendum branch appends `§6.N
polish addendum` sections. Stage 2 reads the newest filled run
log + trailing notes, walks findings, appends rows to §5, and
asks Jamie to land the doc or open another polish round.

**FR-8: `/wind-down` rewrites `TODO.txt` per rule 9.** Completed
items are removed (not struck through); first entry is the
next-session pick-up; other tracking docs (`design-decisions.md`,
`open-questions.md`, `CLAUDE_CODE_PROMPTS.md` deviation footers,
`ARCHITECTURE.md`, `REQUIREMENTS.md`) are surfaced for proposed
edits when the session warrants.

**FR-9: Three status comments in `CLAUDE.md` govern the
configuration ritual lifecycle.** `ONBOARD-STATUS`,
`BOOTSTRAP-STATUS`, `DEPLOYMENT-PLAN-STATUS` — each owned by
exactly one command. Base values: `UNCONFIGURED`,
`COMPLETE <YYYY-MM-DD>`. Status-specific values:
`INSTRUCTIONS-WRITTEN <date>` (BOOTSTRAP only),
`DEFERRED <date>` (DEPLOYMENT-PLAN only).

**FR-10: Two `ONBOARD-FILL` marker blocks in rules files.**
`<!-- ONBOARD-FILL: project-scope -->` in `rules/project-rules.md`
(filled by `/onboard`); `<!-- ONBOARD-FILL: environment -->` in
`rules/environment-rules.md` (filled by `/bootstrap`). Markers
demarcate downstream-owned content from template-owned content for
`/refresh-from-repository --merge` (Phase 2).

**FR-11: Universal content is identical between source and dist.**
The 9 coding-session rules, design philosophy rules, the six
command files, and the placeholder rules files
(`multi-agent-rules.md`, `project-rules.md`,
`environment-rules.md`, `testing-rules.md`) ship identical at root
and inside `cc-template/`. Edits land in `cc-template/` first, then
copy up to root if it's a command or rules file this project
itself uses.

**FR-12: Source-only content lives only at root.** The
source-of-truth project's REQUIREMENTS / ARCHITECTURE /
PROJECT_PLAN / CLAUDE_CODE_PROMPTS, `docs/design/` intake,
`docs/design-decisions.md`, `docs/open-questions.md`, this
project's README, and root `CLAUDE.md` never copy into the dist.

**FR-13: `/refresh-from-repository` is a Phase 2 deliverable.** A
two-stage self-modifying command shipped in the dist. Stage 1
selectively pulls the latest `.claude/commands/*.md` and
`rules/*.md` from the public upstream; stage 2 (`--merge`) runs
the freshly-updated logic and surgically merges deltas in
downstream `CLAUDE.md` and `rules/*.md` while preserving
`<!-- ONBOARD-FILL: ... -->` blocks. Refuses when invoked against
the source repo (the source is upstream; pulling from itself is a
footgun).

**FR-14: `/write-user-documentation` is a Phase 2 deliverable.** A
command that authors end-user documentation. Shipped in the dist;
used in the source-of-truth project to document itself; used in
downstream projects by consumers to create their product
documentation. Spec deferred to Phase 2.2.

**FR-15: `/exit-test-plan` Phase 1.1 revision adds a BLOCKED
disposition.** Test plan items not runnable due to a blocker
discovered during testing get marked `BLOCKED` in the run log.
The blocker may require a cleanup coding pass or a
`/design-review` checkpoint with additional phases before testing
can continue. Blocked items remain in the record as blocked at the
time; the blocked component is re-tested in the next plan
addendum.

**FR-16: Regression-test automation is a Phase 3 deliverable.** A
scripted check that diffs the current `cc-template/` dist against
a tagged baseline and flags invariant-breaking changes (status
comment renames, `ONBOARD-FILL` marker drift, zero-pad-width
changes in `checkpoint-NNN` / `phase-NNN-exit` filenames).
Specifics deferred until a real regression motivates the work.

## Non-functional requirements

**NFR-1: Cross-platform.** Windows 11 is primary; Linux and macOS
are first-class supported. Anything that works on Linux but breaks
on Windows is a bug, not a platform quirk to document around. Per
[`rules/environment-rules.md`](../rules/environment-rules.md).

**NFR-2: Zero-padded 3-digit filenames for recurring artifacts.**
`docs/design/design-review-checkpoint-NNN.md` and
`docs/test-plans/phase-NNN-exit.md` — pad width fixed at 3 so
files sort lexicographically as numeric order through 999.
`/design-review`, `/exit-test-plan`, and `/wind-down` depend on
this glob shape.

**NFR-3: Per-file frontmatter status on recurring artifacts.**
`AWAITING-DECISIONS` / `LANDED` on
`docs/design/design-review-checkpoint-*.md`;
`AWAITING-DISPOSITIONS` / `LANDED` on
`docs/test-plans/phase-*-exit.md`. Frontmatter status is the
source of truth for the recurring-artifact lifecycle (the three
status comments in `CLAUDE.md` govern only the configuration
ritual).

**NFR-4: Stage-detection placeholders are load-bearing.** Stage
detection parses for exact strings:
`> _[UNMARKED — replace this line with your decision per the legend above]_`
(in `/design-review` AUDIT NOTE blocks) and `[PENDING]` (in
`/exit-test-plan` §4 / §6.N run log rows). Changing the
placeholder text breaks stage detection — audit `Step 0`, `Step
S1.5`, and `Step S1.A` of the relevant command file together.

**NFR-5: Failures refuse explicitly.** Commands refuse with a
pointer to the right command when prerequisites aren't met (e.g.
`/bootstrap` refuses if `ONBOARD-STATUS` is still `UNCONFIGURED`).
No silent fallbacks.

**NFR-6: No language runtime.** The deliverable is markdown files
only. No GUI, no web service, no API, no language interpreter
required to use the template. Validation is the live-test
walkthrough described in root `CLAUDE.md`.

**NFR-7: Dual license.** The distributable (`cc-template/`) ships
under MIT — permissive, allows consumer commercial use including
non-fork projects. Source-only docs at repo root (REQUIREMENTS,
ARCHITECTURE, PROJECT_PLAN, CLAUDE_CODE_PROMPTS, design intake,
design-decisions, open-questions, this README, root CLAUDE.md)
ship under CC BY-NC-ND 4.0 — those project-management docs are
not meant to be redistributed.

**NFR-8: File ownership doesn't overlap.** Each command's "What
this command does NOT do" section lists what the others own.
`/design-review` owns `docs/design/design-review-checkpoint-*.md`
and `docs/design/REVIEWS.md` exclusively. `/exit-test-plan` owns
`docs/test-plans/phase-*-exit.md` exclusively. The other commands
surface and warn but never edit those files.

**NFR-9: Source/dist duplication is deliberate and audited.**
Universal content (rules, design philosophy, command files) is
kept identical between root and `cc-template/`. The duplication is
intentional (root project consumes its own template) but creates
drift risk — Phase 3 regression-test automation exists to catch
invariant-breaking drift. Pre-Phase-3 discipline: edit in
`cc-template/` first, copy up.
