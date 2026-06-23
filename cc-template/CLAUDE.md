# CLAUDE.md

> **Before you respond to the first user message of any session:** read
> `rules/coding-session-rules.md` and `rules/design-philosophy-rules.md`
> end-to-end — they are not loaded into context by default. Confirm you
> have read and understood them before proceeding.

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

<!-- CC-TEMPLATE-BLOCK: reading-order -->
## Reading order at session start

1. **This file** — quick orientation and reading list.
2. **`TODO.txt`** — gitignored session handoff. First entry is what
   we pick up next; the rest is the upcoming queue.
3. **`docs/PROJECT_PLAN.md`** — active phase queue.
4. **`docs/CLAUDE_CODE_PROMPTS.md`** — authoritative prompt for the
   current phase.
5. **`docs/design/`** — design intake plus any `/design-review`
   checkpoints in flight. A checkpoint with `AWAITING-DECISIONS`
   frontmatter means Stage 1 ran but Stage 2 hasn't walked
   dispositions yet (or there's an addendum round mid-iteration).
6. **`docs/test-plans/`** — if the current phase has a test plan in
   flight, it'll be `phase-NNN-exit.md` here. `AWAITING-DISPOSITIONS`
   means the phase is somewhere in the iterative test-and-polish
   loop (original run, dispositions, polish coding session, optional
   §6.N addendums); `LANDED` means every run log is PASS / SKIP,
   every Fix-now is verified, and the phase exit is closed.

**Product-vision convention check.** If this project is onboarded (a
`docs/PROJECT_PLAN.md` exists) but has no `docs/PRODUCT_VISION.md`, or a
`docs/design/PRD-<slug>-NNN.md` lacks a valid `status:`
(`DRAFT` / `ACTIVE` / `SUPERSEDED`) in its frontmatter, it predates the
`/product-visioning` convention. Suggest a `/product-visioning` session
to align the PRD and `PRODUCT_VISION.md` before proceeding with movement
work.
<!-- /CC-TEMPLATE-BLOCK -->

<!-- CC-TEMPLATE-BLOCK: collaboration-rules -->
## Collaboration rules

The two universal rule files are named in the directive at the top of
this file; read them end-to-end before responding.

- `rules/coding-session-rules.md` — the 10 standing rules.
- `rules/design-philosophy-rules.md` — design judgment framework.

**Read these when relevant to the current task:**

- `rules/project-rules.md` — project scope discipline.
- `rules/testing-rules.md` — test discipline.
- `rules/environment-rules.md` — cross-platform conventions.
- `rules/multi-agent-rules.md` — subagent use.

If Jamie says "rule 4" or "this is a rule 1 issue" mid-session, that's a
drift signal pointing at `rules/coding-session-rules.md`. Acknowledge,
correct course, move on.
<!-- /CC-TEMPLATE-BLOCK -->
