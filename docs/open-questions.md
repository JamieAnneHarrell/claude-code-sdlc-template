# Open Questions and Engineering Notes

Working surface for unresolved questions, deferred decisions,
abandoned approaches, and things worth revisiting. Update as answers
are found or decisions are made.

`/wind-down` proposes additions to this file at session end. `/onboard`
seeds it with any open questions that the design intake left
unresolved.

## How this file relates to other tracking

- **`TODO.txt`** — questions that *must* be resolved at the start of
  the next session belong there, not here. This file is for genuinely
  open / non-blocking items.
- **`design-decisions.md`** — once a question gets answered with a
  real decision, it moves there.
- **`PROJECT_PLAN.md`** — when an open question becomes scoped work,
  it becomes a phase or a phase deliverable.

## Categories

Use these section headers. Add a category section only when there's a
real entry to put under it; don't pre-create empty sections.

### Deferred User Stories
Things we've thought about and decided not to build yet. Format:
*Context* (what / why), *Proposed approach*, *Open sub-questions*.
Lift the name into PROJECT_PLAN.md when it's time to build.

### Known Limitations
Things the current implementation does NOT handle gracefully, with
the conditions that trigger the limitation and a mitigation if there
is one. Document so future debugging starts here.

### Abandoned Approaches
Things we tried, why they failed, and what replaced them. Useful when
someone (Claude or human) tries the same thing again — the file
reminds them why it didn't work.

### Open Questions
Questions we haven't yet answered, with what we know so far and what
would unblock an answer.

---

### Open Questions

#### Reconciliation algorithm for `/refresh-from-repository`

*What we know.* The 2026-05-26 design cleanup settled the contract
for `/refresh-from-repository` (template marks its own content,
block-level reconciliation matrix, semantic conflict detection runs
in-session — see `docs/design-decisions.md` "Template marks its own
content; reconciliation tolerates divergence"). What it did *not*
settle is the precise mechanism the merge uses to compare
downstream-current against upstream-current and a baseline.

*Options to consider.*
- **Per-block content hash recorded in the marker.** Each template
  block carries `hash=<sha>` in its open marker; refresh recomputes
  the live hash and compares. Self-contained per file; no out-of-
  band state needed.
- **File-level commit-stamp baseline.** A `last-synced-at:
  <upstream-commit>` stamp at the top of each refresh-managed file;
  refresh fetches that upstream commit's version of the file as the
  baseline. Requires upstream to keep history accessible (already
  true via GitHub).
- **Git 3-way merge against tagged upstream releases.** Upstream
  tags releases; downstream records the tag it sync'd from; refresh
  runs `git merge-file` between downstream, current upstream, and
  the baseline-tag content. Leverages git's merge engine; requires
  release-tag discipline upstream.
- **Hybrid.** Hash per block (cheap inline-edit detection) PLUS
  file-level commit stamp (baseline for the in-baseline-vs-new
  decision). The two signals are complementary.

*What would unblock.* The Phase 2.1 build session picks one and the
pre-Phase-2.1 `/design-review` sanity-checks the choice. Constraints:
must support the matrix in `docs/design-decisions.md`; should not
require maintaining release-tag discipline upstream as a hard
prerequisite (acceptable as part of a hybrid).

### Deferred User Stories

#### Plan-mode / checklist execution should surface `/wind-down`, not inline commit handoffs

*Context.* When Claude finishes a plan-mode execution (or any ad-hoc
checklist-driven multi-step task that ends at session close), the
natural last step Claude surfaces today is a rule-7 commit handoff
— stage list, `git status`, `git commit -m "..."` block. That gets
the changes committed, but it skips the rest of `/wind-down`'s job:
TODO.txt refresh (rule 9), `docs/design-decisions.md` /
`docs/open-questions.md` / `docs/CLAUDE_CODE_PROMPTS.md` deviation-
footer / `docs/ARCHITECTURE.md` / `docs/REQUIREMENTS.md` /
command-doc coherence sweep, and any artifact-status callouts. The
consumer who follows the inline commit handoff lands the changes
but leaves the rest of the repo's docs in whatever state the plan
execution left them.

Jamie surfaced this at the end of the 2026-05-26 Prompt-2.1 cleanup:
the surfaced commit block worked, but `/wind-down` would have
duplicated the commit check AND done the doc coherence pass.
Running both is wasteful; choosing only the inline commit skips work.

Distinct from artifact-boundary handoffs in commands like
`/onboard`, `/bootstrap`, `/design-review` Stage 1 initial / Stage
2 landing, `/exit-test-plan` Stage 1 initial / Stage 2 landing —
those commit at their own artifact boundary and are by design
separate from session-end wind-down. The user story is about
**plan-mode / checklist-driven** execution that has no
artifact-boundary handoff of its own.

*Proposed approach.* Update the plan-mode workflow (and any future
checklist-driven execution conventions) so the final step is
"propose `/wind-down`" — Claude says "all plan steps complete;
run `/wind-down` to commit and refresh docs" rather than surfacing
the inline git block. `/wind-down`'s existing Step 4 (re-read rule
7 then surface the commit handoff) absorbs the commit work; its
other steps cover the doc coherence pass.

Implementation surfaces to touch:

- `rules/coding-session-rules.md` rule 9 — add a line clarifying
  that plan-mode and checklist execution route session-end commits
  through `/wind-down` rather than surfacing them inline.
- Anywhere in Claude Code's plan-mode behavior that auto-surfaces
  commit handoffs at plan completion (if any — may be
  emergent-from-rule-7 rather than configured).
- `.claude/commands/wind-down.md` — add a "called-from-plan-mode"
  intake path if needed (most likely just an extra reminder in the
  command preamble: "if you came here from plan-mode completion,
  do the full doc-coherence pass before the commit").

*Open sub-questions.* Whether artifact-boundary commands
(`/onboard`, `/bootstrap`, `/design-review`, `/exit-test-plan`)
should also route through `/wind-down` at their artifact boundaries
(probably no — those *are* the artifact boundary, and forcing them
through `/wind-down` collapses two distinct moments). Whether
"checklist-driven execution" needs a more precise definition
(e.g., does TodoWrite-tracked work count? It probably should).

#### "Before running this prompt" header-block pattern in CLAUDE_CODE_PROMPTS.md

*Context.* `/onboard`'s spec for `docs/CLAUDE_CODE_PROMPTS.md`
(see `.claude/commands/onboard.md` step `docs/CLAUDE_CODE_PROMPTS.md`,
"Design-review checkpoints between prompts") plants a "Before
running this prompt:" header-block note at the top of each
forward-looking prompt to signal a between-phase `/design-review`
checkpoint. The same checkpoint is *also* recorded as a first-class
entry in `docs/PROJECT_PLAN.md` for high-risk transitions (e.g.
`## Design Review Checkpoint — pre-Phase-2.1`).

The header-block notes ended up conflating two things:

- **Gate assertion** — "the pre-Phase-X review has landed" stated as
  a fact, but the prompt has no way to verify the gate at execution
  time. If false, the consumer notices only after starting the work.
- **Next-session reminder** — content that arguably belongs in
  `TODO.txt` at the moment the consumer picks up the prompt, not
  inside the prompt body.

Jamie surfaced this during the 2026-05-26 cleanup of Prompt 2.1 and
the block was stripped from that prompt specifically. The other
forward-looking prompts (2.2, 3) still carry it. Landed prompts
(1.1, 1.2) carry historical-state header blocks; the file's
convention is to leave landed prompts unedited retroactively.

*Proposed approach.* The pre-Phase-2.1 `/design-review` should
evaluate the pattern across the file:

- Retire the `/onboard` header-block-note form entirely; rely on
  PROJECT_PLAN.md first-class checkpoint entries alone as the
  between-phase signal.
- Or, keep the header-block pattern but rewrite the language so it
  reads as a gate-check rather than a gate-assertion ("If the
  pre-Phase-X review hasn't landed yet, stop and run it first.").
- Or, hybrid: keep landed prompts' header blocks (historical
  record) and retire only the forward-looking-prompt form.

The review's disposition should update `/onboard`'s spec
accordingly so future onboards don't reproduce the pattern.

*Open sub-questions.* Whether the same critique applies to other
inside-prompt-body content that arguably belongs in TODO.txt at
prompt-pickup time (e.g. "Read first" lists that re-state the
file-level reading-order banner).

#### In-flight artifact status callout at top of CLAUDE.md

*Context.* ds-niche-stream's CLAUDE.md opens with a 🟡 callout
summarizing current in-flight artifact state (e.g. "Phase 1.3 exit
`AWAITING-DISPOSITIONS` — paused on... Next action is
`/design-review` ... Phase 2 is blocked behind the landing"). At
session start this greets the developer with the single most
important piece of "where are we" context without requiring a
TODO.txt or test-plan scan. ds-niche-stream appears to maintain
this manually.

*Proposed approach.* Surface as a `/wind-down` enhancement:
detect when an artifact is mid-flight (any
`docs/design/design-review-checkpoint-*.md` is
`AWAITING-DECISIONS`, or any `docs/test-plans/phase-*-exit.md` is
`AWAITING-DISPOSITIONS`), and write/refresh a 🟡 callout block at
the top of CLAUDE.md describing the state and the next action.
Clear the callout when nothing is mid-flight. Marker-block
delimited so it doesn't conflict with the rest of CLAUDE.md.

*Open sub-questions.* Whether `/wind-down` should also detect
`/exit-test-plan` polish-loop states (Stage 1 addendum branch,
Stage 2 open-another-round) and reflect those in the callout.
Whether the callout belongs in CLAUDE.md or in a separate file
like `STATUS.md` that CLAUDE.md links to. Whether to introduce a
fourth status comment family (something like
`<!-- INFLIGHT-STATUS: ... -->`) for machine-parseable state.

#### "Project quick orientation" section in onboarded CLAUDE.md

*Context.* ds-niche-stream (the most mature downstream consumer)
evolved a rich `## Project quick orientation` section in its
CLAUDE.md below the banner. Contents include a multi-paragraph
project description, stack bullets that go beyond the banner
(Admin UI, Public stack, Database engine, Formatter / static
analysis, Main source path with a pointer into
`docs/ARCHITECTURE.md`), and full format/lint commands. Neither
`/onboard` nor `/bootstrap`'s spec produces this section — the
banner alone (Project / Language / Multi-agent / Shell / Run /
Test / Developer setup) is terse. A new project onboarded today
would miss the things that come up often at session start: DB
engine, source-tree location, public-vs-admin stack split for
web projects.

*Proposed approach.* Address via a future `/design-review`
checkpoint that compares this project's freshly-produced CLAUDE.md
against ds-niche-stream's shape and decides whether `/bootstrap`
should write a "Project quick orientation" section alongside the
banner, or whether the banner alone is enough (KISS — don't add a
section every project has to maintain).

*Open sub-questions.* If the section is added, which command
owns it — `/onboard` (most project context is already collected
there) or `/bootstrap` (knows the stack + commands)? How much
overlaps with the banner before the duplication starts hurting?

#### Regression-test automation for the distributable

*Context.* The dist is a copy-paste seed that must continue to work
end-to-end (full /onboard → /design-review → /bootstrap → ...
chain). Today regression checking is manual ("copy cc-template/ to
a sandbox dir and run the chain"). A scripted check could diff the
current dist against a tagged baseline and flag invariant-breaking
changes (status comment renames, ONBOARD-FILL marker drift,
zero-pad width changes in checkpoint/test-plan filenames).

*Proposed approach.* Phase 3 roadmap. Specifics deferred until we
have a real regression to motivate the work.

