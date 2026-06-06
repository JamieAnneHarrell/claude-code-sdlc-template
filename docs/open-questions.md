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

#### `/design-review` Step S1.1 should read `docs/open-questions.md` (Abandoned Approaches especially)

*Context.* Surfaced 2026-06-04 while authoring checkpoint 004's first
Abandoned Approaches entry (N3 — the hash/baseline/state-file
mechanism). `docs/open-questions.md`'s `### Abandoned Approaches`
section exists so that "someone (Claude or human) tries the same thing
again — the file reminds them why it didn't work." But the one command
that re-proposes design options — `/design-review` — never reads that
file. Step S1.1 "Read project context"
(`.claude/commands/design-review.md`) loads REQUIREMENTS,
ARCHITECTURE, PROJECT_PLAN, CLAUDE_CODE_PROMPTS, README, DEPLOYMENT,
the `docs/design/` intake, and prior checkpoints — but not
`docs/open-questions.md`. So a Stage 1 review can re-derive an
already-abandoned approach as a fresh Option A/B/C and nobody is
holding the record that says "tried, failed, here's why."

This is the durable cross-session enforcement of coding-session
**rule 3** (rejections are permanent) at the design level. The risk is
concrete and imminent: checkpoint 004's N3 writes the
hash/baseline/state-file mechanism into Abandoned Approaches precisely
so a future session does not re-derive it — and the next
`/design-review` on the refresh machinery is exactly the session that
would re-derive it if S1.1 still doesn't read that section.

*Proposed approach.* Add `docs/open-questions.md` to the Step S1.1
read list (after the design-intake / prior-checkpoint reads), with a
line directing the review to treat `### Abandoned Approaches` as
hard constraints — do not re-propose an abandoned approach as a live
option without explicitly noting it was abandoned and why the
calculus has changed (rule 3's "say so once with new information"
path). Recommended scope: read the **whole file**, not just the
Abandoned Approaches section — Deferred User Stories, Known
Limitations, and Open Questions are all live context for a review (a
review may surface a deferred story as now-live, or is the natural
place an Open Question gets answered), and "read
`docs/open-questions.md`" is a simpler instruction than "read one
section of it." Narrower alternative: load only the Abandoned
Approaches section (leaner, but cherry-picks and misses the other
three categories). Decide at build time.

Read-only — this does **not** change `/design-review`'s edit scope.
`docs/open-questions.md` stays owned by `/onboard` + `/wind-down`
(and `/exit-test-plan` Stage 2); design-review adds it as an *input*
only.

*Implementation surfaces to touch.*

- `.claude/commands/design-review.md` Step S1.1 — add
  `docs/open-questions.md` to the numbered read list, plus the
  Abandoned-Approaches-as-constraint note.
- Mirror to `cc-template/.claude/commands/design-review.md` per NFR-9.

*Open sub-questions.* Whether the addendum branch (S1.1 item 9) needs
the same read or whether the initial-branch read carries forward
within the same checkpoint loop. Whether the
Abandoned-Approaches-as-constraint note belongs in S1.1 (read time) or
S1.4 (finding-composition rules) — probably S1.1 so the constraint is
loaded before any option drafting begins.

#### `/design-review` should carry a security-review lens

*Context.* Surfaced 2026-06-04 during the Phase 2.1.A build. While
pinning the `/refresh-from-repository` design we found a real
attack surface the original design review (checkpoint 002) and the
reframe (checkpoint 004) both missed: the command imports executable
instructions (`.claude/commands/*.md`) from a network upstream, so a
compromised upstream could inject malicious directives. We pinned a
fetch→review→apply security gate for that command in-session (see
`design-decisions.md` "`/refresh-from-repository` reviews the download
before applying"), but the *general* gap remains — `/design-review`
has no standing prompt to consider common attack vectors when it
reviews a high-risk transition. A network-touching, self-modifying,
or credential-adjacent feature should get a security pass as a matter
of course, not because the reviewer happens to think of it.

*Proposed approach.* Add a security-review lens to `/design-review`
Stage 1 — a checklist the review applies when a transition touches an
attack surface: supply-chain / upstream-trust (does this import code or
instructions? from where? is there a review gate before execution?),
injection (does untrusted content reach a place that shapes Claude's
behavior or runs shell?), credential / secret surfaces, and
data-exfiltration paths. The lens produces findings in the normal
severity-tiered shape (likely Blockers for unguarded execution of
imported content). Keep it proportionate — KISS / progressive
disclosure: the lens fires when the trigger conditions are present, not
as boilerplate on every checkpoint (a pure-markdown refactor doesn't
need a threat model).

*Implementation surfaces to touch (when built).*

- `.claude/commands/design-review.md` Stage 1 — add the security lens
  with its trigger conditions to the finding-composition step.
- Mirror to `cc-template/.claude/commands/design-review.md` per NFR-9.

*Open sub-questions.* Whether the lens is a distinct numbered step or a
sub-checklist inside the existing finding pass. What the exact trigger
conditions are (network I/O, self-modification, credential handling,
untrusted input — and how the reviewer detects them). Whether it
warrants a short companion section in `rules/` (e.g. a security-posture
note) or lives entirely in the command spec. Whether `/exit-test-plan`
should gain an analogous "did we test the attack surface" prompt.

#### Encode the design-decisions ↔ abandoned-approaches hygiene rule in the commands

*Context.* Surfaced 2026-06-04 during the Phase 2.1.A Block 1
wind-down. When this session superseded the Option D refresh mechanism,
the first instinct was to leave the abandoned `design-decisions.md`
entries in place with "Superseded by …" tombstone notes. Jamie
corrected the rule: a **fully** abandoned decision is *moved out* to
`open-questions.md` § Abandoned Approaches (design-decisions is for
current decisions only — "why is it this way?"); a **partially**
superseded decision is *rewritten to its surviving content* in place;
neither is left with a tombstone note. Right now that rule lives only
in a session correction — the commands that maintain these docs
(`/wind-down` Step 3, `/design-review` Stage 2 landing) don't encode
it, so a fresh Claude (or a downstream consumer's Claude) would re-make
the tombstone-note mistake. This is the natural extension of the
existing "`/wind-down` owns the open-questions ↔ design-decisions
reconciliation" decision (which encoded the *move* of resolved
questions) to the design-decisions ↔ abandoned-approaches relationship.

*Proposed approach.* Add to `/wind-down` Step 3's design-decisions /
open-questions guidance (and `/design-review` Stage 2's land path,
which also writes `design-decisions.md`): when a decision is superseded
during the session, classify it — **fully abandoned** (the entire
substance is the dead mechanism) → move it to `open-questions.md`
§ Abandoned Approaches (what it was / why abandoned / what replaced it)
and remove it from `design-decisions.md`; **partially superseded** (the
core decision lives, a sub-mechanism changed) → rewrite to surviving
content in place. Never leave a "Superseded by …" tombstone note
lingering in `design-decisions.md`. Part of the full coherence sweep —
applies to entries that predate the session too.

*Implementation surfaces to touch.*

- `.claude/commands/wind-down.md` Step 3 — the `design-decisions.md`
  and `open-questions.md` subsections.
- `.claude/commands/design-review.md` Stage 2 land path.
- Mirror both to `cc-template/.claude/commands/` per NFR-9.

*Open sub-questions.* Whether the rule lives in `/wind-down` only or
also `/design-review` (probably both — both write `design-decisions.md`
at their boundaries). Whether "fully vs partially abandoned" needs
concrete criteria or stays a surfaced judgment call (likely judgment).
Whether the Abandoned Approaches entry is drafted from the removed
entry's content or composed fresh.

#### `/design-review` is iterative — mid-round `/wind-down` is optional, and the user should be told

*Context.* `/design-review` runs across rounds (Stage 1 initial →
Stage 2 walk → optional addendum rounds → land). Only two boundaries
surface a rule-7 commit handoff: Stage 1 *initial* (new checkpoint)
and Stage 2 *landing* (`LANDED`). Stage 1 *addendum* and Stage 2
*open-another-round* are mid-iteration — no handoff; pending changes
wind down at session close (the "commit handoffs only on artifact
boundaries" invariant). Nothing *stops* a user running `/wind-down`
between iterations, but it isn't necessary — and a user who doesn't
know the lifecycle may wind down (and commit) after every round out
of habit, fragmenting one logical checkpoint into many commits.

*Proposed approach.* Add a short advisory so the user knows
mid-iteration wind-down is optional, surfaced either in
`/design-review`'s mid-iteration output (the close of a Stage 1
addendum or a Stage 2 open-another-round) or in the skill's
description / README. A single line of the shape: "this checkpoint
is still in flight; you don't need to `/wind-down` until it lands —
run it now only if you're stopping for the session." Document-vs-skill
placement TBD.

*Open sub-questions.* Where the advisory lives — command output at
the mid-iteration boundary reaches the user at exactly the decision
moment but adds a line to every mid-round close; the skill description
or README is quieter but easier to miss. Whether `/exit-test-plan`
needs the same advisory — it has the identical mid-iteration shape
(addendum branch, open-another-round) and the same "only landing
surfaces a handoff" rule, so probably yes, as a paired change.

#### Parameterize maintainer identity (name / pronouns / initials) for the public repo

*Context.* The repo is public, and shipped files hardcode the
maintainer's identity: "Jamie" by name, "she/her" by pronoun, and
"JAH" as initials (e.g. the `/design-review` AUDIT-NOTE marker
`AUDIT NOTE — JAH:`). A consumer who seeds a project from
`cc-template/` inherits all of it — their rules talk about Jamie,
their design reviews want Jamie's initials. Identity should be
placeholder-driven, with `/onboard` collecting the consumer's name,
pronouns, and initials and substituting them.

*Proposed approach.* Replace hardcoded identity in shipped files
with placeholder tokens; have `/onboard` ask for name / pronouns /
initials and fill them, the same way it already fills `ONBOARD-FILL`
blocks. Scope spans every `cc-template/` file that names the
maintainer: `rules/*.md`, `.claude/commands/*.md`, `CLAUDE.md`,
`README.md`.

*Open sub-questions.* Placeholder syntax — reuse the `ONBOARD-FILL`
marker family, or inline tokens like `{{MAINTAINER}}` / `{{PRONOUN}}`
/ `{{INITIALS}}`? The initials are load-bearing: `AUDIT NOTE — JAH:`
is the string `/design-review` stage detection keys decisions on, so
parameterizing initials means auditing `design-review.md` (both
copies) and `CLAUDE.md`'s invariant wording in lockstep. Whether
`/refresh-from-repository` re-collects identity on update or treats
filled identity as consumer-owned (probably consumer-owned, like
`ONBOARD-FILL`). The fallback if a consumer skips the questions (a
neutral default like "the maintainer" / "they" / a two-letter
placeholder). Whether this gates public-launch readiness — it is
what makes the repo genuinely reusable rather than personalized, so
likely high-priority once public launch is on the table.

#### `/wind-down`'s coherence sweep is repo-wide, not session-scoped

*Context.* Standing intent (and memory) hold that `/wind-down`'s
doc-coherence sweep fixes pre-existing and unrelated drift too — not
just this session's diffs. But the spec contradicts that:
`.claude/commands/wind-down.md` scopes the sweep to "any docs the
calling skill doesn't own — run the full ritual below, **scoped to
what the session actually changed**." So a downstream Claude (or a
fresh maintainer Claude) reading the spec would sweep only
session-touched docs and leave stale cross-doc drift in place. The
intent is the opposite: the sweep should catch anything out of sync
repo-wide and surface it for resolution, regardless of whether this
session touched it, and pull it back into coherence.

*Proposed approach.* Rewrite the coherence-sweep step so it is
repo-wide: surface every doc-coherence drift the sweep finds and pull
stale items in, even in docs the session never opened. Reconcile the
artifact-boundary-invocation line ("scoped to what the session
actually changed") with the repo-wide stance — likely the
artifact-boundary path still runs the full repo-wide sweep, just
attributed to the calling skill's landing.

*Open sub-questions.* Surface-only vs auto-fix for *unrelated* drift
— the standing intent says the sweep "fixes" pre-existing drift, but
a repo-wide auto-fix every session is heavier and riskier than
surface-for-Jamie-to-resolve; the item's own wording ("surface to be
resolved") reads as surface-first. Likely default: fix obvious
mechanical drift inline, surface judgment-call drift for Jamie. The
scope/cost of a genuine repo-wide pass every session vs a bounded set
of tracking docs (REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN /
CLAUDE_CODE_PROMPTS / design-decisions / open-questions / CLAUDE.md /
command docs). How the repo-wide sweep interacts with the
artifact-boundary scoping note.

#### `/design-review` pins every pinnable decision; only truly-unknowable ones reach the coding session

*Context.* Coding sessions should be as fully specified as possible
after a design review. In practice, reviews sometimes leave a
decision open "for the coding session to decide" when it could have
been pinned at review time — lazy under-specification. This is
distinct from, and not licensed by, the "First-level decisions
primary" load-bearing invariant: that invariant legitimately defers
sub-decisions that *genuinely depend* on an unmade first-level A/B/C
pick, and its own text already routes those to "a separate finding in
the next addendum" — never to the coding session. The gap is the
*lazy* leak, not the dependent deferral.

The discipline, in three buckets. Every decision a review touches is
one of:

1. **Pinnable now** — no dependency blocks it; decidable at review
   time. *Pin it.* The default and the bulk; never punt to the coding
   session.
2. **Genuinely dependent** — sub-decisions that only make sense once
   a first-level pick lands, and complex enough to be load-bearing.
   *Open an addendum* to fully spec them once the pick is made;
   surface 1–2 sentence implications now. Not coding-session work.
3. **Truly unknowable until implementation** — resolvable only by
   running code (e.g. "does this API actually return X"). *Explicitly
   flag* as a coding-session decision **and** state the decision
   criteria so the session isn't improvising.

A decision may reach the coding session only if it is bucket 3, with
criteria attached.

*Proposed approach.* Refine `/design-review`'s finding-composition
step to encode the three buckets and the bar (pin by default;
addendum for dependent trees; coding-session only for bucket 3 with
criteria). Extend the "First-level decisions primary" invariant
wording to name the lazy-leak failure mode and the
addendum-as-resolution path. Lands as a **spec refinement, not a
checkpoint** — it clarifies and tightens the existing invariant's
addendum mechanism rather than reversing a contract (precedent: the
"`/wind-down` owns the open-questions ↔ design-decisions
reconciliation" decision landed without a checkpoint as "a root-cause
spec fix honoring an existing contract").

*Implementation surfaces to touch.*

- `.claude/commands/design-review.md` finding-composition step
  (Stage 1).
- The "First-level decisions primary" bullet in root `CLAUDE.md`
  § Load-bearing invariants.
- Mirror the command edit to
  `cc-template/.claude/commands/design-review.md` per NFR-9.

*Open sub-questions.* The bucket-1-vs-bucket-3 boundary is a judgment
call ("could this have been decided at review?") — whether it needs
sharper criteria or stays judgment (likely judgment, surfaced).
Whether the bucket-2 "suggest an addendum" prompt needs a complexity
threshold or fires whenever dependent sub-decisions are non-trivial.
Whether `/exit-test-plan` has an analogous "don't leave the tester
improvising" refinement.

#### `/refresh-from-repository` defers EOL handling to git instead of detecting it

*Context.* During the 2026-06-04 dogfood, `/refresh-from-repository`
spent real effort detecting CRLF-vs-LF differences between root
(CRLF) and `cc-template/` (LF), which surfaced as all-lines-changed
diffs that buried the actual content changes. EOL is a git concern —
`.gitattributes` normalizes it at commit time — so the command
shouldn't be doing line-ending forensics during a merge.

*Proposed approach.* Two coupled actions:

1. Normalize cross-tree EOL via `.gitattributes` so endings are
   handled at commit time (also clears the pre-existing root-CRLF /
   dist-LF drift).
2. `/refresh-from-repository` stops detecting CRLF-vs-LF: diff
   ignoring whitespace, defer to git normalization, handle EOL only
   at the end if at all. Only if git rules don't normalize **and**
   upstream/downstream genuinely differ in ending, ask once whether
   downstream keeps its local EOL or takes upstream's.

*Implementation surfaces to touch.*

- `.gitattributes` at repo root (new).
- `.claude/commands/refresh-from-repository.md` merge/diff step
  (both copies per NFR-9).

*Open sub-questions.* Whether `.gitattributes` alone removes the need
for any in-command EOL handling (likely yes for the common case),
leaving the ask-once path as a rare fallback. Whether the
whitespace-ignoring diff is the default for all of refresh's content
comparison or scoped to EOL only.

#### `/onboard` should author prescriptive prompts — self-contained exit criteria and contract-level scope items

*Context.* Surfaced 2026-06-06 from a downstream consumer (a
sales-tax/Odoo tool). The project's spec, discovery, and design
docs were solid, but the `docs/CLAUDE_CODE_PROMPTS.md` `/onboard`
produced was weakly specified: scope items gestured at FR numbers
without stating the contract, and **every exit-criteria line was a
bare reference — "Per PROJECT_PLAN Phase N."** After running the
first two prompts the consumer realized they were too thin to code
against without chasing references, and had Claude rewrite the whole
file. The rewrite is the evidence for what "prescriptive" means
here; four patterns separate the weak originals from the strong
revisions:

1. **Exit criteria became self-contained and testable.** Original:
   "Per PROJECT_PLAN Phase 2." Revised: a concrete acceptance
   paragraph naming the inputs (the committed fixtures), the required
   outputs, the exact abort behavior with its row-pointed message,
   and the determinism / bounded-memory invariants — something you
   could hand straight to `/exit-test-plan`. The prompt no longer
   forces a hop to PROJECT_PLAN to learn when the phase is done.
2. **Scope items spell out the contract, not just FR pointers.**
   Original: "Parse the rate file to `RateRow`s; active-row filter
   honoring `--as-of` (FR-PJ-1/2/6)." Revised: the full 9-column
   layout, which column carries the rate (col 4 Intrastate), the
   exact active predicate (`effective ≤ as-of ≤ expiration`), the
   sentinel (`29991231`), and the default (`--as-of` = today). FR /
   PRD numbers stay as cross-references, but the prompt is buildable
   without dereferencing them.
3. **"Read first" points at specific anchors.** Revised names the
   `design-decisions.md` entries by their decision title and the
   exact PRD / ARCHITECTURE subsections, not just the file names —
   so the coding session loads precisely the right context.
4. **Constraints name the trap and the why.** Revised constraints
   call out the specific failure each one prevents (the ×100
   decimal→percent conversion lives only in the emit phase — "prevents
   the 100× silent-rate error"; read boundary column 1 exactly, never
   substring-match `Z5`; cross-quarter rate/boundary pairing is a
   normal published state, don't abort).

Root cause in the spec: `onboard.md`'s `### docs/CLAUDE_CODE_PROMPTS.md`
"Per-prompt structure" lists "Exit criteria" as a bare bullet with no
demand that it be self-contained or testable, and the design-review
prompt template it models exit criteria on is itself a pure reference
("Checkpoint LANDED per Phase N exit criteria"). It then points at
*this* project's `docs/CLAUDE_CODE_PROMPTS.md` as "the canonical
shape" — so however thin that exemplar's exit criteria are, `/onboard`
reproduces thin prompts downstream.

*Proposed approach.* Tighten the "Per-prompt structure" requirements
for **feature** prompts in `onboard.md`:

- **Exit criteria must be self-contained and testable** — restate the
  acceptance bar inside the prompt (named inputs, required outputs /
  artifacts, expected abort behavior with the row-pointed message,
  any determinism / performance invariant), reading like an
  `/exit-test-plan` input. Cross-reference the PROJECT_PLAN phase
  exit, but never *only* reference it.
- **Scope items state the contract** — when a phase has a concrete
  data contract (column layout, predicate, sentinel, ID scheme,
  precedence rule), write it into the scope item; keep FR / PRD
  numbers as cross-references, not as the sole source.
- **Read-first names specific anchors** — the `design-decisions.md`
  entries by title and the PRD / ARCHITECTURE subsections, not just
  the file.
- **Constraints name the trap and the why** — the specific failure
  mode each constraint prevents.

The design-review prompt template is explicitly **out of scope** —
its exit criteria *is* the checkpoint landing ("Checkpoint LANDED per
Phase N"), which is correct, because the review's own boundary, not a
feature acceptance test, defines done. This tightening applies only
to feature prompts.

*Implementation surfaces to touch.*

- `cc-template/.claude/commands/onboard.md` § `docs/CLAUDE_CODE_PROMPTS.md`
  "Per-prompt structure" — add the four requirements above; scope
  them to feature prompts and exempt the design-review template.
- Root `.claude/commands/onboard.md` — mirror per NFR-9.
- This project's own `docs/CLAUDE_CODE_PROMPTS.md` — `onboard.md`
  cites it as "the canonical shape," so the exemplar must meet the
  raised bar (audit its feature-prompt exit criteria) or the citation
  propagates thin prompts.

*Open sub-questions.* The KISS / rule-4 tension: self-contained exit
criteria restate part of PROJECT_PLAN's phase exit inside the prompt,
creating two places that can drift. How much to restate vs reference,
and whether `/wind-down`'s coherence sweep should own keeping the two
in sync (the revised file accepted the duplication for
self-containment — name that trade in the spec). Whether "states the
contract" needs a concrete trigger (only when a phase has a
machine-checkable data contract?) or stays a judgment call so
docs-only / UI-polish phases don't get padded. Whether this raises
`/onboard`'s per-prompt authoring cost enough to want a worked
before/after example in `onboard.md` itself rather than only the
narrative requirements.

---

### Abandoned Approaches

#### `/refresh-from-repository` reconciliation via per-block hash + baseline + sidecar state file (Option D)

*What it was.* The first-built mechanism for
`/refresh-from-repository` (checkpoint 002, built 2026-06-03; never
shipped). Template-owned content was wrapped in `CC-TEMPLATE-BLOCK`
markers (that part survives); each block's content was hashed with
`git hash-object` (LF-normalized for cross-platform determinism); a
sidecar `.claude/claude-code-sdlc-template-refresh-state.md` held four
sections (Upstream baseline, Block hashes, Refresh logic version,
Upstream directives). Reconciliation was a **three-way** compare —
downstream-current / upstream-current / baseline-from-state-file —
matched by id, with the per-block hash as the change-classifier and the
baseline distinguishing "block the consumer deleted" from "block that
never existed yet." Source mode used a subtree content-hash of the
local `cc-template/` as the baseline.

*Why it was abandoned (checkpoint 004 B1, 2026-06-04).* Patching the
seed path surfaced that the whole bug class lived in the storage
mechanism itself. The baseline was seeded from the wrong side
(downstream-current) in both the re-seed branch and the pre-marker
migration, so a pre-existing consumer edit would freeze into the
baseline and the *next* run would misclassify it as "downstream
untouched, upstream changed" and silently revert it. Two observations
dissolved the class: (1) the memory of "this block was deleted /
customized on purpose" can live **in the rules file itself** as a
marker state (a tombstone or a `forked` flag), set by asking once — no
sidecar baseline needed; (2) the *merge* is already the executing
session's job, so the per-block hash was only doing **classification**,
which a two-way compare plus a one-time question does without any stored
baseline. Following that thread removed the state file, the per-block
hashes, the `git hash-object` recipe, the `last-synced` reference, and
the baseline — leaving markers + the command's version stamp + the LLM.

*What replaced it.* Option A — stateless marker-state + ask-once. See
`design-decisions.md` "`/refresh-from-repository` reconciliation:
Option A". The marker syntax and coarse-grained wrapping from checkpoint
002 carried forward; the hash/baseline/state-file machinery did not.

*The accepted trade (so it is not re-litigated).* Option A is weaker
than Option D in exactly one place: **provenance**. Option D could infer
"who changed this block" from a stored baseline; Option A asks the
consumer once and records the answer in the marker, relying on the
consumer recognizing their own edit (possibly months later), with "take
upstream" the git-recoverable safe default. Jamie accepted this trade
2026-06-04. The determinism Option D preserved was **not** judged worth
its correctness-and-maintenance surface: keeping seeded/shipped hashes
correct, cross-platform LF/`hash-object` discipline, committing the
state file so the baseline survives a clean clone, and the recurring
drift risk across all of it. **Do not re-propose a stored-baseline /
per-block-hash mechanism** without new information that changes this
calculus (rule 3).

*Sibling mechanisms also considered and rejected* (at checkpoint 002,
when Option D was chosen; none revived by Option A). **Per-block-hash
alone** and **file-level-baseline alone** — each leaves
deleted-vs-never-existed ambiguous (Option A resolves that by asking
once). **git-3-way against tagged upstream releases** — forces
release-tag discipline on a one-maintainer upstream and exposes git
conflict-marker syntax to consumers who read these `.md` files every
session; Option A keeps all conflict resolution inside the running
session with no markers in the files.

---

### Open Questions

#### Portability audit — what makes *this* project work better than a freshly-seeded one?

*Context.* Consumers run the template in their own environments,
seeded only from `cc-template/` via `/onboard`. But this project also
benefits from things that **don't** ship: root `CLAUDE.md`'s
Load-bearing invariants and project context (source-only), the global
`~/.claude/CLAUDE.md` (Jamie's TODO.txt workflow, design philosophy,
migration pointers — machine-scoped), and project memory under
`~/.claude/projects/.../memory/`. Some of that is load-bearing for how
well this project runs. A seeded project gets none of it. The
question: which of those behaviors are doing real work, and which
should move into a *shipped* surface (`project-rules.md` or a skill)
so consumers inherit them?

*What we know so far.* `/onboard` seeds from `cc-template/`; root
`CLAUDE.md` invariants + project context are source-only and never
copied. Global `~/.claude/CLAUDE.md` (TODO.txt workflow,
progressive-disclosure philosophy) applies on Jamie's machine to
*every* project, so a consumer on another machine never sees it.
Project memory is local auto-memory, not shipped. The memories that
shape behavior (e.g. the repo-wide wind-down sweep, the
distributable-self-contained habit) are candidates for promotion to
shipped rules — some are already being promoted (the
self-containment habit lands as a project rule in
`rules/project-rules.md`; the repo-wide sweep is its own story
above).

*Nested sub-question — should `project-rules.md` be an always-loaded
read?* Today the session-start directive at the top of `CLAUDE.md`
mandates only `coding-session-rules.md` and `design-philosophy-rules.md`
end-to-end; `project-rules.md` is "read when relevant." If
project-scope discipline is load-bearing enough to belong in every
session's working memory, it should join the always-loaded set.
Trade-off: more mandatory reading at session start vs. relying on
just-in-time relevance. Resolve as part of the same audit.

*What would unblock an answer.* A deliberate pass that diffs root
`CLAUDE.md` + global/project memory against what a freshly-seeded
project receives, classifying each behavior as (a) correctly
source-only / machine-local, (b) should-be-promoted to a shipped
rule, or (c) should-be-promoted into a skill. The promotion
candidates then become their own stories.

#### One-line `CLAUDE.md` guardrail: "Claude never touches `CC-TEMPLATE-BLOCK` markers"

*Context.* During the `/refresh-from-repository` dogfood, the plan
tried to insert verbose marker-state encoding prose into `CLAUDE.md`
to pin the markers. Rejected — too verbose, and `CLAUDE.md` loads
into *every* session's context even when markers are irrelevant. That
*encoding-pin* form was decided against on 2026-06-04
("`CC-TEMPLATE-BLOCK` marker encoding lives with the skill + design
docs, not `CLAUDE.md`"), which also declined even a one-line
*encoding* pointer.

But the proposal here is a **different artifact**: not the byte-level
encoding, but a one-line **behavioral hands-off guardrail** — e.g.
*"Claude never edits `CC-TEMPLATE-BLOCK` HTML-comment markers; they
belong to the `/refresh-from-repository` skill."* That's a "don't
touch this" tripwire (the kind `CLAUDE.md` § Load-bearing invariants
already carries), not an encoding spec. Whether the behavioral
framing clears the bar the encoding-pin didn't is genuinely open.

*What we know so far.* The encoding-pin was declined for drift +
every-context-weight reasons; its scope note set a re-open trigger:
"Revisit the tripwire-pointer option if a future hand-edit actually
mangles a marker in practice." No marker has been mangled yet. The
behavioral guardrail is new information relative to the rejection
(rule 3's "say so once with new information" path), because its
*purpose* differs (prevent accidental edits, vs. document the
encoding). Counter-argument: a hands-off line still adds permanent
every-session weight for a risk that hasn't materialized, and the
markers are fairly self-evident inline.

*What would unblock an answer.* The scope-note trigger firing (a real
hand-edit mangles a marker), or a decision that the one-line
behavioral guardrail earns its place preemptively as cheap insurance
against a foreseeable mistake. If it lands, the natural home is
`CLAUDE.md` § Load-bearing invariants as a single tripwire line, not
an encoding block.

