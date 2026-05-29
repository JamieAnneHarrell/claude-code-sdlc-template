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

#### Make the rules-read reliably happen — placement, content, enforcement, just-in-time loading

*Context.* Root `CLAUDE.md` Collaboration-rules section says:

> **MUST DO before responding to the first user message of any
> session.** Read these two files end-to-end...

In practice Claude regularly skips the re-read and goes straight to
the user's task. The failure has multiple compounding causes —
addressing any single one in isolation is unlikely to fix it:

- **CLAUDE.md summarizes the rules** (lists all 9 by topic, names
  KISS and progressive disclosure), so Claude treats the summary as
  sufficient priors. Story "Trim CLAUDE.md" addresses this surface
  directly.
- **The instruction is buried** — it's the third section of
  CLAUDE.md, after a status banner and a Reading-order list. By the
  time Claude reaches it, attention is already on the user's
  request.
- **Enforcement relies on Claude's judgment**, which is biased
  toward "respond now." The instruction names the failure mode but
  depends on the reader to overcome it.

For this template, the cost compounds: every downstream project
seeded from `cc-template/` inherits the same skip-prone behavior.
Reliable rules-read is foundational — without it the rest of the
collaboration-rules infrastructure is decorative.

*Proposed approach — four complementary directions (not mutually exclusive).*

**Direction 1 — Move the rules-read instruction to the top of
CLAUDE.md.** "STOP. Read `rules/coding-session-rules.md` and
`rules/design-philosophy-rules.md` end-to-end before responding."
First line, no preamble. Optionally extend with session-type clues —
if the user's message hints at commits, wind-down, or testing,
suggest additional rule files (lines up with Direction 4). *Pro:*
free; minimal mechanical change. *Con:* still relies on Claude's
judgment.

**Direction 2 — Strip rule summaries from CLAUDE.md, move
drift-phrases into the rule files themselves.** Don't tell Claude
what the rules *are* — tell Claude to read them. CLAUDE.md's
current hints ("the rule-7 commit handoff format," "the rule-4
simpler-alternative self-check") let Claude pattern-match as if
it had absorbed them. Move key phrases *into* the rule files so
reading the actual rule is the only path to those priors. Pairs
with Story "Trim CLAUDE.md." *Pro:* removes the "I already know
the rules" pattern-match. *Con:* unverified that this alone
changes behavior; Claude may still skip and rely on training.

**Direction 3 — Hooks-based enforcement.** A `UserPromptSubmit`
hook on the first message of a session reads both rules files into
context unconditionally, removing the judgment call. Hook detects
"first message of session" via session state (e.g. absence of
prior assistant turns, or a sentinel file written on first
invocation and cleared at session close). Ships as part of
`cc-template/`'s default `.claude/settings.json`. *Pro:* zero
reliance on Claude compliance. *Con:* hooks are CLI-feature-specific
— portability across IDE extension / desktop app / web app needs
validation per the env list in `rules/environment-rules.md`;
sentinel detection is non-trivial.

**Direction 4 — Rules co-located with skills; skills load rule
context just-in-time.** Move specific rules into the skill that
exercises them — e.g., rule 7 (commit handoff format and brevity)
lives with `/wind-down`, since `/wind-down` should be the only
command that ever surfaces a commit handoff (cross-refs Story
"Artifact-boundary command landings → `/wind-down`"). Skills load
on invocation, so the relevant rule context arrives fresh,
just-in-time, and doesn't compete for attention with other rules
at session start. Companion change: configure other commands/skills
NOT to surface commit handoffs — they defer to `/wind-down`. *Pro:*
rule context is loaded when relevant and not before; rules stay
short because they don't have to live in CLAUDE.md's permanent
budget. *Con:* not every rule has a canonical skill home (KISS,
rule 4 simpler-alternative are session-pervasive); requires a
per-rule audit.

**Direction 4b — Session entry-point via skill.** Extends
Direction 4: if a session begins without an obvious skill
invocation, Claude asks the user's intent and suggests the
appropriate skill ("Are we onboarding? Running a phase prompt?
Doing a design review? Winding down?"). The selected skill loads
its rule context. *Pro:* makes "the right rules for this work"
the default. *Con:* a friction step on every session start; risks
feeling bureaucratic; needs careful ergonomics (skip when intent
is obvious from the user's first message).

*Open sub-questions.* Per-rule audit for Direction 4: which rules
have a canonical skill home (rule 7 → `/wind-down`, rule 9 →
`/wind-down`, rule 8 manual walkthrough → `/exit-test-plan`) and
which stay session-pervasive (KISS, rules 1–6). Whether Direction
4 changes how `/design-review` and `/exit-test-plan` surface their
commit handoffs today (lines up with the artifact-boundary-commands
story — landing that one first is a prerequisite). Whether
Directions 1 + 2 should land first as a nearly-free baseline
before investing in 3 or 4. Whether Direction 3's hook should be
opt-in or default-on for downstream consumers (default-on matches
the template's "reliable behavior out of the box" stance). Whether
Direction 4b is annoying when the user's first message *is* the
intent ("read TODO and run Prompt 2.1") — needs an
intent-obvious bypass. Whether hook enforcement should extend to
`/wind-down` rule 9 doc-coherence sweep (the rule the 2026-05-26
checkpoint-002 landing missed — same class of "instruction
present, judgment skipped" failure). Whether
`environment-rules.md` needs a section on hook authoring
conventions if Direction 3 lands.

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

#### Trim CLAUDE.md — content that lives in TODO.txt or the rules files shouldn't be repeated here

*Context.* CLAUDE.md is the only file loaded into every session's
initial context, so every line in it is permanent weight on every
session start. Three drift modes have accumulated:

1. **Self-edited "next phase" banners.** Root CLAUDE.md currently
   opens with a status banner ("✅ Onboarded and bootstrapped.
   Ready for the next phase prompt in `docs/CLAUDE_CODE_PROMPTS.md`
   (Phase 1.1 — add `BLOCKED` disposition to `/exit-test-plan`)
   ..."). The banner duplicates TODO.txt, which CLAUDE.md *itself*
   names as the next-step source of truth. When TODO.txt updates
   and the banner doesn't, they disagree. As of 2026-05-27 the
   banner still names "Phase 1.1 — add BLOCKED" even though that
   landed three commits ago.

2. **Recurring-command cadence prose in `cc-template/CLAUDE.md`.**
   The template's banner describes when `/design-review` and
   `/exit-test-plan` run ("two recurring commands run on Jamie's
   cadence... post-onboarding sanity check; between phases that
   touch schema, multi-tenancy, auth..."). The cadence is the
   skill's own job to know — the consumer doesn't need to be told
   at session start. Context bloat that doesn't change session
   behavior.

3. **Rule summaries in the Collaboration-rules section.**
   CLAUDE.md lists each of the 9 rules by topic and gestures at
   KISS / progressive disclosure ("the rule-7 commit handoff
   format," "the rule-4 simpler-alternative self-check"). Claude
   (by its own admission, observed repeatedly) treats the summary
   as sufficient priors and skips the end-to-end re-read. Pairs
   with Story "Rules-read reliability."

*Proposed approach.* Treat CLAUDE.md as a thin index — not a state
report, not a rules summary.

- Remove the self-edited "next phase" banner from root CLAUDE.md;
  no command should write next-phase prose into CLAUDE.md. TODO.txt
  is authoritative for "what's next."
- Strip `cc-template/CLAUDE.md`'s recurring-command cadence prose.
  Skills describe themselves; CLAUDE.md doesn't need to.
- Replace the "9 rules summarized" section with a pointer only —
  "9 rules; read `coding-session-rules.md` end-to-end before
  responding." No topical list Claude can pattern-match as having
  absorbed.

*Open sub-questions.* This story is in tension with existing
"In-flight artifact status callout at top of CLAUDE.md" — that
story proposed automating a 🟡 status banner; this one argues
banners drift and TODO.txt already serves the purpose. Resolve
before either lands; probably retire the callout story in favor of
this one. Whether the configuration-ritual banner (status-comment-
driven, top of `cc-template/CLAUDE.md`) qualifies as bloat —
probably earns its place because it's the entry-point on a fresh
project, but worth a second look. Whether the "Reading order at
session start" section is itself bloat: it tells Claude to read
TODO.txt, PROJECT_PLAN, CLAUDE_CODE_PROMPTS — habits that might
belong in a hook or a skill (see Direction 3 / 4 in Story
"Rules-read reliability"), not in CLAUDE.md prose.

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

#### Skills that process human input shouldn't paraphrase-and-confirm each element back

*Context.* `/design-review` Stage 2's S2.2 walks marked AUDIT NOTE
blocks one at a time, restating Jamie's marking back and asking
for confirmation before appending each Disposition log row. With
16 findings in checkpoint 003 that produced 16 round-trips of "you
said X; I'll record X; confirm?" — busywork, because Jamie's
inline markings are already her settled position. She does her
thinking once, in the doc, before re-invoking. The friction is
the skill treating already-written input as if it needed
re-elicitation.

The same shape risks repeating in `/exit-test-plan` Stage 2, where
the spec also walks each TC's run-log entry, and in any future
skill that processes a batch of pre-written human decisions.

*Proposed approach.* Update the relevant skill specs (starting
with `/design-review` S2.2 and `/exit-test-plan` S2.2) to:

1. Scan all latest-round markings first and triage into
   *unambiguous* (record silently, in batch, in one write) vs
   *genuinely ambiguous* (surface that specific question).
2. Define "genuinely ambiguous" with explicit triggers — malformed
   marking, internally contradictory caveats, scope conflict
   between two findings, novel disposition shape not in the
   legend (until REJECTED lands as a first-class shape per the
   companion finding in checkpoint 003 Addendum 1).
3. Default to batching the recording into one Disposition log
   write rather than N confirmations. Surface only the
   genuinely-ambiguous items inline.

Generalizes as a design principle: *skills processing pre-written
human input read it as authoritative, not as a prompt to
re-elicit.* The human's act of writing the input IS the
confirmation.

*Open sub-questions.* Where exactly to draw the "ambiguous" line —
should the spec enumerate triggers (malformed line; caveat
unclear; cross-finding conflict) or trust the skill's judgment?
How does this interact with the analogous walk step in
`/exit-test-plan` Stage 2 (same friction risk over many TCs)?
Whether the triage step itself should be visible to Jamie ("I'll
batch-record N markings; pausing on M for clarification") or
silent until the questions surface — visible probably wins for
trust, but silent is leaner.

#### Add REJECTED to `/design-review`'s AUDIT-NOTE legend as a first-class disposition shape

*Context.* The current `/design-review` AUDIT-NOTE legend has four
shapes: Accepted / Accepted with caveats / Defer Approved /
DECISION. In checkpoint 003 Round 1, Jamie used a fifth shape —
**REJECTED** — five times (B2, R1, R3, R4, R6). The semantic
emerged organically: "none of the listed recommendations are
satisfactory; opens another addendum; the rejection prose is the
reframing input for Stage 1's addendum-branch authoring." It works
as session behavior but is not encoded in the spec — Step S2.1's
classification step would surface it as "malformed — surface and
ask" in a fresh project, blocking the round.

*Proposed approach.* Formalize REJECTED as a fifth legend item in
both copies of `/design-review` (`.claude/commands/design-review.md`
and `cc-template/.claude/commands/design-review.md`):

1. Step S1.5's checkpoint-template legend gains a fifth bullet:
   `REJECTED: <your reframing prose>`, with the definition above.
2. Step S2.1's classification list adds REJECTED as a fifth
   classification.
3. Step S2.5's land-or-addendum recommendation logic: any REJECTED
   in the latest round is an automatic recommend-open-another-round
   trigger.
4. Step S2.4's read-state explicitly foregrounds REJECTED count.
5. Step 0's placeholder check is unchanged — REJECTED is a
   non-placeholder marking like the other four.
6. Disposition log column value: record `REJECTED` verbatim.
7. Stage 1 addendum authoring (S1.A) convention: rejection prose
   is the authoritative reframe input — do not generate
   alternatives from scratch.

*Open sub-questions.* Whether REJECTED in earlier-but-not-latest
rounds should affect Stage 2's read-state or be treated as
historical record like other earlier-round dispositions (probably
the latter — supersession by latest round applies). Whether the
legend needs an "REJECTED with caveats" variant for rejections
that partially accept some sub-options (probably no — caveats
that change scope are already a DECISION shape; REJECTED with
caveats would be ambiguous). Whether `/exit-test-plan` should
gain an analogous REJECTED shape for its dispositions list
(probably yes for symmetry — though `/exit-test-plan`'s walk
disposes test-case run-log entries rather than finding
recommendations, so the analog is less direct).

