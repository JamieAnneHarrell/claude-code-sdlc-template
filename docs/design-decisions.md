# Design Decisions

Technical choices made during development of this project, with
rationale. Future-Jamie and future-Claude both read this file when
investigating "why is it this way?"

`/wind-down` proposes additions to this file when a session made a
real design decision (not every session does). `/onboard` writes the
first entry recording the choices made during onboarding.

## When to record a decision here

A decision belongs in this file when:
- A future engineer (including future-Claude) might wonder "why is it
  this way?" or "why didn't we do X?"
- Multiple options were considered and the choice has consequences
  beyond a single line of code.
- A constraint or tradeoff drove the choice (performance, platform,
  cost, time-to-market) that won't be obvious from the code.

A decision does NOT belong here when:
- It's a trivial implementation choice (variable name, file location,
  helper function shape).
- It's already obvious from the architecture doc or the requirements.
- The "why" is "the framework requires it."

## Format

Each entry uses the same shape so the file scans cleanly. Pattern
borrowed from `ds-auto-dailies/docs/design-decisions.md`.

```
## <Decision title — what was decided, in one phrase>

**Decision.** What we chose, in one or two sentences.

**Why.** What drove the choice — the constraint, tradeoff, or insight
that made this the right answer.

**Why not <alternative 1>.** Why the obvious alternative didn't win.
Concrete; cite specifics, not general preferences.

**Why not <alternative 2>.** ... (only if there were genuine
alternatives)

**Scope note** (optional). Any boundary on when this decision applies
or what could re-open it.
```

---

## Source/dist subdirectory split

**Decision.** The cc-template project becomes a git-tracked
source-of-truth repo at the existing path
`c:\Users\jamie\Documents\code\cc-template\`. Inside it, a nested
`cc-template/` subdirectory holds the **distributable** — the
copy-paste seed that consumers use to start a new project. Everything
outside that subdirectory (root `CLAUDE.md`, root `README.md`, root
`docs/`, root `rules/`, root `.claude/commands/`) is source-only and
never copies into the distributable.

**Why.** The template has grown complex enough (iterative
`/design-review` and `/exit-test-plan` lifecycles, fifteen-plus
load-bearing invariants in `CLAUDE.md`, six commands with overlapping
concerns) that copy-paste-without-history became a regression risk.
Git history on the source makes changes auditable and revertible
without disrupting the consumer experience of "copy a directory
called cc-template."

**Why not subdir with the dist gitignored, rebuilt on demand.**
Smaller repo but loses the historical snapshot. We want each release
tag to point at a concrete dist state we can diff against. Tracking
the dist in git costs nothing on a content-only repo.

**Why not a sibling directory (../cc-template).** Cross-directory
builds are less self-contained and there's no single repo capturing
the full source-plus-dist state at a given commit.

**Why not a separate repo for the dist.** Two repos to keep in sync
is heavier than the problem warrants. Single repo is simpler.

**Why not stock-git `.gitattributes` with `export-ignore` (source =
dist, with stripping at archive time).** Initially proposed as the
simplest possible shape. Rejected because Jamie wanted an explicit
physical distribution directory so the shippable artifact is visible
in the file tree, not implicit in git filter rules. The
self-modifying-project framing depends on the distributable being a
concrete directory you can point at.

**Scope note.** The subdir is named `cc-template/` to preserve the
consumer-facing name (lighter brand for what consumers copy and
seed projects with). The source repo root is
`claude-code-sdlc-template/` so the two are nominally distinct
and signal different audiences (source = maintainers /
contributors; dist = consumers). Auto-memory continuity has been
reconnected manually at each rename.

---

## Downstream updates use a pull model, not push

**Decision.** When the source repo eventually goes public, downstream
projects (seeded from an earlier version of the template) update
themselves by running a shipped `/refresh-from-repository` command
that pulls the latest commands and rules from the public source repo.
The command lives in the distributable (`cc-template/.claude/commands/`)
and runs in the downstream project, not at the source.

The command is two-stage and self-modifying:

1. **`/refresh-from-repository`** — selectively pulls the latest
   versions of `.claude/commands/*.md` and `rules/*.md` from the
   public upstream into the downstream's working tree. This replaces
   the command files themselves, including `/refresh-from-repository`.
2. **`/refresh-from-repository --merge`** — runs the freshly-updated
   logic from step 1. Diffs the downstream's `CLAUDE.md` and
   `rules/*.md` against the upstream `cc-template/` subtree and
   surgically merges deltas while preserving `<!-- ONBOARD-FILL: ... -->`
   blocks (downstream's project-specific content stays intact).

**Why.** Downstream chooses when to update. No central registry of
downstream projects is required. Works for projects the maintainer
doesn't know about. The pull model is also a standard pattern for
self-updating tools.

**Why not push (a source-only `/refresh-repository <xyz>` command
that the maintainer runs to merge into a specific downstream path).**
Push requires the maintainer to know every downstream path. Downstream
has no agency over update timing. Doesn't work for projects shared
across machines or contributors. And it puts a coordination burden
on the maintainer that scales badly.

**Why the two-stage split.** The first stage replaces files
including itself; the second stage runs the new logic. This is the
standard self-modifying-tool pattern (think `pip install --upgrade
pip` then re-invoke) and avoids the "command updates itself but the
already-running invocation is stale" problem.

**Scope note.** Building `/refresh-from-repository` is Phase 2 work.
The public GitHub URL is resolved — see the "Public GitHub URL for the
source repo" decision below.

**Superseded by** the 2026-05-26 design-cleanup decisions below
(`Refresh-from-repository is single-stage with skills-first staging
on detected drift`, `Template marks its own content; reconciliation
tolerates divergence`, `Source-repo refresh syncs local cc-template/
→ root`, `Refresh progressive-disclosure flags`, `Vendored-template
lock-in as a documented use case`). The pull-model framing in this
entry still stands; the two-stage detail does not.

---

## All six commands ship at the source root, not just the recurring ones

**Decision.** Root `.claude/commands/` carries all six commands —
`/onboard`, `/bootstrap`, `/deployment-plan`, `/design-review`,
`/exit-test-plan`, `/wind-down` — identical to what the dist ships.
The source-of-truth project is itself a downstream consumer of its
own template; it runs the same commands against its own docs that
any seeded project would.

**Why.** This project is self-modifying: the dist it produces is
the dist it consumes. Keeping the command set complete at the
source means template improvements can be exercised on the source
project before they ship, and there is one canonical command shape
rather than a "source uses these, dist uses those" split that
would drift.

**Why not trim root to the three recurring commands.** Briefly
considered (the configuration commands look nonsensical on a
project that wasn't onboarded from a design intake). Rejected
because "this project IS the template" doesn't actually preclude
running `/onboard` on it — the source project has its own design
intake (the restructure design docs in `docs/design/`), its own
REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN trajectory, and will
benefit from the same configuration ritual any other project gets.
Trimming would force a "source is special" carve-out we'd then
have to maintain.

**Scope note.** The one exception: `/refresh-from-repository`
(Phase 2 work) will not target the source from the source. The
source is the upstream; pulling from itself is a no-op at best
and a footgun at worst. The command will detect that case and
refuse.

---

## Public GitHub URL for the source repo

**Decision.** The source-of-truth repo lives at
`github.com/JamieAnneHarrell/claude-code-sdlc-template` (public).
Phase 2's `/refresh-from-repository` command will bake this URL
in as the upstream constant. Renamed from the initial
`cc-template` shortly after creation to avoid
`cc-template/cc-template/` clone collisions and to position the
project's SDLC-methodology weight more accurately.

**Why.** A pull-model `/refresh-from-repository` (see the
"Downstream updates use a pull model" decision above) needs a
known public URL to pull from. Picking the URL now — before the
Phase 2 command is designed — means the design can treat the URL
as a settled input rather than a parameter. Personal-namespace
(`JamieAnneHarrell`) over a fresh org because there's no second
maintainer or sibling project yet; the name can move later by
adding a GitHub redirect.

**Why not a configurable URL with no baked-in default.** Pushes
configuration burden onto every downstream consumer for a value
that, in practice, only changes if the canonical repo moves. Bake
in the default; let consumers override only if they're pulling
from a fork.

**Why not a dedicated GitHub org (e.g. `jah-tools/cc-template`).**
Premature. There's one maintainer and one project today. A
personal-namespace URL is the simplest thing that works and is
trivially redirectable via GitHub's repo-transfer mechanism if a
second project ever shares the org.

**Scope note.** Repo name `claude-code-sdlc-template` and local
source-repo working dir `claude-code-sdlc-template/` match (clean
clone ergonomics); the dist subdir inside the repo stays
`cc-template/` (lighter consumer-facing brand). See the
"Source/dist subdirectory split" decision's scope note.

---

## Onboarding outputs — language, multi-agent mode, scope, license

**Decision.** `/onboard` ran on 2026-05-24 with these choices:

- **Language / runtime.** None. The project ships markdown only.
- **Project name vs. slug convention.** `claude-code-sdlc-template`
  is the overall project / project branding (used in CLAUDE.md
  banner, README title, repo URL); `cc-template` is the
  distributable release (used when referencing the dist
  subdirectory and in consumer-facing copy).
- **Multi-agent mode.** `explore-plus-plan`. Plan subagent earns
  its place because of the load-bearing invariant complexity in
  CLAUDE.md and the recurring-command lifecycles. Implementation
  stays sequential — no worktree spawning.
- **Scope statements.** No GUI in MVP. No ML models in MVP.
  Cross-platform with Windows 11 primary, Linux/macOS first-class
  supported. No language runtime in MVP.
- **License — dual.** The `cc-template/` distributable ships
  under MIT. Everything else in the repo ships under
  CC BY-NC-ND 4.0.

**Why.** The dual license lets the parts of the repo that are
*meant to be copied and built on* (the distributable) be freely
incorporated into consumer projects — including commercial ones
— while the parts that are *not meant to be redistributed* (the
project-management docs, design intake, root CLAUDE.md, root
README) stay protected from repackaging.

**What this means for consumers.** Anyone seeding a new project
from `cc-template/` can copy it, modify it, ship it, and sell
the resulting product — the dist is fully permissive under MIT,
attribution required (retain the MIT LICENSE text). The
source-only docs in the rest of the repo are reference material
only; they can be read but not redistributed or modified for
distribution.

**Scope note.** No secrets-and-credentials-hygiene NFR was added
because the design has no DB and no credential storage.

---

## `/exit-test-plan` BLOCKED tests auto-re-add to next addendum, never dropped

**Decision.** Every `[BLOCKED]` row in the newest run log of an
`/exit-test-plan` plan gets re-added to the next §6.N addendum
automatically — never dropped. The cycle continues until the
test reaches a non-BLOCKED result (PASS / FAIL / SKIP). The plan
cannot flip to `LANDED` while any `[BLOCKED]` row remains in the
newest run log.

**Why.** A blocker is a real signal — the system couldn't run the
test. Dropping it (or making re-add an operator-toggled choice
each addendum) creates a path where a real blocker silently exits
the plan because someone forgot to re-queue it. Auto-re-add makes
"the plan landed" mean "every test ran, or was explicitly accepted
as SKIP."

**Why not ask the operator each addendum which BLOCKED rows to
re-add.** Easy to forget; defeats the safety guarantee. The
initial Phase 1.1 implementation proposed this and was corrected
mid-session.

**Why not drop BLOCKED after one addendum cycle.** A real blocker
can outlast a single fix attempt; dropping loses the signal.

**Why not treat BLOCKED like SKIP for landing purposes.** SKIP is
an explicit tester decision recorded at run time; BLOCKED is the
system reporting the test could not run. Collapsing them would
let unattended blockers ride through to landing.

See `.claude/commands/exit-test-plan.md` Step S1.A Step 2 and
Step S2.2 BLOCKED rows for the implementation, and Phase 1.1's
prompt in `docs/CLAUDE_CODE_PROMPTS.md` for the deviation footer.

## `cc-template/CLAUDE.md` reads correctly in both pre- and post-onboard states

**Decision.** The dist `cc-template/CLAUDE.md` placeholder is
worded so it reads correctly whether a reader is encountering it
pre-onboard (placeholder state) or post-onboard (the seeded
`CLAUDE.md` of a downstream project). The Reading-order header has
no `(configured projects)` parenthetical, and the rules-file
descriptions have no "filled in by `/bootstrap`" / "filled in by
`/onboard`" qualifiers.

**Why.** `/onboard` and `/bootstrap` keep "collaboration rules and
references unchanged," so any pre-onboard qualifier persists
forever in every downstream project's `CLAUDE.md`. Wording that's
stable in both states avoids the stale-noise problem with no
command-side logic to add. The configuration ritual is documented
in the banner above the Reading-order list — that's where
pre-onboard readers find context.

**Why not update `/onboard` / `/bootstrap` to strip qualifiers
after the fact.** More invariants, more work, same outcome. KISS
wins.

**Why not keep the pre-onboard qualifiers.** They served pre-onboard
readers but actively confused post-onboard readers, and the latter
audience is much larger (every downstream session).

Folded into Phase 1.2 cleanup (R4c).

---

## `/refresh-from-repository` is single-stage with skills-first staging on detected drift

**Decision.** `/refresh-from-repository` runs as a single invocation
that pulls the latest commands and rules from upstream and then runs
the merge logic loaded at invocation start. There is no required
two-stage user flow. The command does, however, detect "merge-logic
drift" automatically and stage skills-first when the locally-loaded
refresh logic is N generations behind upstream.

**Why.** The two-stage `pip install --upgrade pip` analogy in the
"Downstream updates use a pull model" entry above cited a footgun
that doesn't actually exist for Claude Code commands. Slash commands
are read fresh from disk at each invocation, not cached in a
long-lived process. The currently-executing invocation continues
following the instructions it loaded at start; replacing the
command file on disk only affects *future* invocations. One
invocation can safely pull-then-merge.

**How drift is detected.** Before pulling anything, the command
fetches upstream's current `refresh-from-repository.md`, compares
the version-stamp / marker-convention metadata to the locally-loaded
copy, and on drift pulls only `.claude/commands/*.md` files and
surfaces "your refresh logic was N versions behind; latest commands
now loaded — re-invoke to merge." Two well-chosen signals beat
either no signal (silent failure on schema drift) or a flag the
consumer doesn't know to use.

**Why not always two-stage.** Adds an invocation to the routine path
for a problem that doesn't exist most of the time. KISS — auto-detect
the case that needs staging; default path is one invocation.

**Why not require the consumer to use `--refresh-skills-only`.** The
consumer often won't know to worry about merge-logic drift; auto-
detect and prompt. The flag exists as a manual override for power
users (see "Refresh progressive-disclosure flags" below).

---

## Template marks its own content; reconciliation tolerates divergence

**Decision.** Template-owned content in rules files (and CLAUDE.md
post-onboard) is wrapped in template-owned marker blocks. Markers
carry "DO NOT EDIT — move elsewhere to personalize" language.
`/refresh-from-repository` reconciles at the block level using a
three-input matrix: downstream-current, upstream-current, and a
baseline (what upstream looked like the last time this downstream
sync'd). Each block carries enough metadata for the refresh to
detect "user edited inline" — a content hash or recorded baseline
content. Mismatch surfaces; refresh doesn't silently overwrite.

**Consumer contract (refresh's promise):**

- Don't edit inside a template marker block. Refresh expects to
  replace its contents.
- To opt out: move the block out of the template region into your
  own personalized-rules section, or delete the block entirely.
  Refresh respects the absence as your decision.
- Personalized rules are yours forever — refresh never touches them,
  and they don't get upstream improvements by design.
- Inline edits are detected and surfaced, not silently overwritten.

**Why.** Two earlier proposals failed. (a) "User wraps opt-outs in
LOCAL markers" is unenforceable — consumers wholesale-delete rules
they disagree with (Android project deletes "iPhone, not Android"),
refresh has no signal it was deliberate, silently re-adds the rule.
(b) "Diff-each-time without a baseline" can't distinguish
"block-new-since-baseline" from "block-the-user-removed." Marking
*our* content (which we can enforce) and tracking a baseline (which
we can stamp ourselves) is the smallest mechanism that handles every
deletion / edit / personalization case the consumer can throw at it.

**Why not git-3-way as the only mechanism.** Requires upstream to
maintain stable release tags AND downstream to track which tag it
sync'd from. Self-contained per-block metadata is simpler for v1
and doesn't preclude git-3-way as a later iteration.

**Semantic conflict detection is the executing session's job.** When
upstream's new blocks land alongside downstream's personalized
content, the Claude session running `/refresh-from-repository` reads
both and surfaces real semantic conflicts ("upstream's new rule R12
says always-X; your personalized rule says never-X") using language
understanding, not a heuristic engine. The command spec instructs
the session to do this analysis directly.

**Scope note.** This entry settles the contract and the matrix; the
mechanism (algorithm, marker syntax, state file) was settled by
checkpoint 002 — see "`/refresh-from-repository` reconciliation
mechanism" below.

---

## Source-repo refresh syncs local `cc-template/` → root

**Decision.** When `/refresh-from-repository` runs in the source-of-
truth repo, "upstream" is the local `./cc-template/` subdirectory
rather than the public GitHub URL. Detection is trivial: the
source-of-truth repo is the only one with a `cc-template/` subdir
at cwd.

**Why.** The source repo itself is a downstream consumer of its own
template; manual root↔dist sync was previously handled by hand or
hypothetically by a "source-only release helper" command (open
question now closed). Folding source-mode behavior into
`/refresh-from-repository` keeps one mental model for "pull
template changes into this project" and avoids adding a fourth
configuration-ritual command.

**Bonus use case — vendored template lock-in.** A consumer who
wants to pin to a specific upstream version can vendor `cc-template/`
into their own repo as a subdirectory and use the same source-mode
sync to apply that vendored copy to root. Upgrading the vendored
copy is a deliberate `git pull` from upstream into the vendored
subdir; the routine sync from subdir to root is a stable, audited
operation. Documented configuration-management pattern.

**Why not refuse at source.** The current Phase 2.1 prompt described
a refusal stance at source ("pulling from itself is a footgun").
Refusing leaves the manual-sync problem unsolved and forecloses the
vendored-lockin pattern. Sync-from-subdir is the right behavior; the
"footgun" framing was based on misreading the source repo's
relationship to its own dist.

---

## Refresh progressive-disclosure flags

**Decision.** `/refresh-from-repository` ships with two
progressive-disclosure flags that name what they suppress:

- `--refresh-skills-only` — pulls only `.claude/commands/*.md`; no
  merge attempted. Manual override for the auto-detected
  drift-staging path; useful when a consumer wants to inspect
  upstream's new refresh logic before letting it touch their
  rules.
- `--no-claudemd` — skips CLAUDE.md from the merge. Rules and
  commands still refresh. For consumers who've heavily-customized
  CLAUDE.md outside the marker model.

**Why.** Default behavior covers the common case (per
`rules/design-philosophy-rules.md` progressive disclosure). Both
flags exist as access-not-foreground escape hatches; neither is
required to use the command.

**Why not more flags.** Each flag is one more thing the consumer
might reach for inappropriately. Two real escape hatches earn their
place; speculative flags ("--dry-run", "--from-tag",
"--exclude-rules", etc.) are deferred to evidence-of-need.

---

## Vendored-template lock-in as a documented use case

**Decision.** Consumers who want to pin to a specific upstream
template version vendor the `cc-template/` subdirectory of an
upstream commit into their own repo and use the source-mode behavior
of `/refresh-from-repository` (see "Source-repo refresh syncs local
`cc-template/` → root" above) to sync that vendored copy to root.

**Why.** The mechanism already exists for the source-of-truth repo's
own root↔dist sync. Naming it as a supported configuration-management
pattern costs nothing and serves consumers with audit or
reproducibility requirements that would otherwise force them off the
template.

**Why not a separate command.** Same mechanism, different motivation.
A dedicated command would force consumers to learn two tools for one
operation and would drift relative to source-mode refresh.

**Scope note.** Upgrading the vendored copy is a deliberate consumer
action (`git fetch` + copy / `git subtree` / similar) — not
something `/refresh-from-repository` automates. The command's job is
to apply whatever the vendored subdir currently holds.

---

## `/refresh-from-repository` reconciliation mechanism

**Decision.** Option D hybrid: per-block content hashes detect inline
edits, file-level baseline references detect deletions, three-way
comparison (downstream-current / upstream-current / baseline) drives
the merge. Template content is wrapped in
`<!-- CC-TEMPLATE-BLOCK: <id> --> ... <!-- /CC-TEMPLATE-BLOCK -->`
markers (id-only kebab-case, file-scoped, no inline metadata).
Per-block hashes and baseline live in a single machine-managed
`.claude/claude-code-sdlc-template-refresh-state.md` carrying four
sections: Upstream baseline, Block hashes, Refresh logic version,
Upstream directives. Source-mode uses the same algorithm with a
subtree content-hash of local `cc-template/` as the baseline.

**Why.** Hash and baseline together resolve the consumer-edit matrix
— inline edit, deletion, never-existed, reorder — that hash-alone or
baseline-alone leave ambiguous (the canonical "Android deletes
'iPhone, not Android'" case silently re-adds under hash-only).
Pushing hashes into the state file rather than the marker is what
makes coarse-grained wrapping (one block per top-level rule section)
bounded to ~5-7% added line count, important because Phase 1.2 just
came up short of its rules-file cleanup target.

**Why not per-block hash alone (Option A).** Can't distinguish
block-never-existed-yet from block-deliberately-deleted; silently
re-adds rules consumers removed on purpose.

**Why not file-level baseline alone (Option B).** Solves deletion
but requires re-fetching upstream's historical file content per
refresh to detect inline edits. Local hash carries that signal
cheaply.

**Why not git-3-way against tagged upstream releases (Option C).**
Forces release-tag discipline on a one-maintainer upstream; exposes
git conflict-marker syntax to consumers reading `.md` files. Real
maintenance cost for negligible upside.

**Why not inline `hash=` on each marker.** Re-inflates rules-file
line count after Phase 1.2's cleanup. State-file storage keeps
markers minimal.

**Why not baseline metadata in user-facing files.** Pollutes the
rules-reading experience consumers do every session per CLAUDE.md.
`.claude/` is the conventional machine-state location and consumers
rarely look there.

**Scope note.** Refresh matches blocks by `id`, not file position —
consumer reordering is preserved (R2c). Inline-edit divergences
surface to the running session one block at a time with three
choices: accept upstream / keep downstream / hand-merge (R2b).
Unrecognized local block IDs surface a `--refresh-skills-only`
suggestion (R2a). First source-mode invocation in a repo
content-inspects the `cc-template/` subdir to confirm it's this
template and caches the determination in CLAUDE.md (N1). Marker
syntax, state-file path, and state-file schema are load-bearing per
NFR-4 — future changes require auditing refresh logic and every
file that carries markers. Full disposition log:
[design-review-checkpoint-002.md](design/design-review-checkpoint-002.md).

---

## `/refresh-from-repository` consumer-boundary partition

**Decision.** CLAUDE.md is partitioned post-onboard: Collaboration-
rules section and Reading-order section are template-owned (wrapped
in CC-TEMPLATE-BLOCK markers); banner, Project-specific context,
and Load-bearing invariants are consumer-owned (free regions;
refresh never touches), with `--no-claudemd` as the escape hatch
for consumers who've heavily customized. Cross-file block migration
is not tracked by default, with one exception: when a
consumer-deleted block's hash matches a known block in another of
the six shipped rules files, refresh offers to apply upstream's
update. Refresh pre-flights by fetching upstream's
`refresh-from-repository.md` to check for merge-logic drift AND
reads the state file's Upstream directives section so upstream can
force `--refresh-skills-only` via a directive.

**Why.** The CLAUDE.md partition lets refresh push improvements to
the load-bearing collaboration infrastructure (Collaboration-rules,
Reading-order) without overwriting the per-project content consumers
customize. The cross-file migration exception catches the case
where a consumer reorganized a template rule into their own
personalized-rules file — without it, that consumer is locked out
of upstream's improvements to that rule forever, which is
inconsistent with the "respect deletion, surface inline edits"
contract elsewhere in the design.

**Why not wrap Load-bearing invariants too.** Hardest section to
partition cleanly — template-defined invariants and project-defined
ones interleave. The `--no-claudemd` escape hatch covers consumers
who need upstream's invariant updates merged.

**Why not track arbitrary cross-file moves.** Consumer can paste
any content anywhere; tracking arbitrary moves requires either an
opt-in `TEMPLATE-MIGRATED-FROM` marker consumers wouldn't know to
add, or expensive cross-file hash-matching across every consumer
file on every refresh. Scoping the exception to the six shipped
rules files is bounded and matches the realistic migration pattern.

**Scope note.** "Six shipped rules files" =
`coding-session-rules.md`, `design-philosophy-rules.md`,
`multi-agent-rules.md`, `project-rules.md`, `environment-rules.md`,
`testing-rules.md`. Adding a seventh in a future phase extends the
migration-detection set.

---

## Retire "Before running this prompt" header-block from `/onboard`'s `CLAUDE_CODE_PROMPTS.md` spec

**Decision.** `/onboard`'s spec for `docs/CLAUDE_CODE_PROMPTS.md`
no longer plants a "Before running this prompt:" header-block note
on forward-looking prompts. Between-phase `/design-review`
checkpoints are signaled only via first-class
`## Design Review Checkpoint — pre-Phase-N` entries in
`docs/PROJECT_PLAN.md`. Landed prompts (1.1, 1.2) keep their
historical-state header blocks per the
don't-edit-landed-prompts-retroactively convention; forward-looking
Prompts 2.2 and 3 lost theirs at checkpoint 002 landing.

**Why.** The header-block conflated two things: (a) a gate
assertion the prompt has no way to verify — false positives caught
only mid-work; (b) a next-session reminder that belongs in TODO.txt
at pickup time, not embedded in the prompt body. PROJECT_PLAN.md
first-class checkpoint entries already carry the authoritative
between-phase signal — they live in the phase-queue file that
session-start reading-order points to. Reproducing the same signal
inside the prompt body duplicates the source of truth and dates
fast when phases reshuffle.

**Why not rewrite the header-block as a gate-check** ("if the
pre-Phase-X review hasn't landed, stop and run it first"). Still
relies on prompt-body content to substitute for what PROJECT_PLAN.md
already does discoverably. KISS — pick one signal location.

**Why not keep landed-prompt header-blocks AND forward-looking
ones.** The historical-record argument applies only to landed
prompts (record of what context existed when the prompt was run).
Forward-looking prompts are not history; they're upcoming work,
and the signal belongs in the discoverable place.

**Scope note.** Spec edit applies to root and dist
`.claude/commands/onboard.md`. Tracked as T2 from checkpoint 002's
Follow-up actions; spec edit deferred to a separate session.
