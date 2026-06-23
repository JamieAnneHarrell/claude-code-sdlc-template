---
description: Run an interactive product-visioning session to produce the next PRD (first or n-th) — settle what to build next and why, draw out the product's personality, and write a stand-alone DRAFT PRD. /onboard then decomposes it into the planning docs.
---

# /product-visioning

`/product-visioning` is the **strategic** counterpart to the tactical SDLC
loop — *what comes next, and why?* It runs at a milestone (after an MVP
ships, before a big feature, when planning v0.2.0 / v1.0.0 / a major-version
shift) as an interactive product-manager session.

Its **single job**: produce the **next** `docs/design/PRD-<slug>-NNN.md` — a
stand-alone PRD for the next *movement*. First PRD or ninety-ninth, the task
is the same: an interactive session (or a series of them) that probes for
what's next, draws out the product's **personality**, relitigates prior
decisions, sorts near-term from far, and writes the PRD.

It does **not** decompose the PRD or touch any other doc. `/onboard` does
that next — it reads the PRD and turns it into PRODUCT_VISION / REQUIREMENTS
/ ARCHITECTURE / PROJECT_PLAN / CLAUDE_CODE_PROMPTS, whether it's the first
PRD or a later movement. A *movement* is a strategic chunk opened this way;
bug fixes, cleanup, and release work are tactical and skip this command.

This command runs in the current working directory and writes files relative
to it.

**Required reading before Step 0:** read `rules/coding-session-rules.md` and
`rules/design-philosophy-rules.md` in full. Rule 4 (no unsolicited design
decisions) governs the session — it settles scope *with* Jamie, never for
her. Rule 10 applies if the session pins any version into the PRD.

---

## What it produces

One artifact: `docs/design/PRD-<slug>-NNN.md`, `status: DRAFT`. The PRD is
**stand-alone** — readable later without the prior PRD series — and captures
the movement's starting point, the bet behind it, its scope (in / out /
deferred), the **proposed PRODUCT_VISION revisions** `/onboard` will apply
when it decomposes, and the decisions made this session.

The session is **interactive and re-runnable**: one sitting or several, all
refining the same DRAFT PRD until it's agreed. Nothing else changes until you
run `/onboard`.

### Arguments

- `/product-visioning` — auto-detect per Step 0.
- `/product-visioning --movement "<name>"` — name the movement explicitly;
  otherwise Claude proposes one and confirms it.

---

## Step 0: Detect the target PRD

No prerequisite — this runs on a mature project (the common case) or a fresh
one (to produce the first PRD before `/onboard`).

1. **Slug.** Read the project `<slug>` from the `project:` field of the newest
   `docs/design/PRD-<slug>-*.md`, or from `docs/PRODUCT_VISION.md`. On a fresh
   project with neither, ask Jamie for the project name + slug as part of the
   session.
2. **Target PRD.** Glob `docs/design/PRD-<slug>-*.md` (sorted lexicographically;
   zero-padded NNN sorts numerically through 999):
   - **A `DRAFT` PRD exists** → you're refining it (a continued session). Keep
     its `NNN`.
   - **Newest is `ACTIVE` / `SUPERSEDED`** → open the next movement,
     `N = newest + 1`.
   - **No PRD** → this is the first PRD, `N = 001`.

---

## Step 1: Read context and sweep open work

Load what exists (skip absent files — a fresh project has little to read):

1. `docs/PRODUCT_VISION.md` — current strategy, personality, roadmap, enablers.
2. The latest `ACTIVE` `docs/design/PRD-<slug>-NNN.md` — the movement just
   completed (its scope + what it deferred).
3. `docs/PROJECT_PLAN.md` — confirm the current movement's phases are
   `(COMPLETE)` (a movement normally opens once the prior lands).
4. `docs/REQUIREMENTS.md` — open / unresolved requirements.
5. `docs/open-questions.md` — Deferred User Stories + Open Questions.
6. `docs/test-plans/phase-*-exit.md` — §5 deferred observations + Known
   Limitations punted out of a phase.
7. Prior `docs/design/design-review-checkpoint-*.md` — Notes / deferred
   findings flagged "revisit later."

Build a **swept inventory**: a deduplicated list of open items, each tagged
with its source. This is the raw material the session routes into or out of
the movement. On a fresh project there's nothing to sweep — the session is
greenfield.

---

## Step 2: Reconnaissance

State back a 4–6 bullet picture: where the product is (which movement just
landed; the milestone prompting this); the swept inventory grouped into
themes; the current strategic shape from PRODUCT_VISION; and what Jamie is
coming in wanting. Ask her to state the idea or note any uploaded material.
Don't proceed until she confirms.

---

## Step 3: The guided conversation

The heart of the command — interactive, using `AskUserQuestion` for genuine
forks and free-form discussion otherwise. Work through, in Jamie's framing:

- **What's next, and why.** Settle the movement's goal, the bet, who it
  serves. Group the idea + relevant swept items + opportunistic additions
  into one coherent direction.
- **Relitigate scope.** Decide each swept item's place — **in** this movement,
  **roadmap** (a future movement), or **out** (cut). Items move both ways:
  roadmap can come in, in-scope can drop out. A movement may contradict prior
  plans (a 1.x→2.x shift); name the shift plainly.
- **Personality & positioning.** Draw out the product's voice, who it's for,
  its market position — probe for it. This durable material becomes the
  proposed PRODUCT_VISION revisions so the conversation never restarts cold.
- **Near vs. far.** Sort the direction into this movement vs. later movements.

Routing happens **in dialogue** and is written straight into the PRD's scope
sections — there is no inline mark-up step.

---

## Step 4: Write (or refine) the DRAFT PRD

Write `docs/design/PRD-<slug>-NNN.md` (`status: DRAFT`) from the conversation,
using this template. On a re-run, refine the existing DRAFT in place.

````markdown
---
prd: NNN
movement: NNN
project: <slug>
created: YYYY-MM-DD
status: DRAFT
opened-by: product-visioning session YYYY-MM-DD
---

# PRD — Movement NNN: <movement name>

## Starting point / current state

<What exists now: the movement just landed, the product's current
capabilities, the baseline this movement departs from. Enough that the PRD
reads coherently on its own.>

## Why this movement (the bet)

<Goal + rationale — how it advances the product's thesis, who it serves, what
success looks like. May contradict prior movements; name the shift.>

## Scope

- **In:** <what this movement builds>
- **Out:** <explicitly excluded, and why>
- **Deferred → roadmap:** <items routed to a future movement>

## Proposed PRODUCT_VISION revisions

<What /onboard carries into docs/PRODUCT_VISION.md when it decomposes: voice /
personality / positioning changes, new or re-ordered roadmap movements,
enabler updates. On the first PRD this is the founding vision.>

## Decisions this session

- <decision — e.g. a constraint relaxed, an approach chosen>
````

If a section has nothing this session, write a one-line italic note rather
than omitting it. Do **not** edit `PRODUCT_VISION.md`, `PROJECT_PLAN.md`, or
any other doc — the PRD is the only output.

---

## Step 5: TODO and report

Rewrite `TODO.txt` so the first entry under "What's next" is:

```
- Review docs/design/PRD-<slug>-NNN.md. When it's agreed, run /onboard to
  decompose it (apply the vision revisions, archive the prior plan, author
  the new movement's PROJECT_PLAN + prompts). Re-run /product-visioning first
  if the PRD still needs work.
```

Carryovers preserved below. Report: the PRD path, the movement name, and that
`/onboard` next operationalizes it — the first PRD then routes through
`/bootstrap` before `/design-review`; a later movement goes straight to
`/design-review` after onboard. The DRAFT PRD is a created artifact —
`/wind-down` owns the commit handoff (rule 7); don't surface git here.

End with: "Movement NNN drafted in `docs/design/PRD-<slug>-NNN.md`. Review it;
when it's agreed, run `/onboard` to decompose it."

---

## Failure modes

- **A `DRAFT` PRD already exists at an unexpected number.** Surface it; confirm
  whether to keep refining it or (rare) start a different one.
- **Frontmatter `status:` is unexpected** (hand-edited, missing). Refuse and
  ask Jamie how to recover.
- **The current movement's PROJECT_PLAN phases aren't all `(COMPLETE)`.**
  Opening a new movement mid-flight is unusual; surface it and confirm (she
  may be deliberately pivoting).
- **The conversation drifts into tactical decomposition** (phases, prompts).
  Stop — that's `/onboard`'s job. Capture the *intent* in the PRD's scope;
  don't author phases here.

---

## What this command does NOT do

- Does not own a status comment in `CLAUDE.md`. Per-file PRD frontmatter
  (`DRAFT` / `ACTIVE` / `SUPERSEDED`) is the source of truth. No
  `PRODUCT-VISION-STATUS` banner.
- Does not decompose the PRD or edit `docs/PRODUCT_VISION.md`,
  `docs/PROJECT_PLAN.md`, `docs/CLAUDE_CODE_PROMPTS.md`, REQUIREMENTS, or
  ARCHITECTURE — `/onboard` does all of that from the PRD.
- Does not flip the PRD to `ACTIVE` or supersede a prior PRD — `/onboard` does
  that when it decomposes.
- Does not archive `PROJECT_PLAN.md` / `CLAUDE_CODE_PROMPTS.md` — `/onboard`
  archives them to `docs/project-plans/` when it opens the new movement.
- Does not edit `docs/design-decisions.md` / `docs/open-questions.md` — it
  reads them to sweep; decisions are recorded in the PRD and propagated when
  `/onboard` decomposes or at `/wind-down`.
- Does not run git commands (rule 7), tests (rule 8), or push to remotes.
- Does not run `/onboard`, `/design-review`, or `/wind-down` automatically —
  it hands off via `TODO.txt`.
