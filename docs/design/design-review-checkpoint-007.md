---
checkpoint: 007
date: 2026-06-27
reviewer: Claude (Opus 4.8) with Jamie
status: LANDED 2026-10-05
trigger: Backlog-cleanup groom at MVP close-out — cluster open-questions into sequencing steps, flag conflicts/duplicates, surface decision-collapsing questions
---

# Design Review — Checkpoint 007

## Context

The MVP is closing out and the product runs well. This is a **backlog-cleanup
design review** — the first real instance of the convention proposed by the
backlog's own story *"Backlog-cleanup `/design-review` near the end of a large
phase or a movement."* It reads the whole working backlog in
[`open-questions.md`](../open-questions.md) — **28 Deferred User Stories, 1
Abandoned Approach (historical), and 4 Open Questions** — against
[`REQUIREMENTS.md`](../REQUIREMENTS.md), [`ARCHITECTURE.md`](../ARCHITECTURE.md),
[`PROJECT_PLAN.md`](../PROJECT_PLAN.md), [`design-decisions.md`](../design-decisions.md),
and the [user/lifecycle](../user/lifecycle.md) + [maintainer](../maintainer/maintaining-the-template.md)
orientation docs.

Per Jamie's instruction, the findings here do three things: **cluster** the
backlog into a small number of logical steps that would sequence cleanly if
built, **flag conflicts** (story-vs-story and story-vs-requirements), and
**recommend prunes/merges** for stale or duplicate items. Each grouping finding
carries a **relative priority rank** but **not** a lane assignment — Jamie
assigns the lane (next movement / in-movement enhancement / tactical / drop)
inline when marking.

**Settled going in (recorded so Stage 2 doesn't re-open them):**

- **Repo is already public.** Launch-readiness items are live gaps, not
  speculative work.
- **Identity de-personalization is rejected in its heavy form** (rule 3) —
  no `/onboard`-driven name/pronoun/initials parameterization. The surviving
  scope is an optional README re-personalization prompt that never touches the
  copyright line. This dissolves the previously-flagged identity-vs-copyright
  conflict (see N1).
- **Newer doctrine wins over older items.** Where a pre-`/product-visioning`
  story tensions with the now-landed product-visioning-first entry path or the
  documentation promotion, the newer method is authoritative.
- **EOL story (#19) is verified resolved/moot** — see R7b. The refresh command
  carries no line-ending detection and the repo carries no `.gitattributes`,
  matching the landed "rely on `core.autocrlf`" decision.

**Ownership constraint on the land path.** A backlog groom's natural output is
re-organizing `open-questions.md`, but that file is owned by `/onboard` +
`/wind-down`, **not** `/design-review` (Stage 2's edit scope is REQUIREMENTS /
ARCHITECTURE / PROJECT_PLAN / CLAUDE_CODE_PROMPTS + this checkpoint + REVIEWS.md).
So on **land**, accepted clusters are promoted into `PROJECT_PLAN.md` as
phase(s)/steps and the open-questions re-groom (re-cluster, promote, drop, merge)
is surfaced as a TODO for `/wind-down` to apply. This checkpoint cannot rewrite
the backlog file itself — it decides the shape; `/wind-down` enacts it. (This is
exactly the open sub-question story #25 raised; treated here as a working
constraint, not re-litigated.)

This gate unblocks the post-MVP sequencing decision: which clusters become the
next movement(s), in-movement enhancements, or tactical batches.

## How to mark up this review

For each finding below, replace the placeholder line under
`AUDIT NOTE — JAH:` with one of:

- `Accepted`
- `Accepted with caveats: <your caveats>`
- `Defer Approved`
- `DECISION: <explicit choice>`
- `REJECTED: <your reframing prose>` — none of the listed recommendations is
  satisfactory. This opens another addendum round; your rejection prose is the
  authoritative input the next addendum reframes the finding from.

For the grouping findings (R1–R6), a useful marking shape is
`Accepted with caveats: priority → P_, lane → <movement/in-movement/tactical>,
split/keep <items>` — accept the cluster, adjust its rank, and pencil the lane
if you already know it.

Each finding has its own AUDIT NOTE block — mark each independently. Then re-run
`/design-review`. Stage 2 walks each disposition with you, appends rows to the
Disposition log, and asks whether to **land** or **open another addendum round**.

**Do not fill the Sign-off Summary table at the bottom** — Stage 2 fills it when
the doc lands.

## Findings

### Blockers — fix before the sequencing gate

*No findings in this tier. Nothing blocks the sequencing decision itself; the
copyright-compliance gap (R1a) is a live issue but does not gate this review's
own decision, so it is the highest-priority Recommendation rather than a Blocker.*

### Recommendations — resolve before or during the sequencing gate

The six grouping findings (R1–R6) are the requested clusters, each with a
suggested **relative priority** (P1 highest). R7 collects prunes/merges; R8 is a
net-new capability surfaced during this groom (a live discovery this session).
The suggested order is P1 `R1` → P2 `R2` → P3 `R3` / `R8` → P4 `R5` → P5 `R4` →
P6 `R6`, with R7 housekeeping folded into whichever `/wind-down` groom lands
first. `R8` couples with `R5` — both extend the Movement / in-movement / tactical
lane taxonomy.

#### R1. Cluster: Public-launch readiness — copyright compliance + adopter doc delivery [Suggested priority: P1]

**Sources.** `open-questions.md` § Deferred User Stories *"Stamp shipped
`cc-template/` files with an inline copyright + license pointer"* and § Open
Questions *"How the source-only `docs/user/` + `docs/maintainer/` doc set reaches
adopters"*; `REQUIREMENTS.md` NFR-7 (dual license), FR-14/FR-12 (doc set is
source-only); `design-decisions.md` *"This project's audience-facing docs…"*
(delivery deferred to `/deployment-plan`); `PROJECT_PLAN.md` — `/deployment-plan`
still `UNCONFIGURED`.

**Problem.** The repo is **already public**, which turns two deferred items into
live gaps. (a) **Compliance**: MIT requires the copyright + permission notice be
retained in "all copies or substantial portions"; the `cc-template/` command and
rules files carry no inline notice, and `/onboard` rewrites the README + a
consumer drops their own LICENSE, so a seeded project can retain *no* record of
the upstream MIT grant. Shipping that today is out of compliance with our own
LICENSE. (b) **Adopter UX**: the canonical `docs/user/` + `docs/maintainer/` set
is source-only and never travels in `cc-template/`, so an adopter seeding today
receives no documentation and the shipping README can't relative-link into a set
that isn't there. These cohere as "what must be true for a public, adoptable
template," and both are real now rather than roadmap.

**Recommendation.** Treat (a) copyright/license stamping of shipped
`cc-template/` files and (b) the adopter doc-delivery decision as **one
high-priority readiness step**. (a) is small and self-contained (an SPDX-style
HTML-comment header on shipped command + rules files; KISS alternative of a
single `NOTICE` file is weaker because it doesn't survive a consumer deleting
root files — the exact failure mode). (b) gates on `/deployment-plan`, which owns
the render/delivery and inherits the recipe + open delivery questions in
`documentation-plan-001`. Downstream implication of sequencing this first:
because the repo is public, (a) is the single most time-sensitive item in the
whole backlog — recommend it leads regardless of how the rest is laned.

> AUDIT NOTE — JAH:
> Accepted with caveats: no SPDX. A plain HTML comment on each distributed
> file, along the lines of "This file is part of the claude-code-sdlc-template.
> See .claude/claude-code-sdlc-template-license.md for info." That license file
> holds the license text, the distribution URL, and a statement that the
> copyright applies to the original files as provided in the distribution.
> `/refresh-from-repository` is changed to deliver the license file into
> `.claude/`, using the two-stage refresh.

#### R2. Cluster: `/design-review` lifecycle hardening [Suggested priority: P2]

**Sources.** `open-questions.md` § Deferred User Stories — *"`/design-review`
Step S1.1 should read `docs/open-questions.md`"*, *"`/design-review` should carry
a security-review lens"*, *"Encode the design-decisions ↔ abandoned-approaches
hygiene rule in the commands"*, *"`/design-review` is iterative — mid-round
`/wind-down` is optional"*, *"`/design-review` pins every pinnable decision"*,
*"Backlog-cleanup `/design-review` near the end of a large phase or a movement"*;
`coding-session-rules.md` rules 3 & 4; `REQUIREMENTS.md` FR-6.

**Problem.** Six stories all refine the same command — the one this project
exercises most — and several encode session corrections so they become portable
to fresh downstream Claude sessions rather than living only in one maintainer's
memory. The pain is concrete and current: this very review had to read
`open-questions.md` manually because S1.1 doesn't yet list it (the read that
would have surfaced the Abandoned Approaches as hard constraints), and the
backlog-cleanup convention we're running has no spec home yet. Built piecemeal,
each is a separate edit to `design-review.md` (+ NFR-9 mirror); built together,
they are one coherent pass over one command file.

**Recommendation.** Bundle the six as a single `/design-review` hardening step.
Most are spec refinements (read open-questions at S1.1; the security-review lens;
the abandoned-approaches hygiene rule, also touching `/wind-down`; the
mid-iteration wind-down advisory; the pin-every-pinnable three-bucket
discipline) plus the backlog-cleanup-review convention this session pilots.
First-level decision is "accept the bundle as one step at P2." Downstream
implications (named, not specced): the security-lens (its own trigger
conditions) and the backlog-cleanup convention (trigger-type vs. lighter ritual;
whether it consumes an internal manifest — see N3) each carry a sub-decision that
can become its own finding in a later addendum if it turns load-bearing.

> AUDIT NOTE — JAH:
> Accepted with caveats: all six land, each as a one-to-three-line edit, with no
> new steps or sections. Read-the-backlog at S1.1, the security check, the
> mid-round wind-down advisory and pin-at-review-time go in `design-review.md`;
> the no-tombstone rule and the backlog-cleanup-review convention go in
> `wind-down.md`.

#### R3. Cluster: `/onboard` authoring quality [Suggested priority: P3]

**Sources.** `open-questions.md` § Deferred User Stories — *"`/onboard` should
author prescriptive prompts — self-contained exit criteria and contract-level
scope items"* and *"`/onboard` captures documentation intent so design-review can
review it"*; `REQUIREMENTS.md` FR-3, FR-14; `design-decisions.md` *"Documentation
is downstream of behavior."*

**Problem.** Both stories target the quality of what `/onboard` plants for the
coding sessions that follow. The prescriptive-prompts story has live downstream
evidence — a consumer found `/onboard`'s prompts too thin to code against (bare
"Per PROJECT_PLAN Phase N" exit criteria) and rewrote the whole file. The
documentation-intent story is the ownership question deferred when Block 2 added
only a one-line `/write-documentation` pointer: `/onboard` captures no doc intent,
so design-review has nothing to scrutinize and `/write-documentation` discovers
the doc set cold. They share a surface (`onboard.md`'s authoring of the planning
docs) and a theme (make the downstream session well-specified).

**Recommendation.** Group as one `/onboard`-authoring step at P3. The
prescriptive-prompts half is the higher-confidence, evidence-backed piece (tighten
the per-prompt structure: self-contained testable exit criteria, contract-level
scope, anchor-named read-first, trap-naming constraints — feature prompts only,
design-review prompts exempt). The documentation-intent half is genuinely
open-with-options and must reconcile with `/write-documentation`'s
infer-don't-prescribe design — recommend it stays a "to weigh" sub-decision
inside the step rather than a pre-committed build. Downstream implication: the
prescriptive-prompts change raises a KISS/rule-4 tension (restating PROJECT_PLAN
exit criteria inside prompts creates two places that can drift) — name the trade
in the spec; don't expand scope to auto-sync them.

> AUDIT NOTE — JAH:
> Accepted with caveats: split. Prescriptive prompts land as four requirement
> lines in `onboard.md`'s per-prompt structure, feature prompts only, replacing
> the line that points consumers at "this template's own
> `docs/CLAUDE_CODE_PROMPTS.md`" (a file that does not ship). The
> documentation-intent story stays deferred in the backlog.

#### R4. Cluster: `/wind-down` repo-wide coherence sweep + the doc-hygiene debt it clears [Suggested priority: P5]

**Sources.** `open-questions.md` § Deferred User Stories — *"`/wind-down`'s
coherence sweep is repo-wide, not session-scoped"*, *"'Broken-window' rule — fix
small not-by-design errors within a larger edit"*, *"Command 'does NOT do'
sections duplicate the ownership map"*, *"Brittle hardcoded command/file counts
across planning docs"*, *"Sweep local Claude-memory items that are really project
guidance into checked-in docs"*; `REQUIREMENTS.md` FR-8, NFR-9; `CLAUDE.md`
ownership map.

**Problem.** Two of these define what the coherence sweep *does* (make it
repo-wide rather than session-scoped; add the broken-window fix-small-errors
behavior), and three are concrete debt a repo-wide sweep would clear (the "does
NOT do" cross-reference duplication; the brittle hardcoded "six commands" /
"seven SDLC commands" counts in non-frozen planning docs; the mis-filed
local-memory items). They pair naturally: change the sweep's contract, then its
first run pays down the standing debt. There is also a genuine story-vs-spec
conflict the repo-wide story names — the current `wind-down.md` scopes the sweep
"to what the session actually changed," which contradicts the standing intent
that the sweep fixes pre-existing drift too.

**Recommendation.** Group as one `/wind-down`-hygiene step at P5: resolve the
repo-wide-vs-session-scoped contradiction (likely: repo-wide surface, fix
obvious mechanical drift inline, surface judgment calls), add the broken-window
behavior, and let the first repo-wide sweep clear the three debt items. First-level
decision is surface-only vs. auto-fix for *unrelated* drift — the story's own
wording leans surface-first with inline mechanical fixes. Downstream implication:
the local-memory-sweep item is partly machine-local housekeeping (some memories
are genuine cross-project preferences that stay) — it rides along but is the
lowest-stakes member and could be split out as tactical.

> AUDIT NOTE — JAH:
> Accepted with caveats: the repo-wide sweep and the broken-window rule land as
> a two-sentence change in `wind-down.md` — the sweep covers the whole repo,
> fixes mechanical drift and small errors inline, and surfaces judgment calls to
> Jamie. The three debt stories (duplicated "does NOT do" ownership text,
> hardcoded counts, misfiled memory notes) get no dedicated work and stay in the
> backlog; hardcoded counts are fixed as the repo-wide sweep meets them.

#### R5. Cluster: Planning doctrine — in-movement lane + Movement→Phase→Step vocabulary [Suggested priority: P4]

**Sources.** `open-questions.md` § Deferred User Stories — *"Document the
out-of-band 'in-movement enhancement' lane"* and *"Coalesce the planning
vocabulary on Movement → Phase → Step, and pin where prompts fit"*; `CLAUDE.md`
§ "Movement vs. tactical"; `ARCHITECTURE.md` § "Phase vs. step terminology"
(checkpoint 006 R4 left them interchangeable); `REQUIREMENTS.md` NFR-2 (filename
counters).

**Problem.** These two are **explicitly coupled** — both rewrite CLAUDE.md's
"Movement vs. tactical" framing, and the backlog itself says they "must land
coherently." But they differ sharply in size. The in-movement-lane story is
cheap and high-value: the lane already exists (we used it for the
write-documentation reshape, and re-derived it this session with a wrong turn
through `/product-visioning`) — it just needs documenting. The
Movement→Phase→Step story is large and load-bearing: it touches NFR-2 filename
semantics (`phase-NNN-exit` NNN becoming a sequential counter), the
"Phase 2.x"/"Step 2.x" interchangeability, and nearly every shipped file, and it
collides with existing names.

**Recommendation.** Group as one doctrine cluster at P4 **but flag the size
split as the first-level decision**: recommend the in-movement-lane half lands
as a small doctrine codification (plausibly soon, given the recurring
re-derivation cost), while the Movement→Phase→Step half is large enough to be
its **own movement** (or at least its own in-movement enhancement) rather than
riding along. The coupling is satisfied by landing the lane-naming in a way the
later vocabulary work extends, not contradicts — i.e. don't pick lane names now
that the vocabulary settlement would have to rename. Downstream implications of
the vocabulary work (numbering scheme rename-vs-renumber; whether a level above
Movement is needed; which filename families inherit the words) are dependent
sub-decisions to spec in that movement's own review, not here.

> AUDIT NOTE — JAH:
> Accepted with caveats: (1) Define the three kinds of work — movement,
> in-movement enhancement, tactical — once, in a block of about five lines in
> shipped `rules/project-rules.md`. (2) The vocabulary lands as a definition only, in the
> same block, in these words: a movement is as already defined. A phase is a
> logically grouped set of tasks achieving a goal; it may take one prompt or
> ten. Each of those prompts is a step of the phase — Step 5.1 is the first
> step of Phase 5, Step 5.2 the second. Each step has one prompt, and generally
> each prompt is an individual Claude Code session, so steps are session-sized
> chunks of work for a phase. Rename, don't renumber: existing "Phase 2.x"
> headings are really Steps 2.x of Phase 2 and stay as written until the full
> sweep; every new entry is written as "Step N.M". No filename changes; the
> full vocabulary sweep stays in the backlog. Adding steps and phases on the
> fly is settled under R8.

#### R6. Cluster: CLAUDE.md / always-loaded / session-start orientation knot [Suggested priority: P6]

**Sources.** `open-questions.md` § Deferred User Stories — *"CLAUDE.md as a thin
index — remaining sub-questions"*, *"'Project quick orientation' section in
onboarded CLAUDE.md"*, *"In-flight artifact status callout at top of CLAUDE.md"*,
*"Promote the 'good morning' session-start + per-prompt kickoff ritual to a
skill"*; § Open Questions — *"Portability audit — what makes this project work
better than a freshly-seeded one?"* and *"One-line CLAUDE.md guardrail: 'Claude
never touches CC-TEMPLATE-BLOCK markers'"*; `design-decisions.md` *"CLAUDE.md
banner carries no next-step prose"*, *"Rules-read reliability."*

**Problem.** This is the most internally-conflicted region of the backlog: the
thin-index story argues CLAUDE.md should shed weight, while the status-callout
and quick-orientation stories propose *adding* sections, and the good-morning
ritual and portability audit ask which behaviors belong in always-loaded
surfaces at all. The backlog already names the tension (thin-index "probably
retire the callout story in favor of the trim stance"; the good-morning story
"resolve together with the thin-index and portability-audit questions"). Decided
piecemeal, these will re-litigate each other; decided together, one stance on
"what CLAUDE.md / always-loaded surfaces are for" collapses ~5 coupled decisions.

**Recommendation.** Group as one "CLAUDE.md / session-start stance" step at P6
(lower priority — valuable but no acute external pain, mostly internal ergonomics).
The first-level decision is the **trim-vs-enrich stance**; everything else falls
out of it. Recommended dependent dispositions, named not specced: **drop the
in-flight status-callout story** (the backlog itself leans retire-in-favor-of-trim,
and it was already deferred at checkpoint 003 R8) — this is the cleanest prune in
the cluster; treat the quick-orientation section, the good-morning ritual
(skill-vs-prose), the marker guardrail line, and the portability-audit promotions
as the items the chosen stance then resolves. If you'd rather decide the
status-callout drop independently of the stance, mark R6 with a caveat splitting
it out.

> AUDIT NOTE — JAH:
> DECISION: `CLAUDE.md` is reasonable as is; no further thinning and no new
> sections. The six items settle as follows.
>
> (1) Thin index: story dropped. `CLAUDE.md` is reasonable as is.
>
> (2) In-flight status callout: story dropped. `TODO.txt` records status and
> has been working well.
>
> (3) Project quick orientation: story dropped as a `CLAUDE.md` section.
> `project-rules.md` is the home for that content. The session-start read grows
> from two rules files to four (`coding-session-rules.md`,
> `design-philosophy-rules.md`, `environment-rules.md`, `project-rules.md`),
> using Jamie's block text for the collaboration-rules block and the matching
> top-of-file directive. Edited in `cc-template/` only; root receives it through
> a local source-mode `/refresh-from-repository` run. Root `CLAUDE.md` is not
> edited directly.
>
> (4) `CC-TEMPLATE-BLOCK` marker guardrail: closed and dropped. Not found
> necessary in practice; the 2026-06-04 decision and its revisit trigger already
> stand in `design-decisions.md`.
>
> (5) Good-morning ritual: story dropped; no skill. The session-start pattern is
> fine as is. In its place the template ships Jamie's `claude-code-sdlc-template`
> output style and the `outputStyle` setting that selects it. Denying Bash is
> offered as a README suggestion for Windows PowerShell users; no skill sets it.
>
> (6) Portability audit: Defer Approved.

#### R7a. Merge the duplicate regression-test-automation stories [Suggested priority: housekeeping]

**Sources.** `open-questions.md` § Deferred User Stories — *"Regression-test
automation for the distributable"* and *"Regression-test automation —
declassified from Phase 3"*; `REQUIREMENTS.md` FR-16.

**Problem.** Two near-duplicate stories describe the same unscheduled candidate
(a script diffing `cc-template/` against a tagged baseline to flag
invariant-breaking drift). The second is the newer, more-complete framing
(post-checkpoint-006, aligned with FR-16's "unscheduled candidate"); the first is
the older Phase-3-roadmap phrasing. Carrying both invites confusion about whether
they're one item or two.

**Recommendation.** Merge into a single Deferred User Story (keep the newer
"declassified" framing; fold in anything unique from the older one), cross-linked
to FR-16, and leave it deferred until a real regression motivates it. Apply via
the `/wind-down` open-questions groom on land.

> AUDIT NOTE — JAH:
> Defer Approved: definitely defer.

#### R7b. Drop the EOL story as verified resolved [Suggested priority: housekeeping]

**Sources.** `open-questions.md` § Deferred User Stories — *"`/refresh-from-repository`
defers EOL handling to git instead of detecting it"*; `design-decisions.md`
*"Line endings: rely on `core.autocrlf`, not a committed `.gitattributes`"*;
verification this session — `refresh-from-repository.md` (both copies) carries no
CRLF/LF/whitespace-detection logic, and no `.gitattributes` exists in the repo.

**Problem.** The story proposed two coupled actions: add a `.gitattributes`, and
make the refresh command stop detecting CRLF-vs-LF. The later "rely on
`core.autocrlf`" decision deliberately **declined** `.gitattributes` for this
repo, and verification shows the refresh command never carried EOL-detection
cruft to remove. Both halves are settled or moot; the consumer-facing
`* text=auto` recommendation already lives in `environment-rules.md` and stands.

**Recommendation.** Drop the story from the backlog as resolved/moot (record a
one-line pointer to the `core.autocrlf` decision so it isn't re-derived). Apply
via the `/wind-down` open-questions groom on land.

> AUDIT NOTE — JAH:
> Accepted: drop the story as resolved, with a one-line pointer to the
> `core.autocrlf` decision so it is not re-derived. `/wind-down` applies the
> drop during the backlog groom.

#### R8. New capability: lightweight in-session plan re-flow ritual [Suggested priority: P3 — live pain; couples with R5]

**Sources.** Live discovery on a greenfield downstream project (Jamie, this
session) — recurring mid-stream blockers / major discoveries that require
re-sequencing the current movement's plan; `REQUIREMENTS.md` NFR-8 (file-ownership
non-overlap), FR-3 (`/onboard` decomposes the PRD into the plan), FR-6
(`/design-review` as the high-risk-transition gate); `ARCHITECTURE.md` §
"PROJECT_PLAN movement header & phase-status token"; `CLAUDE.md` § "Movement vs.
tactical"; `design-decisions.md` *"Design reviews are first-class phases and
prompts"* (the don't-edit-landed-prompts-retroactively convention);
`open-questions.md` § Deferred User Stories *"Document the out-of-band 'in-movement
enhancement' lane"* (R5).

**Problem.** Early- and mid-movement discovery routinely invalidates a plan's
*sequencing* — move / split / defer phases, add a step to the current phase —
without invalidating the product vision. No skill owns that adjustment, so the
maintainer is forced into one of two bad options: (a) edit `PROJECT_PLAN.md` /
`CLAUDE_CODE_PROMPTS.md` / `design-decisions.md` / `open-questions.md` /
`TODO.txt` inline, which violates the ownership map (NFR-8) and, worse, rewrites
an already-run prompt — an immutable execution record whose "revisions since this
prompt ran" footer is for *deviations*, not rewrites; or (b) run a full
`/design-review`, the correct guardrail for a genuinely high-risk transition but
days of overhead for a low-risk re-sequence already understood. The missing
middle is a lightweight, in-session, ownership-respecting way to act on a fresh
discovery. Trigger phrase observed in practice: "re-flow the plan."

**Recommendation.** Adopt an in-session "re-flow the plan" ritual whose
load-bearing disciplines hold regardless of form: **never rewrite a run prompt**
(append a point-release prompt, e.g. `Prompt N.1`, with the refocused scope;
leave Prompt N and its footer as-run — this project already has the `2.1.A`
point-release precedent); **re-sequence ≠ re-scope** (re-ordering / splitting /
deferring phases within the movement is plan-level and in-session-adjustable;
changing *what the product delivers* escalates to `/product-visioning` →
`/onboard`); **risk gates the weight** (a low-risk re-sequence is the lightweight
ritual; a high-risk transition still triggers `/design-review` + a numbered
checkpoint); and **doc edits route through their owners** (`/wind-down` lands the
`PROJECT_PLAN` / `design-decisions` / `open-questions` / `TODO` changes plus the
re-scope rationale). The **first-level decision is the ritual's form** — pick one:

- **(A) Documented convention, no new command (simplest / KISS default).**
  Capture the four disciplines as doctrine in `CLAUDE.md` (extending the
  "Movement vs. tactical" framing into a lane taxonomy) and let `/wind-down`
  enact the doc edits it already owns. No new surface; risk: relies on prose
  adherence at the moment of discovery.
- **(B) A new lightweight skill (e.g. `/reflow`).** Owns the in-session
  re-sequence as a hard ritual: appends `Prompt N.1`, adjusts `PROJECT_PLAN`
  sequencing, records rationale, routes anything heavier to its proper owner.
  Most reliable (an invoked skill beats soft prose) but adds a command — must
  clear the "earned its place" bar, which the live multi-occurrence evidence
  arguably meets.
- **(C) Extend `/onboard` with a current-movement re-scope mode.** Keeps
  plan-decomposition ownership in one place (it already authors `PROJECT_PLAN` +
  prompts); risk: `/onboard` is the decompose-from-PRD command, and a
  mid-movement reflow with no new PRD stretches its identity.

Per rule 4, (A) and (C) are the simpler alternatives to a new command — named so
(B) is a deliberate choice, not a default. **Couple with R5:** this is a fourth
lane in the same Movement / in-movement-enhancement / tactical taxonomy R5 and the
Movement→Phase→Step vocabulary story are settling, so the lane names and the
re-flow ritual must land coherently (don't name lanes here that the vocabulary
settlement would later rename).

> AUDIT NOTE — JAH:
> DECISION: Option A — a written convention, no new command — named "plan
> revision", not "re-flow": a small change to the plan's steps or phases, made
> in the session, when the goal has not changed. It lands as about six lines in
> the same shipped `rules/project-rules.md` block as R5. Rules: (1) A step that
> has run is never changed; follow-up work becomes a new step. (2) A step that
> has not run can be rewritten or removed. (3) Numbers never change once
> assigned: adding at the end takes the next number; inserting in between takes
> a letter (Step 5.2a goes between 5.2 and 5.3); phases work the same way.
> (4) Claude proposes the revision, Jamie approves, Claude edits the plan and
> prompts; `/wind-down` records why. Limits: if the product goal changes it is
> not a plan revision and goes back to `/product-visioning`; if the change is
> risky it gets a `/design-review`.

### Notes — acceptable now, revisit later

#### N1. De-personalization (#16) record — heavy form rejected, README prompt survives

**Sources.** `open-questions.md` § Deferred User Stories — *"Parameterize
maintainer identity (name / pronouns / initials) for the public repo"*;
`REQUIREMENTS.md` NFR-7.

**Problem / record.** The `/onboard`-driven identity-parameterization approach is
**rejected** (rule 3 — permanent): the load on every shipped file plus the
`AUDIT NOTE — JAH:` stage-detection coupling isn't worth it. The previously-flagged
collision with the copyright-stamp story dissolves — depersonalizing a consumer's
copy never rewrites the upstream copyright line, so the two are independent.

**Recommendation.** Replace the story with a small surviving scope: a README-level
prompt a consumer can hand to Claude ("tell me your name/initials and I'll change
Jamie's everywhere **except** the copyright"). Record the rejection so it isn't
re-proposed; the README-prompt residual can ride with R1 (launch readiness) or
stay a standalone low-priority nicety. Apply via the `/wind-down` groom on land.

> AUDIT NOTE — JAH:
> Defer Approved: leave it in the backlog.

#### N2. Documentation-tail items follow the behavior they document

**Sources.** `open-questions.md` § Deferred User Stories — *"Document the
file-lifecycle patterns in the user-facing SDLC docs"* and the documentation half
of *"Document the out-of-band 'in-movement enhancement' lane"*;
`design-decisions.md` *"Documentation is downstream of behavior."*

**Problem / record.** These are `/write-documentation` passes that should land
*after* the behavior they describe (the lane doctrine from R5; the file-lifecycle
patterns already in ARCHITECTURE/design-decisions). Per the downstream-of-behavior
doctrine, they are not independent work to sequence — they reconcile once their
source behavior is settled.

**Recommendation.** Keep deferred; attach each to its source cluster (the
lifecycle-patterns write-up and the lane's user-facing surfacing follow R5/R6
landing) rather than ranking them on their own.

> AUDIT NOTE — JAH:
> Accepted with caveats: not deferred. Both write-ups land in a
> `/write-documentation` pass that runs after the build, as the last step of
> the batch. That pass is needed anyway because the batch changes what users
> see (the four-file session start, the output style, plan revision, the
> license comment); the lane and the file-lifecycle patterns ride it.

#### N3. Low-priority housekeeping singletons — internal manifest + memory sweep

**Sources.** `open-questions.md` § Open Questions — *"Internal project manifest —
record the purpose of internal artifacts"*; § Deferred User Stories — *"Sweep
local Claude-memory items… into checked-in docs"*; precedent `docs/design/REVIEWS.md`.

**Problem / record.** The internal-manifest question (do we maintain per-folder
indexes / a single MANIFEST / nothing) naturally pairs with R2's backlog-cleanup
convention — such a review would consume an index — but is otherwise a low-stakes
"do we need this" call (the existing zero-padded-NNN + per-folder naming already
encodes most intent; option C "do nothing" is the KISS default). The memory-sweep
item is partly machine-local and folds into R4's hygiene sweep.

**Recommendation.** Leave both as low-priority; surface the internal-manifest
decision alongside R2 if that cluster builds, default to "do nothing" otherwise.

> AUDIT NOTE — JAH:
> DECISION: do nothing on the internal manifest; drop the open question. The
> numbered filenames, purpose-named folders and frontmatter already say what
> each file is. Nothing further on the memory sweep; R4 covers it.

#### N4. Landing mechanics — how accepted clusters reach the plan

**Sources.** `REQUIREMENTS.md` NFR-8 (file ownership); `CLAUDE.md` § File
ownership; this command's Stage 2 edit scope.

**Problem / record.** Because `/design-review` doesn't own `open-questions.md`,
the land path can only (a) promote accepted clusters into `PROJECT_PLAN.md` as
phase(s)/steps and (b) surface a TODO for `/wind-down` to enact the backlog
re-groom (re-cluster, the R7 prunes/merges, the N1 replacement). No finding here
edits the backlog file directly.

**Recommendation.** When marking, if a cluster should become a plan phase
immediately, say so in the caveat (`lane → in-movement`, etc.); if it should stay
in the backlog re-grouped, that routes to `/wind-down`. This Note needs no
decision of its own — it's the map for reading the others.

> AUDIT NOTE — JAH:
> Accepted: this is the map. Nothing to build.

#### N5. Net-new (post-groom): per-movement research/intake holding area for `docs/design/`

**Sources.** `open-questions.md` § Deferred User Stories — *"Per-movement
research / intake holding area so `docs/design/` reads as current truth, not
provisional exploration"* (added this session); `docs/design/` current contents
(intake `cc-template-product-spec.md` + PRDs + checkpoints + `REVIEWS.md`
co-located); the `/design-review`-reads-open-questions bundle (R2) and the
internal-manifest singleton (N3).

**Problem / record.** Discovered 2026-06-29 dogfooding `/product-visioning` in an
empty project — surfaced *after* this groom read the backlog, so it is **not** one
of the 28 clustered stories above; it is the 29th. Provisional visioning material
(discarded options, the source design doc, research notes) has no home distinct
from current design truth, so `docs/design/` conflates "what is current" with
"what was provisional or rejected." Recorded here so it gets a lane in this same
sequencing pass rather than waiting for the next groom.

**Recommendation.** Disposition the new backlog story's first-level decision —
whether to add a per-movement research / intake holding area, and where it lives —
and assign a lane (likely low priority; no acute external pain). Named, not
specced: it pairs with the internal-manifest question (N3) and the
`/design-review`-reads-open-questions work (R2), since a quarantined-intake area
changes what the whole-file read should treat as live vs. historical.

> AUDIT NOTE — JAH:
> Defer Approved.

#### N6. Net-new (post-groom): S1.8 inlines a commit handoff its own "does NOT do" clause routes to `/wind-down`

**Sources.** `open-questions.md` § Deferred User Stories — *"`/design-review`
S1.8 inlines a commit handoff its own 'does NOT do' clause (and rules 7/9) route
to `/wind-down`"* (added this session); `.claude/commands/design-review.md`
Step S1.8 vs. its "What this command does NOT do" clause (both copies identical);
`coding-session-rules.md` rules 7 & 9; `CLAUDE.md` § Load-bearing invariants
("Stage 1 *initial* … surface a rule-7 handoff"); the existing "Artifact-boundary
command landings should route session-end through `/wind-down`" story.

**Problem / record.** Discovered 2026-06-29 in the same downstream greenfield
project, post-`/onboard` — surfaced *after* this groom read the backlog, so it is
a coherence defect rather than one of the clustered stories above. S1.8 inlines a
`git add` / `git commit` block for the Stage 1 initial branch, while the file's
own "does NOT do" clause routes that handoff to `/wind-down`; rules 7 & 9 agree
with the clause, so S1.8 is the stale surface. Recorded here so it gets a lane in
this same sequencing pass.

**Recommendation.** Disposition the fix direction (route S1.8 to `/wind-down`,
per rules 7/9 and the file's own clause) and assign a lane. First-level only,
named not specced: landing it reconciles four surfaces — S1.8, the "does NOT do"
clause, the CLAUDE.md invariant wording, and the existing artifact-boundary
story's "Stage 1 … stays as-is" carve-out (now the minority view) — across both
command copies (NFR-9), and likely has an `/exit-test-plan` twin.

> AUDIT NOTE — JAH:
> Accepted: S1.8 is rewritten to invoke `/wind-down` and its inline git block
> is deleted, as part of the command batch. `/exit-test-plan` carries no such
> block, so there is nothing to pair.

#### N7. Net-new (post-groom): `/bootstrap` inlines its commit handoff instead of routing to `/wind-down` — the policy frame for N6

**Sources.** `open-questions.md` § Deferred User Stories — *"Configuration
commands inline their own commit handoff instead of routing through `/wind-down`"*
(added this session); `.claude/commands/bootstrap.md` Step 7 + Stage 2 S3 +
preamble (both copies); `coding-session-rules.md` rules 7 & 9; N6 above and the
"Artifact-boundary command landings…" story.

**Problem / record.** Discovered 2026-06-29 in the same downstream project during
`/bootstrap` — surfaced *after* this groom read the backlog. `/bootstrap` inlines
a `git` commit block borrowing `/wind-down`'s Step 4 format without invoking
`/wind-down`, in tension with rules 7 & 9. Unlike N6 (a self-contradiction the
rules settle), this has a defensible rationale — `/bootstrap` is a mid-stream
config command and full `/wind-down` would wrongly fire its TODO.txt-rewrite +
coherence-sweep ritual — so it is a genuine policy question, and the frame N6 sits
inside: do commit-worthy command boundaries route to `/wind-down` (possibly a
lightweight config-commit mode) or get a sanctioned inline exception?

Reinforcing evidence same day: Stage 2 `/bootstrap` (the `COMPLETE` flip) gave
"what's next" guidance but recommended no `/wind-down`, so `TODO.txt` ended the
session stale and no coherence sweep ran — completion *is* a session boundary, so
the absent full ritual is a real gap, not a saved cost. This weighs the
route-vs-sanction call toward routing (a lightweight inline commit alone would
still leave `TODO.txt` stale) and answers the artifact-boundary story's "probably
no" sub-question for `/bootstrap` Stage 2. Jamie's read: "the wind-down really is
necessary." (Recorded as directional input, not a marking.)

**Recommendation.** Disposition the route-vs-sanction policy and assign a lane,
treating N6, this, and the "Artifact-boundary command landings…" story as one
coherent decision (a candidate merge for the next groom). First-level only, named
not specced: scope spans the config commands (`/bootstrap`, `/deployment-plan`,
maybe `/onboard`) and the artifact-boundary commands, across both copies (NFR-9).

> AUDIT NOTE — JAH:
> DECISION: route. Every command that reaches a commit point invokes
> `/wind-down`, full ritual, no lightweight mode. The inline git blocks in
> `bootstrap` (Step 7 and Stage 2 S3), `deployment-plan` and `onboard` are
> deleted and each ends by invoking `/wind-down`, as part of the command batch.
> This closes the three overlapping routing stories in the backlog (this one,
> N6's, and "Artifact-boundary command landings…"); that story's Thread 2 (scan
> `PROJECT_PLAN.md` for future checkpoints at landing) stays deferred.

## What the design got right (preserve)

- **The backlog discipline itself is working.** Stories cross-reference the
  checkpoints that deferred them, carry explicit re-open triggers, and record
  "settled going in" notes — that hygiene is exactly why a groom this size is
  tractable rather than archaeology.
- **Encoding session corrections so they're portable** (the recurring shape of
  the `/design-review` and `/wind-down` stories) is the right instinct: a fresh
  downstream Claude has no memory of a maintainer correction, so the spec is the
  only teacher.
- **The downstream-of-behavior doctrine** cleanly sorts the documentation items
  out of the sequencing decision — they follow, they don't lead.
- **Deferral with a trigger, not silent backlog growth** — several items
  (status-callout, regression automation) carry the condition that would re-open
  them, which is what makes dropping/merging them now a safe call.

## Next steps

1. Jamie marks each finding inline (AUDIT NOTE blocks above). For grouping
   findings, the useful shape is `Accepted with caveats: priority → P_,
   lane → <…>, split/keep <…>`.
2. Re-run `/design-review`. Stage 2 walks each disposition, appends rows to the
   Disposition log, and asks whether to land the doc or open another addendum
   round.
3. On land: accepted clusters become phase(s)/steps + prompts in
   `PROJECT_PLAN.md` / `CLAUDE_CODE_PROMPTS.md` at the marked priorities, and the
   `open-questions.md` re-groom (R7 prunes/merges, N1 replacement, re-clustering)
   is handed to `/wind-down`.

## Disposition log

*Stage 2 fills this section. Empty until Stage 2's first walk. Rows accumulate
append-only across rounds — later-round dispositions on the same finding ID
supersede earlier ones for the purposes of Stage 2's landing application.*

| Round | Finding | Disposition | Reason |
|-------|---------|-------------|--------|
| Original | R1 | Accepted with Caveats | Plain comment per distributed file pointing at `.claude/claude-code-sdlc-template-license.md`; refresh changed to deliver it; no SPDX; doc delivery stays with `/deployment-plan`. |
| Original | R2 | Accepted with Caveats | All six stories as one-to-three-line edits; four in `design-review.md`, two in `wind-down.md`. |
| Original | R3 | Accepted with Caveats | Four prescriptive-prompt lines in `onboard.md`; doc-intent story stays deferred. |
| Original | R4 | Accepted with Caveats | Two-sentence repo-wide sweep change in `wind-down.md`; debt stories stay in the backlog. |
| Original | R5 | Accepted with Caveats | Three kinds of work plus Movement/Phase/Step defined once in `project-rules.md`; rename only; root `CLAUDE.md` not edited. |
| Original | R6 | Decision | `CLAUDE.md` stays as is; five stories dropped; four-file session-start read; template ships the output style and `outputStyle` setting, delivered by refresh per Jamie; Bash deny is a README suggestion; portability audit deferred. |
| Original | R7a | Deferred | Definitely defer; duplicate stories stay as they are. |
| Original | R7b | Accepted | EOL story dropped as resolved, with a pointer to the `core.autocrlf` decision. |
| Original | R8 | Decision | Option A, "plan revision": four rules and two limits in the same `project-rules.md` block; no new command. |
| Original | N1 | Deferred | Story stays in the backlog. |
| Original | N2 | Accepted with Caveats | Not deferred; a `/write-documentation` pass follows the build as the batch's last step. |
| Original | N3 | Decision | No internal manifest; question dropped; memory sweep covered by R4. |
| Original | N4 | Accepted | The landing map; nothing to build. |
| Original | N5 | Deferred | Intake holding area stays in the backlog. |
| Original | N6 | Accepted | S1.8 invokes `/wind-down`; inline git block deleted. |
| Original | N7 | Decision | Route: every commit point invokes `/wind-down`; inline git blocks deleted from `bootstrap`, `deployment-plan`, `onboard`; closes the three routing stories. |

## Sign-off Summary

*Filled when Stage 2 lands the doc. Jamie does not edit this section — sign-offs
go inline above.*

| ID | Final disposition |
|----|-------------------|
| R1 | Accepted with Caveats |
| R2 | Accepted with Caveats |
| R3 | Accepted with Caveats |
| R4 | Accepted with Caveats |
| R5 | Accepted with Caveats |
| R6 | Decision: `CLAUDE.md` stays as is; four-file session-start read; ship the output style |
| R7a | Deferred |
| R7b | Accepted |
| R8 | Decision: Option A, "plan revision" convention |
| N1 | Deferred |
| N2 | Accepted with Caveats |
| N3 | Decision: no internal manifest |
| N4 | Accepted |
| N5 | Deferred |
| N6 | Accepted |
| N7 | Decision: route every commit point through `/wind-down` |

## Follow-up actions landed

- R1 / R2 / R3 / R4 / R5 / R6 / R8 / N6 / N7: edited `docs/PROJECT_PLAN.md` —
  added Phase 3 "Checkpoint 007 backlog batch" with Step 3.1 (build,
  `cc-template/` only) and Step 3.2 (propagate to root and document).
- Same: edited `docs/CLAUDE_CODE_PROMPTS.md` — added Prompt 3.1 and Prompt 3.2;
  header now reads "one prompt per step".
- R5 / R8: edited `docs/ARCHITECTURE.md` — the terminology bullet is now the
  Movement → Phase → Step definition with the plan-revision numbering rules.
- R1 / R6: edited `docs/ARCHITECTURE.md` refresh paragraph and
  `docs/REQUIREMENTS.md` FR-13 — refresh delivers the `.claude/` template files
  at logic version 4.
- R1: edited `docs/REQUIREMENTS.md` NFR-7 — provenance comment + license file.
- R6: edited `docs/REQUIREMENTS.md` — added FR-19 (session-start output style).
- N2: the `/write-documentation` pass is Step 3.2's second deliverable.
- R7a / N1 / N5: deferred — recorded in this review's audit trail; stories stay
  in the backlog.
- TODO for `/wind-down` (backlog groom; outside this command's edit scope):
  drop the thin-index, status-callout, quick-orientation, marker-guardrail and
  good-morning stories (R6); drop the EOL story with a `core.autocrlf` pointer
  (R7b); drop the internal-manifest open question (N3); close the three
  commit-handoff routing stories, keeping the "scan `PROJECT_PLAN` at landing"
  thread deferred (N6, N7); leave R3's doc-intent story, R4's three debt
  stories, the duplicate regression stories (R7a), the de-personalization story
  (N1) and the intake holding area (N5) in the backlog; record the plan-revision
  convention and the four-file session-start read in `design-decisions.md`.
- Not touched, per Jamie's standing ruling: root `CLAUDE.md` (its "Movement vs.
  tactical" bullet names two kinds of work; `project-rules.md` will name three).
