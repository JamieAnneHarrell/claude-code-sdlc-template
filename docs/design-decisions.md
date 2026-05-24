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
One design problem still open for that phase: the merge strategy for
`CLAUDE.md` specifically (it has no `ONBOARD-FILL` markers today).
Tracked in `docs/open-questions.md`. The public GitHub URL is
resolved — see the "Public GitHub URL for the source repo" decision
below.

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
