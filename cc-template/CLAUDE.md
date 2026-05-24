# CLAUDE.md

<!-- ONBOARD-STATUS: UNCONFIGURED -->
<!-- BOOTSTRAP-STATUS: UNCONFIGURED -->
<!-- DEPLOYMENT-PLAN-STATUS: UNCONFIGURED -->

> ⚠️  **This project is unconfigured.** Configuration is a three-step
> ritual; each command flips its own status comment above and rewrites
> the relevant section of this banner. If you're working on this project
> and this isn't an /onboard directive, stop NOW and ask for confirmation.
>
> 1. Confirm a design doc exists in `docs/design/` (Jamie drops it there).
> 2. Run `/onboard` — produces `REQUIREMENTS.md`, `ARCHITECTURE.md`,
>    `PROJECT_PLAN.md`, `CLAUDE_CODE_PROMPTS.md`, and project-specific
>    rule sections from the design intake. Answers *what are we
>    building*.
> 3. Run `/bootstrap` — plans the developer environment (shell, stack
>    bootstrap, tooling, dev secrets) and writes a working Developer
>    setup section in `README.md`. Hard prerequisite to Phase 0.
>    Answers *how do I start coding*.
> 4. Run `/deployment-plan` when ready — plans dev/test/prod topology,
>    deploy mechanism, and CD strategy. Produces `docs/DEPLOYMENT.md`.
>    Deferrable; ideally run after some dev experience accumulates.
>    Answers *how does this ship*.
>
> If Jamie asks to start coding before `/onboard` and `/bootstrap` have
> run, remind her that those steps produce the docs subsequent sessions
> depend on. She may have a reason to skip; surface it once and respect
> her decision. `/deployment-plan` can always be deferred without
> blocking work.
>
> After configuration, two recurring commands run on Jamie's cadence —
> neither is part of the configuration ritual and neither owns a
> status comment in this file:
>
> - `/design-review` produces sign-off-ready checkpoints at high-risk
>   transitions (post-onboarding sanity check; between phases that
>   touch schema, multi-tenancy, auth, or anything expensive-to-
>   retrofit).
> - `/exit-test-plan` authors and closes out the manual walkthrough
>   that closes each phase: Stage 1 writes the plan from the phase's
>   exit criteria (or appends a §6.N polish addendum after polish
>   lands); Stage 2 reads the newest filled run log + trailing notes,
>   lands dispositions across the project's docs, and asks Jamie
>   whether to land the document or open another polish round.

## Reading order at session start (configured projects)

1. **This file** — quick orientation and reading list
2. **`TODO.txt`** — gitignored session handoff. First entry is what we pick
   up next; the rest is the upcoming queue.
3. **`docs/PROJECT_PLAN.md`** — active phase queue.
4. **`docs/CLAUDE_CODE_PROMPTS.md`** — authoritative prompt for the current
   phase.
5. **`docs/test-plans/`** — if the current phase has a test plan in flight,
   it'll be `phase-NNN-exit.md` here. `AWAITING-DISPOSITIONS` means the
   phase is somewhere in the iterative test-and-polish loop (original
   run, dispositions, polish coding session, optional §6.N addendums);
   `LANDED` means every run log is PASS / SKIP, every Fix-now is
   verified, and the phase exit is closed.

## Collaboration rules

**Read these every session, before any work:**

- `rules/coding-session-rules.md` — the 9 standing rules (root-cause,
  trust diagnosis, rejections permanent, no unsolicited design, decouple
  data from display, reference rule numbers, Jamie runs commits, Jamie
  runs tests, session wind-down rewrites TODO.txt). Includes the rule-7
  commit handoff format.
- `rules/design-philosophy-rules.md` — KISS, progressive disclosure,
  simple defaults. *iPhone, not Android. Macintosh, not PC.*

**Read these when relevant to the current task:**

- `rules/project-rules.md` — project-specific scope discipline (what is
  MVP, what is roadmap, dependency justification).
- `rules/testing-rules.md` — test discipline, language-neutral.
- `rules/environment-rules.md` — cross-platform conventions, shells,
  venv handling, where Claude scratch files go. Project-specific
  environment is filled in by `/bootstrap`.
- `rules/multi-agent-rules.md` — how this project uses subagents
  (filled in by `/onboard` based on the chosen mode).

If Jamie says "rule 4" or "this is a rule 1 issue" mid-session, that's a
drift signal pointing at `rules/coding-session-rules.md`. Acknowledge,
correct course, move on.
