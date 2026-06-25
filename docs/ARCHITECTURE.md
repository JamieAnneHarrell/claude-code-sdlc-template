# Architecture

How `claude-code-sdlc-template` is structured. Lifted from
[`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md)
during `/onboard`. The "code" here is markdown; what matters is the
file layout, the placeholder/marker conventions, and the
duplication discipline between source and distributable.

## High-level shape

Two physical surfaces in one git repository:

- **Source-of-truth** at repo root (`claude-code-sdlc-template/`).
  Holds this project's own REQUIREMENTS / ARCHITECTURE /
  PROJECT_PLAN / CLAUDE_CODE_PROMPTS, design intake, design
  decisions, open questions, root `CLAUDE.md`, and root `README.md`
  — none of which copy into the distributable. Also holds a full
  set of rules and `.claude/commands/` so the source can run the
  commands on its own docs.
- **Distributable** at `cc-template/` (subdirectory of the repo).
  This is what consumers copy when seeding a new project. Ships
  the commands, the curated rules with `ONBOARD-FILL`
  placeholders, and stub docs. Renamed by the consumer after the
  copy so their new project directory IS the renamed
  `cc-template/` contents.

The source-of-truth project is a **downstream consumer of its own
template**. Improvements get exercised on the source before they
ship.

```
claude-code-sdlc-template/                ← repo root (source-of-truth)
├── CLAUDE.md                             ← source-side session-start
├── README.md                             ← source-side README
├── TODO.txt                              ← session handoff (gitignored)
├── LICENSE                               ← CC BY-NC-ND 4.0 (source-only docs)
├── .gitignore
├── rules/                                ← rules (mirrored to cc-template/rules/)
├── docs/                                 ← source-only project docs
│   ├── REQUIREMENTS.md                   ← (this project's requirements)
│   ├── ARCHITECTURE.md                   ← (this project's architecture)
│   ├── PRODUCT_VISION.md                 ← (this project's strategic north star)
│   ├── PROJECT_PLAN.md                   ← (this project's phase queue)
│   ├── CLAUDE_CODE_PROMPTS.md            ← (this project's per-phase prompts)
│   ├── design-decisions.md
│   ├── open-questions.md
│   ├── design/                           ← design intake + /design-review checkpoints + PRD-<slug>-NNN.md movement PRDs
│   ├── project-plans/                    ← archived movement plans + prompts (created when a 2nd movement opens)
│   ├── test-plans/                       ← /exit-test-plan artifacts (created on demand)
│   └── published/                        ← /write-documentation sources + manifest (created on demand)
├── .claude/commands/                     ← live commands the source uses on itself
└── cc-template/                          ← THE DISTRIBUTABLE
    ├── CLAUDE.md                         ← new-project session-start (with status comments)
    ├── README.md                         ← shipping README stub
    ├── TODO.txt                          ← starter TODO
    ├── LICENSE                           ← MIT (the dist's permissive license)
    ├── .gitignore
    ├── rules/                            ← rules with ONBOARD-FILL placeholders
    └── .claude/                          ← shipping commands + settings example
```

## Module layout (what each surface owns)

**Universal content** — identical between root and `cc-template/`:

- The 10 coding-session rules
  ([`rules/coding-session-rules.md`](../rules/coding-session-rules.md))
- Design philosophy rules
  ([`rules/design-philosophy-rules.md`](../rules/design-philosophy-rules.md))
- Environment, multi-agent, project, and testing rules placeholders
  with `ONBOARD-FILL` markers
- The command files
  ([`.claude/commands/onboard.md`](../.claude/commands/onboard.md),
  `bootstrap.md`, `deployment-plan.md`, `design-review.md`,
  `exit-test-plan.md`, `product-visioning.md`, `wind-down.md`,
  `refresh-from-repository.md`, `write-documentation.md`)

**Source-only content** — lives only at repo root:

- REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN / CLAUDE_CODE_PROMPTS /
  PRODUCT_VISION
- `docs/design/` (design intake docs, this project's `/design-review`
  checkpoints, and `PRD-<slug>-NNN.md` movement PRDs when they exist)
- `docs/project-plans/` (archived movement plans + prompts when they
  exist)
- `docs/test-plans/` (this project's `/exit-test-plan` artifacts
  when they exist)
- `docs/published/` (this project's `/write-documentation` sources,
  images, and documentation-plan manifest when they exist)
- `docs/design-decisions.md`, `docs/open-questions.md`
- `README.md` at repo root (the "this is the source-of-truth"
  README)
- `CLAUDE.md` at repo root (the source-of-truth session-start
  guide)
- This project's own `TODO.txt`

**Dist-only content** — lives only inside `cc-template/`:

- A different `CLAUDE.md` with the three configuration-status
  comments in `UNCONFIGURED` state
- A different `README.md` written as the consumer-facing template
  intro
- A starter `TODO.txt` whose first entry is "Run /onboard"
- A `LICENSE` (MIT) covering the dist
- Stub `docs/` and `docs/design/README.md` telling the consumer
  where to drop their design intake

## Data model — load-bearing invariants

There's no traditional data model. What functions as the "schema"
is the set of conventions that command files parse literally and
that stage detection depends on. Breaking one of these without
auditing the whole chain causes silent regressions.

### Status comments in `cc-template/CLAUDE.md`

Three configuration-ritual status comments, each owned by exactly
one command:

| Comment                          | Owner             | Base values                                  | Extra values                          |
|----------------------------------|-------------------|----------------------------------------------|---------------------------------------|
| `<!-- ONBOARD-STATUS: ... -->`         | `/onboard`        | `UNCONFIGURED`, `COMPLETE <YYYY-MM-DD>`      | —                                     |
| `<!-- BOOTSTRAP-STATUS: ... -->`       | `/bootstrap`      | `UNCONFIGURED`, `COMPLETE <YYYY-MM-DD>`      | `INSTRUCTIONS-WRITTEN <YYYY-MM-DD>`   |
| `<!-- DEPLOYMENT-PLAN-STATUS: ... -->` | `/deployment-plan`| `UNCONFIGURED`, `COMPLETE <YYYY-MM-DD>`      | `DEFERRED <YYYY-MM-DD>`               |

The recurring commands (`/design-review`, `/exit-test-plan`) do
NOT have status comments in `CLAUDE.md`. Per-file frontmatter
status (`AWAITING-DECISIONS` / `LANDED`,
`AWAITING-DISPOSITIONS` / `LANDED`) on individual checkpoint and
test-plan files is the source of truth for those lifecycles.

### ONBOARD-FILL marker blocks in rules files

| Marker                                              | File                              | Filled by    |
|-----------------------------------------------------|-----------------------------------|--------------|
| `<!-- ONBOARD-FILL: project-scope -->` ... `<!-- /ONBOARD-FILL -->` | `rules/project-rules.md`          | `/onboard`   |
| `<!-- ONBOARD-FILL: environment -->` ... `<!-- /ONBOARD-FILL -->`  | `rules/environment-rules.md`      | `/bootstrap` |

The markers demarcate downstream-owned content (everything inside)
from template-owned content (everything outside) so
`/refresh-from-repository` (Phase 2) preserves the inside while
updating the outside. They are the inverse primitive to the
`CC-TEMPLATE-BLOCK` markers below: `ONBOARD-FILL` wraps
consumer-owned regions refresh never touches; `CC-TEMPLATE-BLOCK`
wraps template-owned regions refresh keeps in sync.

### CC-TEMPLATE-BLOCK marker state (refresh-managed content)

`<!-- CC-TEMPLATE-BLOCK: <id> --> ... <!-- /CC-TEMPLATE-BLOCK -->`
wraps the template-owned regions of `rules/*.md` and `CLAUDE.md`
that `/refresh-from-repository` keeps in sync with upstream. Each
block has a stable kebab-case `<id>` (file-scoped) used to match
the same block across upstream and downstream.

Under the Option A reconciliation model (FR-13, checkpoint 004),
**the template files are their own memory** — each block also
carries a state recorded in the marker itself. There is **no
sidecar state file, no per-block content hash, and no baseline
reference** (this is what distinguishes Option A from the abandoned
Option D mechanism; see `design-decisions.md`). The state vocabulary:

| State            | Meaning                                                        | Refresh behavior                              |
|------------------|----------------------------------------------------------------|-----------------------------------------------|
| `template-owned` | Default — the block tracks upstream.                           | Update it to match upstream.                  |
| `forked`         | The consumer has taken ownership (set once, by asking).        | Leave it; may note when upstream diverges.    |
| `removed`        | A **tombstone** — the block was deleted or moved out.          | Respect it; never re-add the block.           |

Reconciliation is a **two-way compare** (downstream-current vs
upstream-current) matched by `<id>`, with a one-time question
recorded in-file when a block diverges or is absent with no
tombstone (keep mine → `forked` / take upstream / hand-merge;
never-had-it → add / removed-it → write tombstone). The executing
Claude session performs the merge. The **state vocabulary** and the
no-state-file rule are load-bearing per NFR-4. The marker encoding
(pinned by the Phase 2.1.A build): `template-owned` is the bare
`<!-- CC-TEMPLATE-BLOCK: <id> -->` (refresh never writes
`state=template-owned`); `forked` is `state=forked` on the open
marker with body and closer unchanged; `removed` is a closerless
tombstone — a single `<!-- CC-TEMPLATE-BLOCK: <id> state=removed -->`
comment with no body and no closer. Logic drift between a downstream's loaded
refresh and upstream's is handled separately by the command file's
`Refresh-logic-version` stamp, not by these markers.

### Filename conventions for recurring artifacts

- `docs/design/design-review-checkpoint-NNN.md` where N is
  zero-padded 3 digits (`001`, ..., `999`).
- `docs/test-plans/phase-NNN-exit.md` same pad width.
- `docs/design/PRD-<slug>-NNN.md` — the stand-alone per-movement PRD
  (`<slug>` = the project slug from `/onboard`); lives in the
  `docs/design/` intake set, same pad width.
- `docs/project-plans/project-plan-NNN.md` +
  `docs/project-plans/claude-code-prompts-NNN.md` — the archived
  (landed) movement plan + prompts, same pad width.
- `docs/published/documentation-plan-NNN.md` — the numbered
  `/write-documentation` plan (manifest); newest governs. Same pad
  width; its own counter (not the movement counter).

Pad width is fixed at 3 so files sort lexicographically as numeric
order through 999. PRD and plan-archive share one **movement
counter**: movement N is `PRD-<slug>-N`; opening movement N archives
movement N−1's in-flight plan as `project-plan-(N-1)`. The
documentation-plan counter is independent (it advances on a doc-set
re-scope, not on a movement). Multiple commands (`/design-review`,
`/exit-test-plan`, `/product-visioning`, `/wind-down`,
`/write-documentation`) depend on these glob shapes.

### Stage-detection placeholders

Stage detection in `/design-review` and `/exit-test-plan` parses literal
strings to distinguish marked from unmarked findings and filled from
pending run logs:

- `> _[UNMARKED — replace this line with your decision per the legend above]_`
  — AUDIT NOTE blocks in design-review checkpoints
- `[PENDING]` — run log rows in test plans
- `> _[UNMARKED — approve / adjust: <note> / drop, per the legend above]_`
  — DOC DECISION blocks in `/write-documentation` documentation plans

Changing a placeholder text without auditing the consuming command file's
stage-detection steps (each command's Step 0 plus its Stage 1 steps) breaks
detection.

### Recurring-artifact section structure

`docs/design/design-review-checkpoint-NNN.md`:
- Findings sections (Round 1, then each Addendum N) with
  severity-tiered findings (Blockers / Recommendations / Notes),
  one `AUDIT NOTE — JAH:` block per finding, one finding = one
  decision
- `## Disposition log` table with a `Round` column distinguishing
  `Original` from `Addendum N` — rows append-only across rounds

`docs/test-plans/phase-NNN-exit.md`:
- §1 scope, §2 prep, §3 test cases (Steps / Expected / Fail
  signals), §4 run log with `[PENDING]` placeholders, §5
  dispositions, §6.N polish addendums (with their own first-class
  TCs + run log + comments)

`docs/design/PRD-<slug>-NNN.md`:
- Frontmatter `prd` / `movement` / `project` / `created` /
  `status: DRAFT | ACTIVE | SUPERSEDED <date>` / `opened-by`
- A "Starting point / current state" section (the PRD is stand-alone —
  readable without the prior PRD series), the movement's goals + scope
  (in / out / deferred), the proposed PRODUCT_VISION revisions `/onboard`
  applies on decompose, and the decisions made this session. May contradict
  earlier movements' PRDs; the latest `ACTIVE` is definitive.

`docs/PRODUCT_VISION.md`:
- A durable, slowly-changing strategic doc: thesis, a surviving
  *Product personality & positioning* section, the roadmap of movements,
  and the foreclosure-watch enablers. Written by `/onboard`, which applies
  each PRD's proposed vision revisions when it decomposes.

`docs/published/documentation-plan-NNN.md` (`/write-documentation`):
- Frontmatter `documentation-plan` / `project` / `created` /
  `status: AWAITING-APPROVAL | ACTIVE | SUPERSEDED <date>` (newest governs) /
  `documented-through` (a `{ movement, phase }` tuple).
- A proposed-doc-set section with one `DOC DECISION — JAH:` block per doc, an
  audience×feature matrix, a product-type-aware **delivery recipe** (consumed by
  `/deployment-plan`, which owns the render/build), a release-readiness ledger
  (stale / unfilled visual / conformance gap), and an append-only authoring
  log. Markdown sources + `images/` live alongside under `docs/published/`.
- Currency is **movement-aware**: a re-run is STALE when the active movement
  differs from `documented-through.movement`, or a later phase shipped in the
  same movement (phase numbers compared only within one movement).

### `PROJECT_PLAN` movement header & phase-status token

- `docs/PROJECT_PLAN.md` carries a header naming the `movement:` it
  represents and its `source-prd:` (the `PRD-<slug>-NNN` it was
  decomposed from). `/onboard` compares the latest `ACTIVE` PRD's movement
  ordinal against this header to decide whether a newer PRD is a movement
  to decompose (archive the prior plan + author fresh) vs. already done.
- Phase headers in `docs/PROJECT_PLAN.md` mark completion with the
  literal token `(COMPLETE)` / `(COMPLETE <date>)`. `/wind-down` reads
  it to detect a fully-landed movement (every non-roadmap phase
  `(COMPLETE)`, ignoring `(roadmap)` / `(SUPERSEDED …)`). Don't rename
  the token without auditing `/wind-down` and `/design-review`.

## Runtime / data flow

**Consumer-facing flow** (new project from the dist):

1. Consumer copies `cc-template/` into a new project directory and
   renames it.
2. Consumer drops one or more design docs into `docs/design/`.
3. Consumer runs `/onboard` — decomposes the first PRD into REQUIREMENTS /
   ARCHITECTURE / PRODUCT_VISION / PROJECT_PLAN / CLAUDE_CODE_PROMPTS +
   `PRD-<slug>-001`, fills rules `ONBOARD-FILL` blocks, writes README
   stub, flips `ONBOARD-STATUS`.
4. Consumer runs `/bootstrap` — plans dev environment in two
   modes (write, then verify), fills README "Developer setup",
   flips `BOOTSTRAP-STATUS`.
5. Consumer optionally runs `/deployment-plan` when test/prod
   planning is needed.
6. Throughout project life: `/product-visioning` at milestones to plan the
   next movement (writes a `PRD-<slug>-NNN`), which `/onboard` then
   decomposes into the next PROJECT_PLAN + prompts (archiving the prior
   movement's pair to `docs/project-plans/` and updating PRODUCT_VISION);
   `/design-review` reviews the decomposition and gates high-risk
   transitions; `/exit-test-plan` at phase exits; `/wind-down` at session
   close.

**Self-consumption flow** (this project):

Same as above, with the source project itself producing planning
docs from its own design intake at
[`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md).

**Refresh flow** (Phase 2): downstream projects pull updated
commands and rules from upstream via `/refresh-from-repository` —
single-stage by default. It replaces the command files wholesale,
then reconciles `rules/*.md` + `CLAUDE.md` against the downstream's
current state using the CC-TEMPLATE-BLOCK marker-state model above
(two-way compare + ask-once; the executing session merges,
preserving `ONBOARD-FILL` regions). When a downstream's loaded
refresh logic is behind upstream, it self-modifies: pulls the
commands skills-only first, then asks the consumer to re-invoke so
the new logic runs the merge. Built in Phase 2.1; the Option A
rewrite that replaces the abandoned Option D mechanism is Phase
2.1.A.

## Key technical decisions

The full rationale lives in
[`docs/design-decisions.md`](design-decisions.md). Summary:

1. **Source/dist subdirectory split.** Source-of-truth at repo
   root, distributable at `cc-template/`. Single repo. Rejected:
   gitignored-and-rebuilt dist; sibling directory; separate repo;
   `.gitattributes` `export-ignore` filtering.
2. **Pull model for downstream updates.** Downstream projects
   choose when to refresh via `/refresh-from-repository` (Phase 2).
   Rejected: push model where the maintainer runs a command to
   merge into specific downstream paths.
3. **All commands at source root, not just recurring.** The
   source-of-truth project is itself a downstream consumer of its
   own template; commands stay identical between root and dist to
   avoid drift.
4. **Public GitHub URL.**
   `github.com/JamieAnneHarrell/claude-code-sdlc-template`.
   Personal-namespace over a dedicated org for now;
   trivially-redirectable later if needed.
5. **Dual license.** Distributable ships MIT (consumer commercial
   use unrestricted); source-only docs ship CC BY-NC-ND 4.0
   (project-management docs not meant for redistribution).
6. **Multi-agent mode: explore-plus-plan.** Plan subagent earns
   its place because of the load-bearing invariant complexity in
   `CLAUDE.md` and the recurring-command lifecycles. Implementation
   stays sequential.

## What this architecture explicitly does NOT include

- **No language runtime.** Markdown only. No interpreter,
  compiler, or build step. (`/write-documentation` writes markdown + a delivery
  recipe and owns no renderer; `/deployment-plan` builds the delivered docs
  against the environment's runtime at release time — the template ships nothing
  and prescribes no runtime.)
- **No GUI, no web service, no API.** The deliverable is the file
  tree.
- **No traditional test suite.** Validation is the live-test
  walkthrough described in root `CLAUDE.md` plus the
  recurring-command iterative lifecycles applied to the project's
  own docs.
- **No automated markdown linting.** Style consistency is
  human-reviewed.
- **No package distribution** (npm, pip, etc.). Distribution is
  git-based.
- **No real-time sync between source and consumers.** Consumers
  update on their own schedule via Phase 2's
  `/refresh-from-repository`.
- **No CI for the source repo today.** May change with Phase 3
  regression-test automation; Phase 3 specifics deferred.
