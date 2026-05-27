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

#### Enforce the "MUST DO before responding" rules re-read (CLAUDE.md is necessary but not sufficient)

*Context.* Root `CLAUDE.md` Collaboration-rules section says:

> **MUST DO before responding to the first user message of any
> session.** Read these two files end-to-end — their contents are
> *not* loaded into context by default. Skipping is the #1 cause of
> rule drift...

In practice Claude regularly skips the re-read and goes straight to
the user's task. Observed failure mode: CLAUDE.md *summarizes* the
rules (KISS, rule 4, rule 7 commit shape, etc.), so Claude treats
the summary as sufficient priors and the end-to-end re-read as
redundant. The instruction names the failure mode but enforcement
relies on Claude's judgment, which is biased toward "respond now."
The downstream consequence is exactly what the rule warns about:
silent drift on KISS / rule 4 simpler-alternative / rule 7 commit
brevity, surfaced only when Jamie calls "rule N" mid-session.

For this template, the cost compounds: every downstream project
seeded from `cc-template/` inherits the same skip-prone behavior.
A reliable rules-read is foundational — without it the rest of the
collaboration-rules infrastructure is decorative.

*Proposed approach (options to weigh).*

- **Option A — Hooks-based enforcement.** A `UserPromptSubmit` hook
  on the first message of a session reads
  `rules/coding-session-rules.md` and `rules/design-philosophy-rules.md`
  into context unconditionally, removing the judgment call. Hook
  detects "first message of session" via session state (e.g.
  absence of prior assistant turns in the transcript, or a sentinel
  file written on first invocation and cleared at session close).
  Ships as part of `cc-template/`'s default `.claude/settings.json`.
  Pro: zero reliance on Claude compliance. Con: hooks are
  CLI-feature-specific; need to check portability across IDE
  extension / desktop app / web app per the env list in
  `rules/environment-rules.md`.
- **Option B — Inline the rules in CLAUDE.md.** Stop relying on
  pointers; embed the rules content directly. Pro: no judgment
  call, no hook. Con: re-inflates CLAUDE.md (Phase 1.2 just cut
  it); rules duplication across `rules/*.md` and `CLAUDE.md`
  creates a maintenance liability the marker system is designed
  to handle but doesn't fix the duplication itself.
- **Option C — Sentinel-file ritual.** First response in a session
  must include a specific acknowledgment token (e.g. "Rules
  re-read: coding-session + design-philosophy") that a hook
  validates and blocks the response if absent. Pro: forces the
  read. Con: ritual visible to Jamie every session; brittle if
  hook can't enforce.
- **Option D — Skill that wraps first-message handling.** A
  `/start` or auto-fired skill at session begin reads the rules
  and primes the session. Pro: explicit, debuggable. Con: requires
  Jamie to remember to run it, OR requires hook integration
  anyway — degrades to Option A.

*Implementation surfaces to touch (Option A — most likely pick).*

- `cc-template/.claude/settings.json` — add `UserPromptSubmit`
  hook config that runs on first message of session.
- A small shell/PowerShell script (cross-platform per
  `rules/environment-rules.md`) that detects first-message
  state and emits the rules content into the context channel
  the hook supports.
- Validate the hook works in VSCode extension, CLI, desktop app —
  per `update-config` skill notes, hooks are settings.json-based
  and harness-executed; need to confirm the harness fires
  `UserPromptSubmit` consistently across surfaces.
- Document the mechanism in `cc-template/CLAUDE.md` (e.g. a
  one-line note under Collaboration-rules explaining the hook
  handles the re-read so downstream consumers don't have to know).
- Decision needed: does the hook fire on every session start, or
  only on sessions that don't already have rules content in the
  initial context window? (Cheaper second option but harder to
  detect reliably.)

*Open sub-questions.* Whether hook enforcement should extend to
`/wind-down` rule 9 doc-coherence sweep (the rule the 2026-05-26
checkpoint-002 landing missed — same class of "instruction
present, judgment skipped" failure). Whether the hook should be
opt-in for downstream consumers (some projects may prefer the
judgment call) or default-on (matches the template's
"reliable behavior out of the box" stance). Whether
`environment-rules.md` needs a section on hook authoring
conventions if Option A lands.

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

