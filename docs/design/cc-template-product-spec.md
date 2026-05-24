# cc-template source-of-truth project — design

> Written after-the-fact (2026-05-23) to capture where this project
> landed after Phase 0 (source/dist restructure). This document is
> the `/onboard` input that produces REQUIREMENTS / ARCHITECTURE /
> PROJECT_PLAN / CLAUDE_CODE_PROMPTS for the source-of-truth project
> itself.

## Concept

`cc-template` is a project seed for new software projects, distributed
as a copyable directory. The seed ships six Claude Code slash commands
that walk a developer through configuration (`/onboard` → `/bootstrap`
→ `/deployment-plan`) and recurring lifecycle checkpoints
(`/design-review`, `/exit-test-plan`, `/wind-down`), plus a curated
set of collaboration rules.

The project is **self-modifying**: the source-of-truth repo is itself
a downstream consumer of its own template. Template improvements get
exercised on the source before they ship to consumers. The only
asymmetry: `/refresh-from-repository` (Phase 2) won't target the
source from the source.

## Architectural premise

- **Two physical surfaces in one repo.** The source-of-truth project
  lives at repo root. The **distributable** lives at `cc-template/`
  — that's what consumers copy. The naming preserves the
  consumer-facing affordance ("copy a directory called
  `cc-template`") while putting the source under git.
- **Content duplication is deliberate.** Universal content (the 9
  rules, design philosophy, the six command files) is kept identical
  between root and `cc-template/`. Source-only content (this design
  doc, REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN / CLAUDE_CODE_PROMPTS,
  this project's internal docs) lives only at root.
- **No runtime.** Every artifact is markdown. The "product" is the
  set of commands and rules that drive Claude Code sessions inside
  a downstream project.
- **Configuration is three commands, lifecycle is two commands.**
  The three configuration commands (`/onboard`, `/bootstrap`,
  `/deployment-plan`) run in sequence at project start. The two
  recurring commands (`/design-review`, `/exit-test-plan`) run on
  demand throughout the project's life — each lifted into the
  template only after the pattern proved itself on a live project.
  `/wind-down` is the sixth, governing session close.

## Stack

- **No language runtime.** Markdown content only.
- **Windows 11 primary**, Linux/macOS first-class supported.
- **PowerShell** is the primary shell (per Jamie's environment).
- **Git** for version control (initialization pending Phase 1).
- **Claude Code** as the consuming runtime — the commands are
  slash commands invoked inside Claude Code sessions.

## Data model

Not applicable — no data, just files. What matters instead is the
set of **load-bearing conventions** that must remain stable:

- Three status comments in CLAUDE.md (`ONBOARD-STATUS`,
  `BOOTSTRAP-STATUS`, `DEPLOYMENT-PLAN-STATUS`) with a small
  controlled vocabulary of values.
- Two ONBOARD-FILL marker blocks in rules files
  (`<!-- ONBOARD-FILL: project-scope -->` in
  `rules/project-rules.md`, `<!-- ONBOARD-FILL: environment -->`
  in `rules/environment-rules.md`).
- Zero-padded 3-digit filename conventions for the two recurring
  artifacts: `docs/design/design-review-checkpoint-NNN.md` and
  `docs/test-plans/phase-NNN-exit.md`.
- Placeholder strings that stage detection parses literally
  (`> _[UNMARKED — replace this line with your decision per the
  legend above]_`, `[PENDING]`).
- Per-file frontmatter status on recurring artifacts
  (`AWAITING-DECISIONS` / `LANDED` on checkpoints;
  `AWAITING-DISPOSITIONS` / `LANDED` on test plans).

Root `CLAUDE.md` enumerates these as "load-bearing invariants" with
explicit don't-break-without-auditing-the-chain warnings.

## Runtime / data flow

Consumer-facing flow:

1. Consumer copies `cc-template/` subdir into a new project 
   directory, and renames this folder as their project name. This template is now the top level directory of the new project. There is no cc-template folder within the new project, the cc-template rename means the original cc-template folder IS new project folder, which will later become git repository top level.
2. Consumer drops one or more design docs into `docs/design/`.
3. Consumer runs `/onboard` — writes REQUIREMENTS / ARCHITECTURE /
   PROJECT_PLAN / CLAUDE_CODE_PROMPTS, fills rules ONBOARD-FILL
   blocks, writes README stub, flips `ONBOARD-STATUS`.
4. Consumer runs `/bootstrap` — plans dev environment in two modes
   (write, then verify), fills README "Developer setup", flips
   `BOOTSTRAP-STATUS`.
5. Consumer optionally runs `/deployment-plan` when test/prod
   planning is needed.
6. Throughout project life: `/design-review` at high-risk
   transitions, `/exit-test-plan` at phase exits, `/wind-down` at
   session close.

Self-consumption flow (this project): same as above, with the source
project itself producing planning docs from its own design intake
(this document).

Future flow (Phase 2): downstream projects pull updated commands and
rules from upstream via `/refresh-from-repository`. Two-stage,
self-modifying: stage 1 replaces command files; stage 2 runs the
new logic to merge rules and CLAUDE.md.

## Phased plan

**Phase 0 — Source/dist restructure (complete).** Repo restructured
into source-of-truth root + `cc-template/` distributable. All six
commands present at both locations. Rules and docs duplicated where
universal. Root CLAUDE.md and README written as template-improvement
guides.

**Phase 1 — Onboard the source-of-truth (in progress).** First,
rename the source root to `jah-template-project/` and `git init` +
baseline commit, so all subsequent work is recoverable. Then run
the configuration ritual on this project: `/onboard` against this
design intake produces REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN /
CLAUDE_CODE_PROMPTS at root, fills ONBOARD-FILL blocks in root
rules files, rewrites root `rules/multi-agent-rules.md` to
explore-plus-plan mode, writes the README stub, and flips
`ONBOARD-STATUS`. `/onboard` overwrites root CLAUDE.md wholesale,
so the template-improvement content (load-bearing invariants,
testing-template-changes, etc.) gets merged back in from
`CLAUDE.md.bak` afterward. Then `/bootstrap` to plan the dev
environment.

**Phase 1.1 — `/exit-test-plan revision` (deficiency)** Add a test plan disposition "BLOCKED" for items in the test plan that are not runnable due to a blocker discovered during testing. Blockers may require a cleanup coding pass, or even a design review then additional phases added before testing can continue. Blocked test items will be left in the record as blocked at the time, with testing of those components added again to the next test plan Addendum.

**Phase 2 — Planned project enhancements.** Known items to add and improve:

**Phase 2.1 — `/refresh-from-repository` (roadmap).** Build the
two-stage downstream update command. Ships in the dist. Open
sub-questions: public GitHub URL for the source repo; merge
strategy for `CLAUDE.md` (which has no ONBOARD-FILL markers).

**Phase 2.2 — `/write-user-documentation` (roadmap).** Build a
command to write end user documentation. Ships in the dist. Will
be used in this project to document how to use this project; will
be used in downstream projects by this project's consumers to
create their product documentation.

**Phase 3 — Regression-test automation (roadmap).** Scripted check
that diffs the current dist against a tagged baseline and flags
invariant-breaking changes (status comment renames, marker drift,
zero-pad-width changes in checkpoint/test-plan filenames). Specifics
deferred until a real regression motivates the work.

## Out of scope

- No GUI, no web service, no API. The deliverable is markdown files.
- No language runtime, no dependencies to install, no traditional
  test suite. Validation is the live-test walkthrough described in
  root `CLAUDE.md`.
- No automated markdown linting. Style consistency is human-reviewed.
- No package distribution (npm, pip, etc.). Distribution is
  git-based: clone or copy the dist subdir.
- No real-time sync between source and consumers. Consumers update
  on their own schedule via `/refresh-from-repository` (when built).

## Runtime / deployment intent

- **Local development.** Edit on the source-of-truth project. No
  build step.
- **"Production"** is the public upstream repo. Path TBD (see open
  questions). Consumers copy from the upstream's `cc-template/`
  subdirectory.
- **Updates downstream.** Pull-model only. `/refresh-from-repository`
  in the dist runs in the downstream project and pulls from the
  public upstream. No push from upstream to downstream.

## Multi-agent mode

**explore-plus-plan.** The project is non-trivial enough (recurring
command lifecycles, ten-plus load-bearing invariants in CLAUDE.md,
six commands with overlapping concerns) that validating a design
proposal with a Plan subagent before implementing is worth the
round-trip. Implementation stays sequential — no worktree spawning.

## Key technical decisions

Captured in `docs/design-decisions.md`:

1. **Source/dist subdirectory split** (vs gitignored dist, sibling
   directory, separate repo, or `.gitattributes` filter).
2. **Pull model for downstream updates** (vs push).
3. **All six commands at source root** (vs trimming to recurring
   only).

## Open questions

Tracked in `docs/open-questions.md`:

1. Public GitHub URL for the source repo (blocks Phase 2 design).
2. CLAUDE.md merge strategy for `/refresh-from-repository --merge`
   (rules files have ONBOARD-FILL markers; CLAUDE.md does not).
