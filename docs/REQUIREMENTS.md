# Requirements

Functional and non-functional requirements for the
`claude-code-sdlc-template` project. Lifted from
[`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md)
during `/onboard`. Each item is a single testable statement.

## Functional requirements

**FR-1: Ship the SDLC slash commands.** The distributable
(`cc-template/.claude/commands/`) ships the SDLC commands as markdown
files: `/onboard`, `/bootstrap`, `/deployment-plan`, `/design-review`,
`/exit-test-plan`, `/product-visioning`, `/wind-down`,
`/refresh-from-repository` (FR-13), and `/write-documentation` (FR-14). The
same commands live at the source root (`.claude/commands/`) for the
source-of-truth project's own self-consumption.

**FR-2: Distributable is a copyable subdirectory.** Consumers copy
the `cc-template/` subdirectory of this repo into a new project
directory and rename the destination to their project name. There
is no nested `cc-template/cc-template/` after the copy — the
consumer's new project directory IS the rename of the copied
`cc-template/`.

**FR-3: `/onboard` decomposes the next PRD into the planning docs.** On the
**first PRD** (the design intake in `docs/design/` — a dropped design doc or
a `/product-visioning` `PRD-<slug>-001`), `/onboard` writes
`docs/REQUIREMENTS.md`, `docs/ARCHITECTURE.md`, `docs/PRODUCT_VISION.md`,
`docs/PROJECT_PLAN.md`, `docs/CLAUDE_CODE_PROMPTS.md`, and adopts the
design intake as `docs/design/PRD-<slug>-001.md` (`ACTIVE`); fills the
`<!-- ONBOARD-FILL: project-scope -->` block in `rules/project-rules.md`,
rewrites `rules/multi-agent-rules.md` to the chosen mode, writes a README
stub, and flips `<!-- ONBOARD-STATUS: ... -->` to `COMPLETE <date>`. On a
**later movement's PRD** it applies the PRD's vision revisions to
`PRODUCT_VISION.md`, archives the prior plan to `docs/project-plans/`, and
authors the new movement's PROJECT_PLAN + prompts (no re-setup).

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
`/refresh-from-repository` (Phase 2).

**FR-11: Universal content is identical between source and dist.**
The 10 coding-session rules, design philosophy rules, the
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
single-stage command shipped in the dist that pulls latest
`.claude/commands/*.md` from upstream and merges latest
`rules/*.md` + `CLAUDE.md` against the downstream's current state
using **stateless marker-state reconciliation** (adopted by
[checkpoint 004](design/design-review-checkpoint-004.md) B1,
replacing checkpoint 002's Option D hash/baseline/state-file
mechanism — now an abandoned approach). Template-managed regions
are wrapped in `CC-TEMPLATE-BLOCK` markers that carry per-block
state: `template-owned` (normal, tracks upstream), `forked`
(consumer-owned, set by asking once), and `removed` (a tombstone
where a block was deleted or moved out). The marker memory lives
in the rules / CLAUDE.md files themselves — there is **no sidecar
state file, no per-block content hash, and no baseline reference.**

Reconciliation is a **two-way compare + ask-once-and-record**,
matched by block id: a block matching upstream is skipped; a
divergent unmarked block triggers a one-time question (keep mine →
`forked` / take upstream / hand-merge); a `forked` block is left
alone (optionally noting upstream now differs); a `removed`
tombstone is respected and never re-added; a block absent with no
tombstone triggers a one-time question (never had it → add /
removed it → write tombstone); a new upstream block is added. The
executing Claude session performs the compare and the surgical
merge (per `design-decisions.md` "Template marks its own
content"). Markers use the pinned `CC-TEMPLATE-BLOCK` syntax with a
stable kebab-case `<id>` and carry per-block state per NFR-4
(`template-owned` / `forked` / `removed`); the exact encoding of
that state in the marker is settled by the Phase 2.1.A build.

**Drift** is handled by the command file's own
`Refresh-logic-version` stamp as the **sole lever**: when
upstream's stamp is higher, refresh pulls commands skills-only and
asks the consumer to re-invoke. **Source-mode and public-mode
converge** — the only difference is reading upstream-current from
the local `cc-template/` subdir (when present at cwd) versus the
public GitHub URL; there is no baseline to fetch, and public mode
no longer needs git *history*. First-seed divergence is not a
special case: the first refresh is a one-time guided
yours/take/merge walk over diverged blocks, then quiet. On first
source-mode invocation in a repo, refresh content-inspects the
`cc-template/` subdir to confirm it's this template and caches the
determination in CLAUDE.md. Two progressive-disclosure flags ship
in v1: `--refresh-skills-only` (manual override for the
drift-staging path) and `--no-claudemd` (skip CLAUDE.md from the
merge). Full design contract in checkpoint 004 (B1 Option A) plus
the surviving checkpoint 002 marker-syntax / coarse-wrapping /
CLAUDE.md-partition decisions.

**FR-14: `/write-documentation` authors audience-facing documentation.**
A two-stage command shipped in the dist — used in the source-of-truth
project to document itself and by downstream consumers to document their
products. It is a *reasoning* command. Stage 1 surveys the project (internal
docs **and** the real implementation), infers the product type and audiences,
and writes a numbered sign-off manifest
`docs/published/documentation-plan-NNN.md` (status
`AWAITING-APPROVAL` → `ACTIVE` → `SUPERSEDED`, newest governs). Stage 2 authors
or reconciles the markdown sources under `docs/published/` (the user-facing
README sections, quick-start, guides, reference, troubleshooting) applying an
embedded documentation-craft doctrine (Diátaxis modes; per-audience voice
profiles; CLI-reference, readability, and anti-pattern rules). Markdown is the
deliverable; PDF rendering is an opt-in, environment-side build via a
per-project-recorded toolchain (NFR-6 unaffected). Currency is **derived and
movement-aware**: the manifest stamps `documented-through { movement, phase }`,
and a re-run is STALE when the active movement differs or a later phase shipped
in the same movement. The manifest's release-readiness ledger (stale docs /
unfilled visuals / doc-vs-code **conformance gaps**) is a signal a release
process may gate on; the command surfaces conformance gaps but never edits
product code (rule 8). Renamed from the working title
`/write-user-documentation` — it documents every audience, not only end users.
Full spec in the shipped command file.

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

**FR-17: `/product-visioning` is an interactive session that produces the
next PRD.** First PRD or n-th, its single output is a stand-alone `DRAFT`
`docs/design/PRD-<slug>-NNN.md` — the movement's starting point, scope (in /
out / deferred), proposed `PRODUCT_VISION.md` revisions, and decisions.
Routing happens in dialogue (no inline mark-up step). It never edits
`PRODUCT_VISION.md`, `PROJECT_PLAN.md`, `CLAUDE_CODE_PROMPTS.md`, or other
docs — `/onboard` decomposes the PRD (FR-3), which is what writes those and
flips the PRD to `ACTIVE`.

## Non-functional requirements

**NFR-1: Cross-platform.** Windows 11 is primary; Linux and macOS
are first-class supported. Anything that works on Linux but breaks
on Windows is a bug, not a platform quirk to document around. Per
[`rules/environment-rules.md`](../rules/environment-rules.md).

**NFR-2: Zero-padded 3-digit filenames for recurring artifacts.**
`docs/design/design-review-checkpoint-NNN.md`,
`docs/test-plans/phase-NNN-exit.md`, `docs/design/PRD-<slug>-NNN.md`,
and `docs/project-plans/{project-plan,claude-code-prompts}-NNN.md` —
pad width fixed at 3 so files sort lexicographically as numeric order
through 999. PRD and plan-archive share one movement counter
(movement N → `PRD-<slug>-N`; opening N archives N−1 as
`project-plan-(N-1)`). `/design-review`, `/exit-test-plan`,
`/product-visioning`, and `/wind-down` depend on these glob shapes.

**NFR-3: Per-file frontmatter status on recurring artifacts.**
`AWAITING-DECISIONS` / `LANDED` on
`docs/design/design-review-checkpoint-*.md`;
`AWAITING-DISPOSITIONS` / `LANDED` on
`docs/test-plans/phase-*-exit.md`; `DRAFT` / `ACTIVE` /
`SUPERSEDED <date>` on `docs/design/PRD-<slug>-*.md` (latest `ACTIVE`
is the definitive movement scope). Frontmatter status is the source
of truth for the recurring-artifact lifecycle (the three status
comments in `CLAUDE.md` govern only the configuration ritual).

**NFR-4: Stage-detection and refresh-marker placeholders are
load-bearing.** Stage detection parses for exact strings:
`> _[UNMARKED — replace this line with your decision per the legend above]_`
(in `/design-review` AUDIT NOTE blocks) and `[PENDING]` (in
`/exit-test-plan` §4 / §6.N run log rows). Changing the placeholder
text breaks stage detection — audit `Step 0`, `Step S1.5`, and `Step
S1.A` of the relevant command file together. The
template-marker syntax for `/refresh-from-repository` is similarly
load-bearing. Pinned by checkpoint 002 as
`<!-- CC-TEMPLATE-BLOCK: <id> --> ... <!-- /CC-TEMPLATE-BLOCK -->`
with `<id>` a stable human-readable kebab-case identifier, and
**revised by checkpoint 004 (B1 Option A)** so each block carries
per-block **state**: `template-owned` (default, tracks upstream),
`forked` (consumer-owned), or `removed` (a tombstone for a deleted
or moved-out block). This state lives in the marker itself, in the
rules / CLAUDE.md files — which **supersedes checkpoint 002's "no
inline metadata" pin** (the marker now carries the one piece of
metadata reconciliation needs) and **removes the refresh state file
entirely** (checkpoint 004 abandoned the hash/baseline/state-file
mechanism: there is no
`.claude/claude-code-sdlc-template-refresh-state.md`, no per-block
hashes, and no baseline reference). Marker memory in the template
files is the sole reconciliation state. The marker encoding,
pinned by the Phase 2.1.A build: `template-owned` is the **bare
marker** `<!-- CC-TEMPLATE-BLOCK: <id> -->` (refresh never writes
`state=template-owned`); `forked` adds `state=forked` to the open
marker with body and `<!-- /CC-TEMPLATE-BLOCK -->` closer unchanged;
`removed` is a **closerless tombstone** — a single
`<!-- CC-TEMPLATE-BLOCK: <id> state=removed -->` comment with no body
and no closing marker. Changing the marker open/close
strings or the state vocabulary (`template-owned` / `forked` /
`removed`) requires auditing the refresh logic and all template
files that carry markers.

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

**NFR-8: File ownership doesn't overlap.** Each command's "What this
command does NOT do" section lists what the others own. `/onboard` owns
`docs/PRODUCT_VISION.md`, the in-flight `docs/PROJECT_PLAN.md` /
`docs/CLAUDE_CODE_PROMPTS.md`, their `docs/project-plans/` archives, and
PRD status transitions. `/product-visioning` authors
`docs/design/PRD-<slug>-*.md` (DRAFT). `/design-review` owns
`docs/design/design-review-checkpoint-*.md` and `docs/design/REVIEWS.md`
(and edits the tactical docs on its review-land path). `/exit-test-plan`
owns `docs/test-plans/phase-*-exit.md`. The other commands surface and
warn but never edit those files.

**NFR-9: Source/dist duplication is deliberate and audited.**
Universal content (rules, design philosophy, command files) is
kept identical between root and `cc-template/`. The duplication is
intentional (root project consumes its own template) but creates
drift risk — Phase 3 regression-test automation exists to catch
invariant-breaking drift. Pre-Phase-3 discipline: edit in
`cc-template/` first, copy up.
