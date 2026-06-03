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

### Deferred User Stories

#### Artifact-boundary command landings should route session-end through `/wind-down`

*Context.* When Claude finishes a self-contained multi-step task
that ends at session close — plan-mode execution, TodoWrite-driven
checklist work, OR an artifact-boundary command like
`/design-review` Stage 2 landing or `/exit-test-plan` Stage 2
landing — the natural last step Claude surfaces today is a rule-7
commit handoff (stage list, `git status`, `git commit -m "..."`
block). That gets the immediate scope committed, but it skips
the rest of `/wind-down`'s job: TODO.txt refresh (rule 9),
`docs/design-decisions.md` / `docs/open-questions.md` /
`docs/CLAUDE_CODE_PROMPTS.md` deviation-footer /
`docs/ARCHITECTURE.md` / `docs/REQUIREMENTS.md` / command-doc
coherence sweep, and any artifact-status callouts. The result:
the immediate scope lands but the surrounding doc cluster drifts.

Jamie surfaced this twice on 2026-05-26 — first at the end of the
Prompt-2.1 cleanup, then again at the end of checkpoint 002's
Stage 2 landing. Both times, the artifact-boundary command's
commit handoff fired (its job done by spec), but the *session*
had additional doc work that only `/wind-down` would have caught
(design-decisions updates, open-questions migrations, etc.). The
session-aware wind-down beats next-session drift discovery —
in-session, we know what we just did; next session has to
rediscover it.

*Proposed approach (two coupled threads).*

**Thread 1 — Route session-end commits through `/wind-down`.**
Update the plan-mode workflow, TodoWrite-driven execution
conventions, AND artifact-boundary command landings
(`/design-review` Stage 2 land path; `/exit-test-plan` Stage 2
land path) so the final step is "propose `/wind-down`" — Claude
says "all steps complete; run `/wind-down` to commit and refresh
docs" rather than surfacing the inline git block as the terminal
action. `/wind-down`'s existing Step 4 (re-read rule 7 then
surface the commit handoff) absorbs the commit work; its other
steps cover the doc coherence pass. Stage 1 artifact-boundary
commits (e.g. checkpoint authored, plan authored) are
mid-iteration and stay as-is — they're not session-end events.

**Thread 2 — `/design-review` Stage 2 landing should also review
the overall project plan.** When a checkpoint lands, Stage 2 should
take a pass at `docs/PROJECT_PLAN.md` to ask: "given what was just
decided, are any future high-risk transitions in the plan now
worth a first-class `/design-review` checkpoint?" If yes, insert
a `## Design Review Checkpoint — pre-Prompt-N` first-class entry
into `docs/CLAUDE_CODE_PROMPTS.md` between the relevant prompts.
This keeps design reviews in the prompt flow as discoverable
first-class items that surface in TODO.txt at session start.
Distinct from the header-block-note form retired by checkpoint
002 R5 — first-class entries live in the phase queue, not embedded
in prompt bodies.

*Implementation surfaces to touch.*

- `rules/coding-session-rules.md` rule 9 — add a line clarifying
  that plan-mode, TodoWrite-driven execution, AND artifact-boundary
  command landings route session-end commits through `/wind-down`
  rather than surfacing them inline.
- `.claude/commands/wind-down.md` — add an intake path for
  "called from an artifact-boundary command's landing" so wind-down
  picks up where the command's Stage 2 left off (likely just a
  preamble note: "if you came here from a `/design-review` or
  `/exit-test-plan` landing, the artifact-boundary commit was
  already drafted there; bundle it with this wind-down's edits").
- `.claude/commands/design-review.md` Stage 2 S2.6 land path — add
  a step that walks `docs/PROJECT_PLAN.md` for high-risk
  transitions and proposes first-class checkpoint inserts in
  `docs/CLAUDE_CODE_PROMPTS.md` when appropriate (per Thread 2).
- `.claude/commands/design-review.md` and
  `.claude/commands/exit-test-plan.md` Stage 2 S2.7 — replace the
  terminal commit handoff with "propose `/wind-down`" on the land
  path. Mid-iteration paths (open-another-round) stay as-is —
  they already defer to wind-down for staging.
- Mirror all spec edits to `cc-template/.claude/commands/` per
  NFR-9.

*Open sub-questions.* Whether `/onboard` and `/bootstrap` Stage 2
landings should also route through `/wind-down` (probably no —
those are configuration-ritual commands whose landings are also
typically the last action of their session by design; the
artifact-boundary IS the session boundary for them). Whether
"checklist-driven execution" needs a more precise definition
(e.g., does any TodoWrite-tracked work count? Probably yes).
Whether Thread 2's "scan PROJECT_PLAN for high-risk transitions"
heuristic needs concrete criteria, or whether it's best handled
as a session-judgment call surfaced for Jamie to disposition.

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

*Deferred 2026-05-29 per checkpoint 003 R8* — re-open if downstream
consumers report difficulty discovering mid-flight artifacts despite
`/wind-down` safety nets; consider skill-owned banners as a possible
mechanism (skills surface mid-iteration state independently of
TODO.txt).

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

*Re-open trigger added 2026-05-29 per checkpoint 003 N2* — when a
third downstream consumer (beyond ds-niche-stream and ds-auto-dailies)
evolves a Project quick orientation section independently, OR when
Jamie's session-start experience surfaces the banner alone is
consistently too thin.

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

#### CLAUDE.md as a thin index — remaining sub-questions

*Resolved core.* The three drift modes of the original "Trim
CLAUDE.md" story are all resolved:
1. **Self-edited "next phase" banner** — dropped from root CLAUDE.md
   (Phase 1.3 wind-down, 2026-06-02), and the banner-writing commands
   no longer emit next-phase prose. See `design-decisions.md`
   "CLAUDE.md banner carries no next-step or next-phase prose."
2. **Recurring-command cadence prose** — already removed from
   `cc-template/CLAUDE.md`.
3. **Collaboration-rules rule-by-rule summaries** — stripped. See
   `design-decisions.md` "Rules-read reliability."

*Still open.*
- Tension with the "In-flight artifact status callout at top of
  CLAUDE.md" story below — that one proposed an automated 🟡 status
  banner; the trim stance argues banners drift and TODO.txt already
  serves the purpose. Resolve before either lands; probably retire the
  callout story in favor of the trim stance.
- Whether the configuration-ritual banner (status-comment-driven, top
  of `cc-template/CLAUDE.md`) qualifies as bloat — probably earns its
  place as the entry point on a fresh project, but worth a second look.
- Whether the "Reading order at session start" section is itself bloat:
  it tells Claude to read TODO.txt, PROJECT_PLAN, CLAUDE_CODE_PROMPTS —
  habits that might belong in a hook or a skill (see "Rules-read
  reliability" in `design-decisions.md`), not in CLAUDE.md prose.

#### Seed `cc-template/TODO.txt` as the active onboarding checklist; teach the TODO-driven habit from day one

*Context.* The first thing a downstream user does is open their
freshly-copied `cc-template/` and look for "what do I do now?"
CLAUDE.md and README both name TODO.txt as the next-step surface.
Today's seeded TODO.txt has one item: "Run /onboard to configure
this project from the design doc in `docs/design/`." That makes
`/onboard` look like the literal first action, but the README
correctly says the user should review the rules first to confirm
they agree with them. That review step is silently expected, and
TODO.txt doesn't reflect it.

This compounds with the in-progress `/refresh-from-repository`
work and the CC-TEMPLATE-BLOCK marker design (the current
TODO.txt's first item). Once markers exist, the *correct*
customization sequence is: review the rules, decide which
TEMPLATE-BLOCK sections to keep verbatim vs. lift out of markers
for local edits, edit accordingly, THEN `/onboard`. None of that
is in the seeded TODO.txt today.

Two outcomes today:

1. New users skip the rules review and run `/onboard` cold; later
   discover a rule doesn't match their team with no clean
   divergence path (or, post-`/refresh-from-repository`, get
   caught off-guard by marker constraints).
2. Users don't learn the TODO-driven habit. TODO.txt is treated as
   a Claude-maintained tracking file, not a checklist humans work
   through.

*Proposed approach.* Seed `cc-template/TODO.txt` with the actual
customization-and-onboarding sequence as a numbered checklist the
user walks through:

1. Read `README.md` to understand the template shape.
2. Read each `rules/*.md` file; edit anything you disagree with
   (or, once markers ship, lift it out of its CC-TEMPLATE-BLOCK
   so `/refresh-from-repository` leaves your edits alone).
3. Drop a design doc into `docs/design/`.
4. Run `/onboard`.
5. (next session) Run `/bootstrap`.
6. (when distribution mechanism is pinned) Run `/deployment-plan`.

Update `cc-template/README.md` to describe the rule-divergence
workflow (review → edit → optionally lift from markers) and point
at TODO.txt as the working checklist. Frame TODO.txt explicitly as
"the human's checklist for this and every future session," not
just an automated handoff. Set the habit early.

*Open sub-questions.* Whether the seeded checklist should branch
by user intent ("customizing before use" vs "accepting defaults,
skip to /onboard") — risks complexity for marginal value. Whether
step 2's depth ("read every rules file") is realistic — users
often want to get to /onboard fast and only diverge later; a
softer "skim now, refine later" framing may be more honest.
Whether `/onboard` should check that TODO.txt has been walked
(e.g., does the user know they can edit rules?) or whether that's
paternalistic. Whether the seeded TODO.txt should be a sample
users can rewrite freely vs. a structured artifact whose shape
`/wind-down` preserves.

### Known Limitations

#### `/onboard` does not preserve CC-TEMPLATE-BLOCK markers when it rewrites mode-specific rules files

*Context.* `/onboard` rewrites `multi-agent-rules.md` wholesale based
on the chosen multi-agent mode. The dist placeholder wraps only the
universal `briefing-rule` block in a CC-TEMPLATE-BLOCK; the
mode-specific content is intentionally left unmarked (consumer/mode
owned). If `/onboard`'s rewrite drops or relocates the `briefing-rule`
markers, a freshly-onboarded project's `multi-agent-rules.md` may have
no markers at all.

*Why it's not currently broken.* `/refresh-from-repository`'s
pre-marker migration (Step 6) self-heals this: the first refresh in a
project with marker-less rules files re-inserts markers by aligning
against upstream. So the worst case is "markers absent until first
refresh," not "refresh misbehaves." The same reasoning covers the
onboard-appended tooling tails of `testing-rules.md` /
`environment-rules.md` — those are free regions by design.

*Open question.* Whether `/onboard`'s spec should be updated to
preserve the `briefing-rule` marker when it rewrites
`multi-agent-rules.md` (tighter, but couples `/onboard` to the marker
convention), or whether relying on pre-marker migration to re-establish
markers is sufficient (looser, already works). Surfaced during the
Phase 2.1 build; out of scope for that phase since it touches
`/onboard`, not `/refresh-from-repository`.

