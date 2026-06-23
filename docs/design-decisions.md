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

**Write forward-looking.** State the decision and why the *current*
design is right; don't narrate the earlier approach that was wrong — the
git history records what changed. `Why not <alternative>` covers genuine
design alternatives, not a corrected mistake reframed as one.

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
load-bearing invariants in `CLAUDE.md`, commands with overlapping
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

## All commands ship at the source root, not just the recurring ones

**Decision.** Root `.claude/commands/` carries all commands,
identical to what the dist ships.
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
post-onboard) is wrapped in `CC-TEMPLATE-BLOCK` marker blocks, matched
by id. `/refresh-from-repository` reconciles each block with a two-way
compare (downstream-current vs upstream-current); a block that diverges
or is absent with no tombstone triggers a one-time question whose answer
is recorded in the marker itself. Refresh never silently overwrites a
consumer's edit and never re-adds a block the consumer removed.

**Consumer contract (refresh's promise):**

- Edit inside a template block freely. The first time refresh sees the
  divergence it asks once — take upstream, or keep mine (the block
  becomes `forked` and is never asked about again).
- To opt out: delete the block (refresh asks once, then writes a
  `removed` tombstone and never re-adds it) or fork it. Either way the
  absence/ownership is respected as your decision.
- Personalized (forked or relocated) content is yours forever — refresh
  never touches it, and it doesn't get upstream improvements by design.
- Inline edits are detected and surfaced, not silently overwritten.

**Why.** Two earlier proposals failed. (a) "User wraps opt-outs in
LOCAL markers" is unenforceable — consumers wholesale-delete rules they
disagree with (Android project deletes "iPhone, not Android"), and a
naive refresh has no signal it was deliberate, so it silently re-adds
the rule. (b) "Diff-each-time" can't distinguish a brand-new upstream
block from one the user removed. Both failures are about *memory of a
deliberate choice*. Marking *our* content (which we can enforce) plus
recording the consumer's one-time answer in the marker (`forked` /
`removed`) is the smallest mechanism that handles every deletion / edit
/ personalization case — and it keeps that memory in the files
themselves rather than a sidecar (see "reconciliation: Option A").

**Semantic conflict detection is the executing session's job.** When
upstream's new blocks land alongside downstream's personalized content
in the same file, the Claude session running `/refresh-from-repository`
reads both and surfaces real semantic conflicts ("upstream's new rule
says always-X; your adjacent rule says never-X") using language
understanding, not a heuristic engine. The command spec instructs the
session to do this directly, scoped to the file being merged.

**Scope note.** This entry settles the contract; the marker syntax and
the stateless reconciliation mechanism are settled by
"`/refresh-from-repository` reconciliation: Option A" below. The
contract here is unchanged from checkpoint 002 — only the mechanism
beneath it moved from a stored baseline to marker-state + ask-once.

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

## `/refresh-from-repository` reconciliation: Option A (stateless marker-state + ask-once)

**Decision.** Reconciliation moves from checkpoint 002's Option D
(per-block content hash + file-level baseline in a sidecar state file
+ three-way merge) to **Option A**: template-managed blocks carry
their own state in-file (`template-owned` / `forked` / `removed`);
refresh is a two-way compare + ask-once-and-record, matched by block
id, with the executing Claude session performing the surgical merge;
the command file's version stamp is the sole drift lever. No state
file, no per-block hashes, no baseline. Adopted at design review
checkpoint 004 (B1), 2026-06-04.

**Why.** Patching Option D's seed path surfaced that the whole bug
class (wrong-side baseline in the re-seed branch and the pre-marker
migration; "when is the baseline established") existed *because* the
baseline + hashes were stored in a sidecar file. Moving the memory
into the files themselves (a tombstone / `forked` flag set by asking
once) and letting the LLM do the compare + merge it already owns
dissolves the class outright and removes the state file, the hashes,
the `git hash-object` recipe, the `last-synced` reference, and the
baseline.

**Why not keep Option D and fix the seed path.** It retains
deterministic provenance — the one thing Option A trades away — but
keeps the correctness-and-maintenance surface this review existed to
chase: keeping seeded/shipped hashes correct, cross-platform
LF/`hash-object` discipline, committing the state file so the
baseline survives a clean clone, and the recurring drift risk across
all of it. Per rule 4, Option A is the simpler alternative; the
provenance trade (refresh asks the consumer once instead of inferring
from a baseline, with "take upstream" the git-recoverable safe
default) was accepted 2026-06-04.

**Scope note.** The Option-D-era entries this replaced have been
reconciled (Phase 2.1.A): the two whose entire substance was the
abandoned mechanism — "reconciliation mechanism" and "content hashing
uses `git hash-object`" — were removed and recorded in
`docs/open-questions.md` § Abandoned Approaches; the partially-affected
ones ("Template marks its own content", "consumer-boundary partition",
and the build/dogfood entry) were rewritten to their surviving content.
The surviving checkpoint 002 decisions (marker syntax, coarse-grained
wrapping, CLAUDE.md merge partition, "template marks its own content")
carry forward unchanged.

---

## `/onboard` preserves the `briefing-rule` marker when rewriting `multi-agent-rules.md`

**Decision.** When `/onboard` rewrites `multi-agent-rules.md` for the
chosen multi-agent mode, it preserves the `briefing-rule`
`CC-TEMPLATE-BLOCK` marker pair verbatim rather than emitting an
unmarked file. Adopted at design review checkpoint 004 (N1),
2026-06-04.

**Why.** Under the Option A reconciliation model, markers are
memory-bearing (they carry a block's `template-owned` / `forked` /
`removed` state), so a dropped marker loses provenance, not just
formatting. `briefing-rule` is the one refresh-managed block in
`multi-agent-rules.md` (mode content is consumer-owned and unmarked),
so preserving it at onboarding keeps the block trackable from day one
instead of relying on a later refresh to re-establish it.

**Why not rely on pre-marker-migration self-heal.** The first
`/refresh-from-repository` can re-insert a missing marker, but that
leaves a window where a freshly-onboarded project has no marker at
all and couples correctness to running refresh. Instructing
`/onboard` to preserve the marker is the tighter guarantee; the
self-heal path remains a backstop.

**Scope note.** Closes the former Known Limitation "`/onboard` does
not preserve CC-TEMPLATE-BLOCK markers." End-to-end behavior is
confirmed when the Phase 2.1.A source-mode dogfood runs.

---

## `/refresh-from-repository` consumer-boundary partition

**Decision.** CLAUDE.md is partitioned post-onboard: the
Collaboration-rules and Reading-order sections are template-owned
(wrapped in `CC-TEMPLATE-BLOCK` markers); the banner, Project-specific
context, and Load-bearing invariants are consumer-owned (free regions
refresh never touches), with `--no-claudemd` as the escape hatch for
consumers who've heavily customized. Refresh works **within its own
boundaries** — the blocks it owns by id, per file — and does not scan
across files to track relocated content.

**Why.** The partition lets refresh push improvements to the
load-bearing collaboration infrastructure (Collaboration-rules,
Reading-order) without overwriting the per-project content consumers
customize.

**Why not wrap Load-bearing invariants too.** Hardest section to
partition cleanly — template-defined invariants and project-defined
ones interleave. The `--no-claudemd` escape hatch covers consumers
who need upstream's invariant updates merged.

**Why not track cross-file relocations.** A consumer can move a
template rule into their own section or file. Refresh does not chase
it — detecting relocations means scanning every consumer file on every
refresh (false-positive-prone) and contradicts the within-our-own-
boundaries principle. A relocated block is simply `forked`/owned now;
its original location gets a `removed` tombstone (or is re-added) via
the normal ask-once. The consumer trades upstream updates for ownership
— the same deal as any forked block.

**Scope note.** "Six shipped rules files" that carry template-owned
markers = `coding-session-rules.md`, `design-philosophy-rules.md`,
`multi-agent-rules.md`, `project-rules.md`, `environment-rules.md`,
`testing-rules.md`. An earlier Option-D design offered a hash-based
cross-file migration upgrade (checkpoint 002 R2d); it was dropped with
the hash mechanism — see § Abandoned Approaches in
`docs/open-questions.md`.

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

**Scope note.** Spec edit landed 2026-05-27 (T2 from checkpoint
002's Follow-up actions, applied to root and dist
`.claude/commands/onboard.md`). Partially superseded by the next
entry ("Design reviews are first-class phases and prompts"): the
header-block retirement stands, but the
"first-class `## Design Review Checkpoint — pre-Phase-N` entry in
PROJECT_PLAN.md" form named here is itself retired in favor of
design reviews being numbered phases. Forward-looking signaling
now lives at parity with feature phases/prompts.

---

## Design reviews are first-class phases and prompts

**Decision.** A planned design review is a **numbered phase** in
`docs/PROJECT_PLAN.md` and a **matching numbered prompt** in
`docs/CLAUDE_CODE_PROMPTS.md`, with shared minor-version
numbering (Phase 2.2 ↔ Prompt 2.2). Inserting a design review
between Phase X.Y and Phase X.(Y+1) renumbers the downstream
phase to Phase X.(Y+2) and so on within the same minor family
— no sub-numbering like X.Y.5. The phase carries the standard
Goal / Deliverables / Exit-criteria shape (deliverable: a
LANDED `docs/design/design-review-checkpoint-NNN.md`). The
corresponding prompt has a minimal body: Read first / Scope:
"Run `/design-review`" / Exit criteria / revisions footer.

Three older forms are retired by this convention:

1. "Before running this prompt" header-block preambles inside
   another prompt's body (already retired at checkpoint 002 R5
   landing; T2 spec-edit landed 2026-05-27).
2. The one-line "`**Design review checkpoint:** before Phase N+1
   begins. Run `/design-review`.`" marker that `/onboard`'s old
   spec inserted into PROJECT_PLAN.md after a phase's exit
   criteria. Gone — design reviews are no longer "between"
   phases.
3. The freestanding `## Design Review Checkpoint — pre-Phase-N`
   block in CLAUDE_CODE_PROMPTS.md, sitting between two
   prompts. Gone — that block becomes its own `## Prompt N`
   entry.

**Why.** The three retired forms each duplicated a signal
across surfaces with different shapes — a marker in
PROJECT_PLAN.md and a preamble or freestanding block in
CLAUDE_CODE_PROMPTS.md. Different shapes meant the eye scanned
past them differently than it scanned feature work, and the two
representations drifted. Making design reviews first-class
phases/prompts puts them in the same flow as feature work:
they're numbered, they appear in the phase queue and prompt
queue Jamie reads at session start, and TODO.txt references
them by ID (`Prompt N`) like any other prompt. Jamie's framing:
"force it into the flow, because then it gets treated just like
a prompt because it IS a prompt and step in the project plan."

The convention also resolves a recurring confusion: when a
between-phases marker or freestanding block sits next to a
prompt, the *body* of the next prompt sometimes restated the
signal ("Before running this prompt:..."), which would then get
pasted into the running Claude session as if it were
prompt-instruction content — but it was human-consumption
guidance about *whether* to run the prompt. Removing the
duplicate signaling layer removes the temptation.

**Why not keep the between-phases marker form.** Reproduces the
signal in two locations with two shapes. Reading PROJECT_PLAN.md,
the eye scans by phase header — adjacent markers between phases
are easy to miss and don't show up the same way in tooling that
treats the file as a phase queue.

**Why not full Prompt shape (Read first / numbered Scope items /
Constraints / Exit criteria) for design-review prompts.** A
design review is one slash-command invocation; numbered scope
items would invent structure. Minimal body is honest about what
the prompt does — point at the right Read-first context and
invoke `/design-review`.

**Why not sub-numbering like X.Y.5 for inserted design
reviews.** Sub-numbering would create a second-class tier of
phases. Renumbering downstream phases is cheap (markdown edits;
no code paths depend on phase numbers) and preserves the
"design reviews are equal citizens" property.

**Why not retroactively reshape this project's landed Phase 2.1
design-review block.** Don't-edit-landed-prompts-retroactively
convention. The existing PROJECT_PLAN.md "`## Design Review
Checkpoint — pre-Phase-2.1 (LANDED 2026-05-26)`" block stays as
historical record. Future design reviews planned for this
project use the new convention.

**Scope note.** Spec lives in `.claude/commands/onboard.md`
(root + cc-template mirror per NFR-9), landed 2026-05-27.
Downstream impact for the queued "Artifact-boundary commands →
`/wind-down`" OQ user story (Thread 2): when that work lands,
`/design-review` Stage 2 landing's "propose-a-future-checkpoint"
behavior will produce design-review phases under this
convention (numbered, with a matching prompt) rather than
inserting standalone marker blocks. No spec edits required to
`/design-review` itself as part of this convention shift —
`/design-review` Stage 1 still reads checkpoint inputs and
writes `docs/design/design-review-checkpoint-NNN.md` the same
way regardless of how its trigger was scheduled.

---

## `REJECTED` is a first-class `/design-review` disposition shape

**Decision.** The `/design-review` AUDIT-NOTE legend has five shapes,
not four: Accepted / Accepted with caveats / Defer Approved /
DECISION / `REJECTED: <reframing prose>`. REJECTED means no listed
recommendation was satisfactory; Stage 2 records it verbatim in the
Disposition log and treats it as an automatic open-another-round
trigger, with the rejection prose as the next addendum's reframe
input (Stage 1 re-authors the finding from the objection rather than
inventing fresh alternatives). Landed in both command copies plus
`CLAUDE.md` § Load-bearing invariants (checkpoint 003 Addendum 1 R11).

**Why.** Jamie used REJECTED five times in checkpoint 003 Round 1
before it existed in the spec. The behavior was well-defined but
unencoded — a fresh downstream Claude would hit REJECTED at S2.1,
classify it "malformed," and block the round. Formalizing makes the
organic shorthand portable to fresh projects.

**Why not leave it as session shorthand.** Future-Claude in a fresh
project has no memory of the convention; the spec is the only
teacher. Unencoded, it breaks stage detection on first contact.

**Why not add it to the legend only (skip the S2.5 auto-trigger).**
A REJECTED carrying no explicit "investigate X" directive would read
as land-eligible, contradicting its meaning. The auto-trigger is what
makes REJECTED self-completing.

**Scope note.** Step 0's placeholder check is unchanged — REJECTED is
a non-placeholder marking. Whether `/exit-test-plan` gains an
analogous REJECTED shape is left open (its walk disposes run-log
entries, not finding recommendations, so the analog is less direct).

---

## Disposition walks triage-then-batch, not confirm-each

**Decision.** `/design-review` S2.2 and `/exit-test-plan` S2.2 scan a
whole round's markings first, then surface only the genuinely
ambiguous ones (malformed / internally contradictory caveats / scope
conflict between findings) instead of restating and confirming each
marking one at a time. The two skills differ by input shape:
`/design-review` records unambiguous markings silently in one batch
(Jamie pre-writes each decision, so her writing IS the confirmation);
`/exit-test-plan` *proposes* dispositions, so it presents the whole
batch as one table for a single confirmation. BLOCKED rows in
`/exit-test-plan` are definitive — recorded without discussion, since
a blocked test mandates a follow-up addendum regardless of its
path-to-unblock. Landed checkpoint 003 Addendum 1 N6.

**Why.** Checkpoint 003's own Stage 2 walk produced 16 restate-and-
confirm round-trips for 16 already-settled markings — pure busywork.
A human's written input is authoritative; re-eliciting it wastes the
session.

**Why not record everything silently in both skills.** In
`/exit-test-plan` Claude proposes the dispositions rather than reading
Jamie's pre-written ones, so a confirmation surface is still needed —
but one batched confirmation, not N.

**Why not extend triage-then-batch to `/exit-test-plan`'s S2.3
out-of-scope observations.** Those are larger, distinct threads that
usually warrant individual discussion; the triage step would route
them to "surface individually" almost every time, so batching there
adds spec wording without changing behavior.

**Scope note.** N6 landed spec-only (checkpoint 003 Option B); no
`design-philosophy-rules.md` principle was added.

---

## Rules-read reliability — foregrounding + skills-own-rituals, not hook enforcement

**Decision.** Reliable session-start rules-read is addressed by three
complementary, already-landed changes in the distributable rather than
by runtime enforcement:

1. The rules-read directive is the **first block of
   `cc-template/CLAUDE.md`**, above the configuration banner — "before
   you respond to the first message, read both rules files end-to-end;
   confirm you have read and understood them" (checkpoint 003 B2-A1).
2. The Collaboration-rules section **no longer summarizes the rules by
   topic**, so Claude cannot pattern-match the summary as already
   absorbed and skip the read (B1).
3. **Skills own the rituals they exercise** — the "Skills own rituals"
   principle in `project-rules.md` plus the operational gate in
   `coding-session-rules.md` (rule 9: invoke the owning skill instead
   of inlining its ritual) route skill-owned behavior to the owning
   skill, so the relevant rule context arrives just-in-time rather
   than competing for attention at session start (checkpoint 003
   N3-A1 + N4-A1; e.g. commit handoffs → `/wind-down`).

These map to the four "directions" explored in the now-retired
open-questions story: Directions 1, 2, and 4 landed; Direction 3
(hooks) is rejected; Direction 4b (session-entry skill prompt) is
deferred.

**Why.** The skip-prone behavior had several compounding causes (a
buried instruction, a summary that read as sufficient priors, and
enforcement that relied on judgment biased toward "respond now");
addressing any one alone was judged unlikely to fix it. The three
landed changes are nearly free, portable across every Claude surface,
and reinforce each other.

**Why not Direction 3 (a `UserPromptSubmit` hook that unconditionally
reads both rules files on the first message).** Hooks are
CLI-feature-specific; portability across the IDE-extension / desktop /
web surfaces named in `environment-rules.md` is unvalidated, and
"first message of session" detection is non-trivial. The template's
"reliable out of the box" stance is better served by the nearly-free
foregrounding + skills-own-rituals approach first.

**Why not Direction 4b (ask the user's intent at session start and
load the matching skill's rules).** A friction step on every session
start; risks feeling bureaucratic and is redundant when the user's
first message already states intent.

**Scope note.** Re-open Direction 3 or 4b only if, after these landed
measures have been exercised in practice, Claude still reliably skips
the end-to-end rules re-read at session start. Direction 4's
skills-own-rituals form covers only rules with a canonical skill home
(commit handoff and wind-down rituals → `/wind-down`; manual
phase-exit walkthrough → `/exit-test-plan`); session-pervasive rules
(KISS, rules 1–6, the rule-4 simpler-alternative self-check) keep
living in the always-loaded rules files. Landed in the distributable
(`cc-template/`): B1 + B2-A1 (Session B), N3-A1 + N4-A1 (Session A).
Full disposition: [design-review-checkpoint-003.md](design/design-review-checkpoint-003.md).

---

## `/wind-down` owns the open-questions ↔ design-decisions reconciliation

**Decision.** `/wind-down` Step 3 is the session-close maintainer of
the `open-questions.md` ↔ `design-decisions.md` relationship. It
scans — non-skippably — for questions resolved this session, moves
them *out* of `open-questions.md` and records them in
`design-decisions.md`, and reads each file's own format section
(`design-decisions.md` § Format / § When to record; `open-questions.md`
§ Categories) before editing rather than reproducing the shape from
memory. `/wind-down` is the session-close *maintainer*, not the only
*writer* of `design-decisions.md` — `/onboard` (seeds it) and
`/design-review` Stage 2 (routes its own dispositions) legitimately
write it at their own boundaries.

**Why.** Session E surfaced two live gaps: `/wind-down` left resolved
stories sitting in `open-questions.md` (only the inbound add was
encoded, never the outbound move), and Claude had to rediscover both
files' formats at runtime. The skill that owns the doc-coherence sweep
didn't carry the structural knowledge of the two docs it most often
edits. Encoding the move and the read-format-first discipline in the
spec makes the reconciliation non-skippable rather than
judgment-dependent. Connects to the "Skills own rituals" principle —
the skill that owns the ritual carries the knowledge the ritual needs.

**Why not a `/design-review` checkpoint first.** The outbound move
encodes a protocol `open-questions.md`'s own header already documents;
the change is a root-cause spec fix honoring an existing contract, not
a fresh design decision. A checkpoint would be heavier machinery for a
settled design, and a detour before Phase 2.1.

**Why not inline the file formats into `wind-down.md`.** Duplicates the
format definitions and drifts — the prior inlined hint ("Decision →
Why → Why not") had already drifted, omitting the Scope note. Pointing
at each file's own format section keeps one source of truth.

**Scope note.** Spec edit landed in both `.claude/commands/wind-down.md`
copies (root + cc-template, NFR-9). The read-format-first discipline is
scoped to the two decision/question docs, not every Step 3 tracking
doc. Originated as an `open-questions.md` story surfaced in Session E.

---

## CLAUDE.md banner carries no next-step or next-phase prose

**Decision.** The post-configuration banner in `CLAUDE.md` is static
orientation only — it never names a specific next prompt or phase.
`TODO.txt` is the single source of truth for "what's next." Commands
that write the banner (`/onboard`, `/bootstrap`) point readers at
`TODO.txt` instead of naming a phase, and `/wind-down` Step 3 checks the
banner for drift rather than re-deriving next-step prose. Resolves drift
mode 1 of the now-closed "Trim CLAUDE.md" story.

**Why.** The banner duplicated `TODO.txt` — which `CLAUDE.md` itself
names as the next-step source — so the two disagreed whenever `TODO.txt`
moved and the banner didn't (the banner named "Phase 1.1" for several
phases after it landed). `CLAUDE.md` loads into every session's initial
context, so stale next-step prose is permanent weight that actively
misleads.

**Why not keep a status banner a command updates each phase.**
Self-edited next-phase banners are the drift source; any
write-it-each-phase scheme re-introduces the disagreement. The
status-comment configuration banner (`ONBOARD-STATUS` etc.) stays — it
is status, not next-step, and earns its place as the entry point on a
fresh project.

**Scope note.** Companion to "Rules-read reliability" (which removed the
Collaboration-rules per-rule summaries — drift mode 3) and the
consumer-boundary partition (banner stays consumer-owned). All three
drift modes of the "Trim CLAUDE.md" story are resolved; its remaining
open sub-questions stay in `open-questions.md`.

---

## The refresh build edits `cc-template/` only; root is brought current by the source-mode dogfood

**Decision.** The build session edits `cc-template/` only — markers,
the command file, and the dist docs. Root does **not** get hand-edited
markers or a hand-mirrored command. Instead, root receives them by
**running `/refresh-from-repository` in source mode against this repo**
(the dogfood): the command mirrors itself to root and inserts the
`CC-TEMPLATE-BLOCK` markers into root's rules + CLAUDE.md via the
pre-marker migration. The only thing persisted is the markers in those
files (no sidecar — see "reconciliation: Option A").

**Why.** This repo is a consumer of its own distribution, so the honest
way to get markers and the command onto root is the same path every
downstream consumer uses — which also makes the first run the real
end-to-end test of source-mode + pre-marker migration on a live
project, not a contrived sandbox.

**Why not hand-author root's markers and hand-mirror the command.**
Faster to a committed result, but skips the dogfood validation and
risks the hand-written artifacts diverging from what the command
actually produces.

**Scope note.** Originated in the Phase 2.1 build under the Option D
mechanism (which seeded a committed state file); under Option A the
dogfood writes only markers, but the build-in-dist-then-dogfood-to-root
discipline is unchanged. The dogfood run and the root `CLAUDE.md`
invariant pin are Phase 2.1.A Block 2.

---

## The self-updater is bootstrapped by a manual skills-only copy

**Decision.** A project that predates `/refresh-from-repository` (or is
brand-new and lacks it) installs the command by hand once: copy the
command file(s) from the upstream `cc-template/.claude/commands/` into
the project's `.claude/commands/`, then run `/refresh-from-repository`
normally. This is documented in `cc-template/README.md` under "Keeping
a project up to date."

**Why.** A self-updating command can't install itself — there is a
genuine chicken-and-egg gap at the start. The manual copy is exactly a
`--refresh-skills-only` pass done by hand, so it reuses a concept the
consumer already has rather than inventing a separate installer. It
also serves the source-of-truth repo's own root: copy the command from
local `cc-template/` to root, then run the source-mode dogfood.

**Why not an installer script.** Would add a language runtime / script
to a markdown-only template (NFR-6) to solve a one-time, two-step
manual copy.

**Why not ship the command pre-placed at root in seeded projects.**
`/onboard` already copies the command set into a seeded project, so new
projects get it; the manual path exists for projects seeded before the
command existed. Naming the manual path costs a README section and
closes the gap for everyone.

**Scope note.** Surfaced during the Phase 2.1 build when working out
how root and other repos first obtain the command.

---

## `/refresh-from-repository` public-mode fetch is a shallow clone

**Decision.** In public mode, `/refresh-from-repository` reads upstream
by shallow-cloning the repo (`git clone --depth 1 <baked-in-url>`) into
a throwaway temp directory, reading the `cc-template/` subtree from it,
and deleting the temp dir at the end of the run. History is not fetched
(`--depth 1`) because Option A keeps no baseline. Adopted during the
Phase 2.1.A build, 2026-06-04.

**Why.** Under Option A both modes need only upstream's *current*
content. A shallow clone gives the merge step the same thing source
mode already has — a directory tree to read — so after the fetch the
two modes converge on one mechanism ("read files from a directory"),
which is exactly the convergence checkpoint 004 asked for. git is
already a hard requirement of the surrounding workflow (every refresh
ends in a `/wind-down` commit; this is a git repo), so requiring it for
the fetch costs nothing new.

**Why not raw HTTPS fetch** (`raw.githubusercontent.com/...` of a known
file list). It would make public mode git-free, lowering the consumer
bar — but it diverges the two modes (disk-dir vs N URL fetches) and
needs a hardcoded file list that goes stale if the shipped set changes.
The git-free benefit is small when the workflow is git-bound anyway.

**Why not a tarball** (`api.github.com/.../tarball/HEAD`). One request
and git-free, but adds tar-extraction tooling and unauthenticated API
rate limits for negligible gain over the shallow clone.

**Scope note.** This decision was not pinned by checkpoint 004 (which
said only "public mode no longer needs git *history*" without naming
the fetch mechanism). Recorded here so the build decision is on the
record outside the checkpoint trail. The temp clone doubles as the
quarantine for the security gate below.

---

## `/refresh-from-repository` reviews the download before applying

**Decision.** `/refresh-from-repository` fetches upstream into a
throwaway staging area and **reviews it before any live-tree write**.
In **public mode** the executing session does an adversarial security
read of the staged content (command files first — they are executable
instructions a later session runs) looking for injected directives,
exfiltration, safeguard-disabling, and surprising shell, then presents a
change summary and asks for an explicit go/no-go; declining deletes the
staging area and changes nothing. In **source mode** (the consumer's own
local `cc-template/`, already trusted) the adversarial read is skipped
and a plain change summary is shown. There is no flag to skip the
review. Adopted during the Phase 2.1.A build, 2026-06-04.

**Why.** Refresh imports executable instructions and behavior-shaping
context from a network upstream. A compromised upstream — a stolen
maintainer credential is the concrete scenario — could inject malicious
instructions that a later session would then follow. Replacing files
from the network is a trust boundary, and `design-philosophy-rules.md`
already carves out destructive/expensive operations as requiring an
explicit affirmative. The shallow-clone temp dir is already a natural
quarantine, so gating between download and apply costs almost nothing.

**Why the public/source split.** Source mode's upstream is the
consumer's own repo content — already under their control — so an
adversarial scan there is friction without a threat. Public mode is the
untrusted-network case the gate exists for. Simple by default, with the
heavyweight check where the risk actually is (progressive disclosure).

**Why not a `--skip-review` / `--trust` flag.** The safe path should be
the default, and the review is cheap (a handful of small markdown
files). A skip flag is exactly the affordance an attacker's social-
engineering ("just run it with --trust") would target.

**Why not route this through a `/design-review` checkpoint first.** It
is a safety guardrail Jamie directed in-session, not a speculative
feature; a settled safety requirement doesn't need a checkpoint to
authorize it. The *broader* idea — a standing security-review lens for
`/design-review` that flags common attack vectors — is queued as a
deferred user story in `docs/open-questions.md`, separate from this
build.

**Scope note.** Like the shallow-clone decision above, this was not
pinned by checkpoint 004; it surfaced during the 2026-06-04 build when
Jamie flagged the supply-chain risk. Recorded here so it is on the
record outside the checkpoint trail.

---

## `CC-TEMPLATE-BLOCK` marker encoding lives with the skill + design docs, not `CLAUDE.md`

**Decision.** The byte-level marker-state encoding (`template-owned`
= bare marker; `forked` = `state=forked` on the open marker;
`removed` = closerless tombstone) is documented in the owning command
`.claude/commands/refresh-from-repository.md` (operational authority),
`docs/REQUIREMENTS.md` NFR-4, and `docs/ARCHITECTURE.md`. It is
deliberately **not** added to root `CLAUDE.md`'s "Load-bearing
invariants" section.

**Why.** `/refresh-from-repository` is the sole consumer of the
encoding, and the command file is self-contained and shipped — the
natural source of truth; the design docs carry the maintainer's
record. CLAUDE.md's invariants section is a "don't break this"
tripwire list; duplicating the full encoding there creates a second
copy that drifts from the skill, for a string only one skill parses.

**Why not mirror the `ONBOARD-FILL` precedent.** `ONBOARD-FILL`
marker names ARE pinned in CLAUDE.md invariants, which invites
"fix the inconsistency" by adding CC-TEMPLATE-BLOCK too. The
difference: `ONBOARD-FILL` names are short identifiers referenced
across multiple skills (`/onboard`, `/bootstrap`,
`/refresh-from-repository`); CC-TEMPLATE-BLOCK's `state=` syntax is
single-skill operational detail.

**Why not a one-line CLAUDE.md tripwire pointer.** Considered and
declined — the markers are self-evident inline in the files, and a
pointer carrying no encoding adds maintenance surface for little gain.

**Scope note.** Decided 2026-06-04 during the Phase 2.1.A Block 2
pins, superseding that block's planned "pin marker syntax into root
CLAUDE.md" scope item. Revisit the tripwire-pointer option if a
future hand-edit actually mangles a marker in practice.

---

## `/product-visioning` authors PRDs; `/onboard` decomposes them; `/design-review` only reviews

**Decision.** The strategic-planning layer is three commands, one job
each. `/product-visioning` runs an interactive session whose sole output
is the next `docs/design/PRD-<slug>-NNN.md` (`DRAFT`). `/onboard`
decomposes a PRD — the first PRD is full project setup (adopting the
design intake as `PRD-<slug>-001`); a later movement applies the PRD's
vision revisions to `PRODUCT_VISION.md`, archives the prior plan to
`docs/project-plans/`, authors the new `PROJECT_PLAN` + prompts, and
flips PRD status (`DRAFT`→`ACTIVE`, prior→`SUPERSEDED`). `/design-review`
only reviews the result. PRDs and plan archives share one movement
counter; `PRODUCT_VISION.md` is onboard-owned with a surviving "Product
personality & positioning" section. Per-file PRD frontmatter
(`DRAFT`/`ACTIVE`/`SUPERSEDED`) is the source of truth — no status banner.

**Why.** One job per command keeps ownership clean: the review command
stays read-only and findings-only (rule 4), and decomposition stays with
`/onboard`, which already does it for the first PRD — so a later movement
is the same job, one decomposer, no second to drift.

**Why not let `/product-visioning` decompose too.** Re-merges
intent-capture with doc-generation; `/onboard` already decomposes the
first PRD, so reusing it for later movements is simpler than a second
decomposer.

**Why not a `PRODUCT-VISION-STATUS` comment/banner.** Per-file PRD
frontmatter is the source of truth (mirrors the recurring-artifact
frontmatter-not-banner separation); a fourth status comment would
conflate the strategic loop with the three-status configuration ritual.

**Scope note.** Built out-of-band (a downstream project needed the
movement workflow first), edited in `cc-template/` only; root command
files + the `CLAUDE.md` `reading-order` convention-check nudge sync via
the source-mode dogfood next session. `design-review.md` is pure-review,
owned by Jamie via git. Recorded as Phase 2.2. Full model + invariants
live in `CLAUDE.md` § "/product-visioning + PRD / product-vision
artifacts" and the file-ownership list.
