---
description: Decompose the next PRD into the project's planning docs. The first PRD configures a new project (from a dropped design doc or /product-visioning) and routes to /design-review then /bootstrap; a later movement's PRD updates the vision, archives the prior plan, opens the new movement, and routes straight to /design-review.
---

# /onboard

`/onboard` **decomposes the next PRD** into the project's planning docs. It
runs two ways, auto-detected (Step 0):

- **First PRD** — a new project. The design intake in `docs/design/` (a design
  doc Jamie dropped, or a `PRD-<slug>-001` from `/product-visioning`) becomes
  REQUIREMENTS, ARCHITECTURE, PRODUCT_VISION, PROJECT_PLAN, CLAUDE_CODE_PROMPTS,
  the design-decisions seed, and the project-specific rules sections. This is
  the first of three configuration commands: `/onboard` → `/bootstrap` (dev
  environment, prerequisite to Phase 0) → `/deployment-plan` (deferrable).
- **A later movement** — `/product-visioning` has produced a new PRD.
  `/onboard` applies its vision revisions, archives the prior movement's plan,
  and authors the new movement's PROJECT_PLAN + prompts. No re-setup; the next
  action is `/design-review`, not `/bootstrap`.

After onboard completes, Jamie reviews the docs at her own pace; the next action
is whatever Step 0's case routed to.

This command runs in the current working directory and writes files
relative to it.

**Required reading before Step 0:** read `rules/coding-session-rules.md`
and `rules/design-philosophy-rules.md` in full. These are the universal
rules that govern this session. The final step surfaces a commit
handoff, whose format is owned by `/wind-down`'s Step 4 (rule 7 routes
commit handoffs through `/wind-down`). Skipping these leads to drift
(e.g. surfacing a bash heredoc commit block in a PowerShell session).

---

## Step 0: Detect the case (first PRD or a later movement)

Read `CLAUDE.md`'s `<!-- ONBOARD-STATUS: ... -->`:

- **`UNCONFIGURED` → first PRD.** A new project. The design intake in
  `docs/design/` is the first PRD — a design doc Jamie dropped, or a
  `PRD-<slug>-001.md` from `/product-visioning`. Run the **First PRD** path
  (Steps 1–7). Next action after onboard: `/design-review` (the post-onboard
  sanity check that gates `/bootstrap`), then `/bootstrap`.
- **`COMPLETE <date>` → check for a later movement.** Read the project `<slug>`
  from `docs/PRODUCT_VISION.md` (or the newest PRD's `project:` field) and glob
  `docs/design/PRD-<slug>-*.md`. Compare the newest PRD's `movement:` to
  `docs/PROJECT_PLAN.md`'s `movement:` header:
  - Newest PRD's movement is **newer** (and `DRAFT` or `ACTIVE`-not-yet-decomposed)
    → run the **Decompose a later movement** path below.
  - Newest PRD matches the in-flight plan (already decomposed) → nothing to do;
    redirect: "This movement is already decomposed — run `/product-visioning` to
    plan the next, or `/design-review` to review the current." (To deliberately
    re-run first-PRD setup and overwrite, Jamie says so explicitly.)
- **Missing comment** → surface the unexpected state and ask. Don't assume.

The other two status comments (`BOOTSTRAP-STATUS`, `DEPLOYMENT-PLAN-STATUS`)
are not this command's concern — leave them as-is.

---

# First PRD — set up a new project (Steps 1–7)

Reached from Step 0 when `ONBOARD-STATUS` is `UNCONFIGURED`.

## Step 1: Verify a design intake exists

List files in `docs/design/`. Ignore any `README.md` in that folder
(it's the placeholder telling Jamie where to drop her design).

- If no design files found: stop and tell Jamie to drop her design
  doc(s) into `docs/design/` first, then re-run `/onboard`.
- If one or more design files found: read all of them in full. They
  are the source of truth for what we're building. One may be a
  `PRD-<slug>-001.md` from `/product-visioning` — that *is* the first
  PRD; treat it as the design intake.

## Step 2: Reconnaissance

Before asking Jamie anything, build a concise mental model from the
design doc(s):
- What is the project? (one-paragraph summary)
- What's the proposed architecture / stack?
- What does MVP scope look like in the design's own framing?
- What's roadmap / post-MVP per the design?
- What language and tooling does the design imply?

State this back to Jamie as a 4–6 bullet summary so she can confirm
you've understood the design correctly. Don't proceed until she
confirms.

## Step 3: Setup questions

Ask via `AskUserQuestion`, batched into 2–3 dialogs of 3–4 questions
each. Each question must be material — if the design already answers
it, don't re-ask. Cover at minimum:

**Version recency.** Before suggesting a version in any answer
back to Jamie or in any artifact, follow **Rule 10** (never assume
training-time versions are current) in
`rules/coding-session-rules.md` — surface the staleness caveat,
offer to look up the current stable/LTS, wait for go-ahead before
`WebSearch`/`WebFetch`. This applies even if the design doc names a
version: ask whether the pin was intentional or a training-time
residual before perpetuating it. The illustrative versions below
(e.g. "Python 3.12", "Node 20") are placeholders only — never copy
them as pins.

### Project identity
- **Project name** and short slug (e.g. "Nichestream Platform" /
  `nichestream`).
- **One-paragraph description** for `README.md` and `CLAUDE.md`.
- **License** (default MIT).

### Language and tooling
- **Primary language and version** (Python 3.12 / PHP 8.3 + Laravel
  11 / Node 20 + TypeScript / etc.).
- **Test runner** (pytest / phpunit / vitest / jest).
- **Formatter** (ruff format / black / prettier / php-cs-fixer).
- **Linter** (ruff check / eslint / phpstan).
- **Package manager** (pip / poetry / composer / npm / pnpm).

### Multi-agent mode
Pick one based on project shape. Show all three with the trade-off:

- **explore-only** — parallel Explore subagents for research; all
  implementation sequential and Jamie-driven. Recommended for solo
  CLI tools, single-developer projects, anything where coordination
  overhead would dominate.
- **explore-plus-plan** — adds a Plan subagent for design proposals
  alongside Explore. Recommended for non-trivial platform designs
  (multi-tenant, multi-module) where validating an approach before
  implementing is worth a round-trip.
- **parallel-worktrees** — spawn implementation agents in git
  worktrees for genuinely independent features. Recommended only when
  feature work is easily parallelizable (separate modules, separate
  UI features) and Jamie wants the throughput.

### Scope statements (Jamie's defaults, project may override)
- **No GUI in MVP?** (default yes — CLI/web only for v0.1)
- **No ML models in MVP?** (default yes — classical algorithms only)
- **Cross-platform?** (default yes — Windows primary, Linux/macOS
  supported)

### Repository
- **Git status** — existing repo, fresh init, or no git for now?
- **Remote** — GitHub repo planned? URL if known. (Just record; don't
  push.)

### Project-specific dependencies
List the runtime dependencies the design implies, with one-line
rationale per dep. Confirm the list with Jamie before writing it into
`rules/project-rules.md`.

### Secrets and credentials hygiene
If the design implies a database, a secrets store, or any persistent
credentials, an NFR for secrets hygiene gets written into
`docs/REQUIREMENTS.md` (see Step 4). No question to Jamie — this is
template policy, not preference. Note it in the Step 3 confirmation
("an NFR for secrets and credentials hygiene will be added because the
design uses a DB") so she sees it before write-time. If the design has
no DB and no credential storage, skip — don't add the NFR.

## Step 4: Generate documents

Generate each file. **If a file already exists with non-template
content, do not overwrite — surface a diff and ask Jamie.** The
template-shipped skeletons (`docs/design-decisions.md`,
`docs/open-questions.md`) are safe to overwrite if they're still the
shipped headers.

### `docs/REQUIREMENTS.md`
Extract from the design doc(s). Format as numbered functional
requirements (FR-1, FR-2, …) and non-functional requirements (NFR-1,
…). Each requirement is a single testable statement. If the design
doc already has a requirements section, lift it as-is and renumber if
needed.

**Conditional NFR — secrets and credentials hygiene.** If the design
implies a database, a secrets store, or any persistent credentials,
add this NFR verbatim (renumber to fit the existing NFR sequence):

> **NFR-X: Secrets and credentials hygiene.**
> - Application database users MUST NOT be `root`, `admin`, `sa`, or
>   any privileged default. Use a project-scoped name (e.g.
>   `<slug>_app`).
> - CI ephemeral database users MUST be a two-word-with-number
>   pattern (e.g. `swift-otter-42`) — never reuse the app username,
>   never reuse the project slug. The point is to leak no project
>   identity if the credential leaks.
> - Production database users follow the same scoped-name rule as
>   the app user.

If the design has no DB and no credential storage, skip this NFR.

### `docs/ARCHITECTURE.md`
Extract the architectural narrative from the design doc(s):
- High-level architecture (layers, components, data flow)
- Module / package layout
- Data model summary
- Key technical decisions with rationale
- What this architecture explicitly does NOT include

If the design doc is already in this shape, lift it directly with
section headers normalized.

### `docs/PRODUCT_VISION.md`
The strategic north star — slowly-changing, the counterpart to the
tactical PROJECT_PLAN. Extract from the design doc(s) the *why* and the
*where it's going* (not the phase-by-phase how). If the intake is a
`/product-visioning` PRD, its *Proposed PRODUCT_VISION revisions* section
is the founding vision — treat it as the primary source. Sections:
- **Vision / thesis** — the problem, the bet, why this product exists.
- **Product personality & positioning** — who it's for, the voice, the
  market position. This section **survives and evolves** across future
  `/product-visioning` sessions; write it so later sessions append or
  refine rather than restart. If the design doc carries little of this,
  capture what's there and leave a short "to deepen in product
  visioning" note.
- **Roadmap of movements** — the post-MVP direction from the design's
  roadmap / post-MVP framing, named as *movements* (strategic chunks),
  not phases. The MVP itself is movement 001.
- **Enablers** (optional) — anything cheap to honor now but expensive to
  retrofit later that the design flags; a foreclosure-watch list.

Vision/strategy lives here — keep PROJECT_PLAN purely tactical (phases +
exit criteria, no "long-term vision" section). This is the first
PRODUCT_VISION, built from the first PRD; each later movement updates it
when `/onboard` applies that PRD's proposed revisions (see "Decompose a
later movement"). `/product-visioning` proposes the changes; `/onboard`
writes them.

### `docs/PROJECT_PLAN.md`
Phased plan derived from architecture. Each phase has:
- Phase letter or number (A, B, C, … or 0, 1, 2, …)
- One-sentence goal
- Bullet list of deliverables
- Exit criteria (what "done" looks like)

Phase 0 is always project scaffolding. Phase 1+ are functional
deliverables. Final phase is launch hardening.

Open PROJECT_PLAN.md with YAML frontmatter naming the movement it
represents — `movement: 001` and `source-prd: PRD-<slug>-001` (the first
movement; `/onboard` reads this header to tell a later movement's PRD
from the one already decomposed) — then a short orientation paragraph
that names the two recurring close-out commands:

> Phases close with `/exit-test-plan` (manual walkthrough against
> the phase's exit criteria) when the phase ships user-observable
> behavior. High-risk transitions get their own design-review
> phase in the queue — a numbered phase whose deliverable is a
> landed `docs/design/design-review-checkpoint-NNN.md`. User-facing
> documentation is maintained by `/write-documentation` — run it when
> a movement ships features users interact with; it surveys what
> shipped and proposes the doc set, tracking currency per movement
> rather than gating each phase.

Phases that ship no user-observable surface (docs-only,
pure-infrastructure, pure-refactor) can skip the manual
walkthrough — don't force one. The orientation paragraph is
enough; per-phase exit-test-plan markers would be noise.

**Design-review phases.** For each high-risk transition identified
above, insert a dedicated design-review phase into the queue.
Design reviews are first-class numbered phases, not between-phase
markers. Each carries the standard phase shape:

```
## Phase N — Design review: <scope>

**Goal.** Surface findings against <what's being reviewed> before
<next phase> begins. Trigger: <one-line rationale>.

**Deliverables.**
- `docs/design/design-review-checkpoint-NNN.md` LANDED with
  dispositions walked.

**Exit criteria.** Checkpoint flips to `LANDED <date>`; all
Blocker findings have dispositions; any Recommendations that
require doc edits are applied or surfaced as TODO.
```

Numbering: design-review phases share the same minor-version
numbering as the surrounding phases. Inserting one between
Phase X.Y and Phase X.(Y+1) bumps the downstream phase to
Phase X.(Y+2) and so on within the same minor family — don't
introduce sub-numbering like X.Y.5. The corresponding prompt in
`docs/CLAUDE_CODE_PROMPTS.md` is numbered to match.

PROJECT_PLAN.md is the at-a-glance phase queue, so design reviews
show up as phases Jamie sees when she scans the plan.

### `docs/design/PRD-<slug>-001.md` — adopt the intake as the first PRD
The first PRD **is the design intake itself**, recorded under the PRD name
so later `/product-visioning` PRDs and onboard's movement decompositions
have a baseline. Don't author a separate PRD — adopt the intake:

- **Intake is already `PRD-<slug>-001.md`** (from `/product-visioning`):
  you decomposed it above — just flip its frontmatter
  `status: DRAFT → ACTIVE`.
- **Intake is a raw design doc** (e.g. `<name>-spec.md`): **rename it** to
  `docs/design/PRD-<slug>-001.md` and add the frontmatter below; its
  existing content is the PRD body — don't rewrite it. If several design
  docs were dropped, adopt the primary one and leave the rest as
  supporting intake in `docs/design/`.

Frontmatter (add for a raw design doc; confirm for a `/product-visioning`
PRD):

```
---
prd: 001
movement: 001
project: <slug>
created: <YYYY-MM-DD>
status: ACTIVE
opened-by: onboard (first PRD)
---
```

This `PRD-<slug>-001` becomes `SUPERSEDED` when the next movement opens.

### `docs/CLAUDE_CODE_PROMPTS.md`

**File-level header.** The file opens with a short header explaining
the deviation-footer convention. Include verbatim (the deviation
block below `# Claude Code prompts` is load-bearing — `/wind-down`
and any future tooling that touches prompt footers depend on it):

```
# Claude Code prompts

One prompt per phase. Paste the prompt body into a Claude Code
session to run that phase. Each prompt has a "revisions since this
prompt ran" footer where deviations from the original plan
accumulate during the actual coding session.

**The footer is for plan deviations only — never a recap of what
landed.** The git log is the recap; the prompt body is the plan;
the footer captures where the plan had to change. A clean execution
of the prompt leaves the footer empty except for the landing-date
marker. Example deviation: "Scope item 5 skipped — conditional on
§3 contract changing, which it didn't." Not a deviation: "Edited
file X to add Y" — that's commit-log material.
```

(Any project-specific intro paragraphs — historical-phase notes,
reading-order pointers — go after the deviation block.)

**Per-prompt structure.** One prompt per phase. Each prompt:
- Has a header `## Prompt N: <Phase Name>`
- States what to read first (REQUIREMENTS, ARCHITECTURE, PROJECT_PLAN)
- Numbered scope items for the phase
- Constraints (what NOT to do this phase)
- Exit criteria
- Ends with a "revisions since this prompt ran" footer (initially
  "none tracked"). Revisions accumulate *after* the prompt runs, so the
  block sits below the prompt body, not above it. Per the convention
  above, the footer is for plan deviations only — not a recap of what
  landed.

Pattern reference: this template's own
`docs/CLAUDE_CODE_PROMPTS.md` is the canonical shape.

**Design-review prompts.** Every design-review phase in
PROJECT_PLAN.md gets a matching numbered prompt entry in
`docs/CLAUDE_CODE_PROMPTS.md`, alongside the feature prompts.
Design reviews live in the prompt flow as real prompts — never as
"Before running this prompt" preambles attached to other prompts.

Prompt shape (matches PROJECT_PLAN.md phase numbering — Prompt 2.2
corresponds to Phase 2.2):

```
## Prompt N: Design review — <scope>

**Read first.**
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase N
- <any docs the review specifically scrutinizes>

**Scope.** Run `/design-review`. Trigger: <one-line — matches the
PROJECT_PLAN Phase N "Goal" line>.

**Exit criteria.** Checkpoint LANDED per Phase N exit criteria.

**Revisions since this prompt ran:** none tracked.
```

Routine phases (UI polish, docs-only work) get no design-review
phase and therefore no design-review prompt. The goal is gating
the high-risk transitions, not every boundary.

### `docs/design-decisions.md`
Already exists as a header skeleton. Add a first decision describing
the onboarding outputs: language choice, multi-agent mode, scope
statements. Format follows the auto-dailies pattern: **Decision** /
**Why** / **Why not (alternatives)**. If the secrets-hygiene NFR was
added to REQUIREMENTS.md above, mention it in the decision entry so
the rationale (no privileged defaults, no project-identity leakage in
CI users) is recorded.

### `docs/open-questions.md`
Already exists as a header skeleton. Add any genuinely-open questions
from the design doc that need answering before or during MVP work.
Categories: Deferred User Stories / Known Limitations / Abandoned
Approaches.

### `rules/multi-agent-rules.md`
Replace the placeholder with the variant Jamie chose:

- **explore-only:** strict scoping — Explore for research, no
  implementation agents.
- **explore-plus-plan:** Explore + Plan. State when each is
  appropriate.
- **parallel-worktrees:** Explore + Plan + worktree implementation
  agents. Document max concurrent (default 2), branch naming
  (`agent/<task>`), how Jamie merges back, what's out of scope (shared
  schema, evolving API contract, anything where two agents would step
  on each other).

All variants reference the "smart colleague who just walked in" briefing
rule from the global `~/.claude/CLAUDE.md`. **Preserve the
`briefing-rule` `CC-TEMPLATE-BLOCK` markers verbatim** when you rewrite
this file — it is the one refresh-managed block in
`multi-agent-rules.md`, and `/refresh-from-repository` relies on those
markers to track the block across updates. Wrap the chosen variant's
content; do not drop or relocate the marker pair.

### `rules/project-rules.md` (append)
Inside the `<!-- ONBOARD-FILL: project-scope -->` block, append:
- MVP scope statements from Jamie's answers
- Runtime dependency allowlist with rationales
- Out-of-scope-until-further-notice list

### `rules/environment-rules.md`
**Do not modify.** The `<!-- ONBOARD-FILL: environment -->` block
is filled by `/bootstrap` once the dev environment has been planned
(shell choice, venv path, build/run/test commands, tooling). Leave
the placeholder content untouched here.

### `rules/testing-rules.md` (append)
At the bottom, append project-specific section:
- Test runner and the marker syntax for integration tests
- Expected test counts (will fill in as suite grows)

Do not write canonical command lines here — those depend on shell
and venv decisions that `/bootstrap` makes. They live in
`rules/environment-rules.md`.

### `README.md`
Generate or rewrite as a **stub**. The Developer setup and
Deployment sections are written by `/bootstrap` and
`/deployment-plan` respectively — do not write placeholders that
look like instructions ("run npm install") since they'd be wrong
until the dev environment is actually planned.

Required sections (use these exact section names — `/bootstrap` and
`/deployment-plan` look for them):
- Project name + one-paragraph description
- A "Developer setup" section containing only: "Developer setup is
  generated by `/bootstrap`. Run that next before starting Phase 0."
- A "Deployment" section containing only: "Deployment plan is
  generated by `/deployment-plan` (deferrable; run when test/prod
  planning is needed)."
- Usage placeholder ("see Phase 1+")
- License (default MIT, link to LICENSE)
- Reference: "For development context, see CLAUDE.md and docs/."

### `TODO.txt`
The template ships a starter `TODO.txt` whose first entry is "Run
/onboard". Rewrite it after onboarding completes:
- First entry under "What's next": "Run `/design-review` for a
  post-onboarding sanity check (did REQUIREMENTS / ARCHITECTURE / the
  plan decompose cleanly from the PRD; dependency justifications;
  design-intake fidelity) before `/bootstrap`. Mark up the resulting
  checkpoint inline (one decision per AUDIT NOTE block), then re-run
  `/design-review` to walk dispositions. Stage 2 asks whether to land
  the doc or open an addendum round."
- Second entry: "After the review lands, run `/bootstrap` to plan the
  dev environment before starting Phase 0."
- Third entry: "After `/bootstrap`, paste Prompt 0 from
  `docs/CLAUDE_CODE_PROMPTS.md` to run Phase 0."
- Leave Carryover and Deferred decisions empty (`/wind-down`
  populates them as work accumulates).

### `.gitignore`
If no `.gitignore` exists yet, create from the template's seed.
Always includes: `TODO.txt`, OS junk, IDE junk, language-specific
patterns. Make sure `TODO.txt` is gitignored — it's a personal
handoff, not a tracked deliverable.

### `LICENSE`
If MIT (default) and no LICENSE exists, write standard MIT with
`<year>` set to current year and copyright holder set to Jamie's name
(ask if unknown).

## Step 5: Flip the status comment

Rewrite `CLAUDE.md`:
1. Replace `<!-- ONBOARD-STATUS: UNCONFIGURED -->` with
   `<!-- ONBOARD-STATUS: COMPLETE <YYYY-MM-DD> -->` using today's
   date. Leave `<!-- BOOTSTRAP-STATUS: UNCONFIGURED -->` and
   `<!-- DEPLOYMENT-PLAN-STATUS: UNCONFIGURED -->` untouched.
2. Replace the unconfigured 4-bullet banner block with a
   post-onboard banner:

   > ✅ **Onboarded.** Planning docs are written. See `TODO.txt`
   > for the current next step — it is the authoritative handoff,
   > rewritten at each session close.
   >
   > Before proceeding in any later session, check `docs/test-plans/`
   > and `docs/design/` for an in-flight test plan, review checkpoint,
   > or `DRAFT` PRD.
   >
   > **Project:** \<name\> — \<one-paragraph description\>
   > **Language / framework:** \<from Step 3\>
   > **Multi-agent mode:** \<from Step 3\>

   Do **not** write build/test commands or a Developer setup
   pointer in this banner — those are unknown until `/bootstrap`
   runs and would be wrong placeholders.

Keep the reading order, collaboration rules, and references
unchanged.

## Step 6: Surface git commands (rule 7)

Do not run any git commands. Surface a copy/paste-ready block per
the commit-handoff format in `/wind-down`'s Step 4:

```
git init
```

Show what to stage (note: `rules/environment-rules.md` is not yet
modified — `/bootstrap` will append to it later):

```
git add CLAUDE.md README.md LICENSE .gitignore TODO.txt rules/ docs/ .claude/
```

```
git status
```

Then a single-block commit:

```
git commit -m "Initial commit from cc-template

Onboarded for <project name>. See docs/PROJECT_PLAN.md for phase queue.
- Multi-agent mode: <mode>
- Language: <language>"
```

If GitHub remote was specified, surface the `git remote add origin` and
`git push -u origin main` commands separately. **Do not push** unless
Jamie explicitly asks in this same session.

## Step 7: Final report

Summarize for Jamie:
- Files generated (one bullet per file with one-line description)
- Multi-agent mode chosen
- Which phases are design-review phases (numbered in
  PROJECT_PLAN.md and CLAUDE_CODE_PROMPTS.md alongside the
  feature phases they precede)
- What the next session should do: run `/design-review` for the
  post-onboard sanity check, then `/bootstrap`, then Prompt 0
  from `CLAUDE_CODE_PROMPTS.md`. The first action in `TODO.txt` is
  the design review.
- Any open questions that surfaced during onboarding (write these into
  `docs/open-questions.md` as well)

End with: "Onboarding complete. Read `docs/PRODUCT_VISION.md` for the
strategy and `docs/PROJECT_PLAN.md` for the phase queue, then run
`/design-review` for a post-onboard sanity check before `/bootstrap`.
`/deployment-plan` is deferrable — run it when test/prod planning is
needed."

---

# Decompose a later movement (subsequent PRD)

Reached from Step 0 when an onboarded project has a newer undecomposed PRD
from `/product-visioning`. This applies that PRD **without** the one-time
first-PRD setup (no project identity, no `ONBOARD-FILL`, no `/bootstrap`). The
target PRD is the newest `docs/design/PRD-<slug>-NNN.md`.

## Step M1: Read the PRD and current state
Read the target PRD in full — its scope (in / out / deferred), its proposed
PRODUCT_VISION revisions, its decisions. Then read `docs/PRODUCT_VISION.md`,
`docs/REQUIREMENTS.md`, `docs/ARCHITECTURE.md`, and the in-flight
`docs/PROJECT_PLAN.md` (confirm its phases are `(COMPLETE)`; if not, surface it
— opening a movement over unfinished work is unusual; confirm before
proceeding). If the PRD still has unresolved scope (a section left as a
placeholder), stop and point back to `/product-visioning` to finish it.

## Step M2: Apply PRODUCT_VISION revisions
Apply the PRD's "Proposed PRODUCT_VISION revisions" to
`docs/PRODUCT_VISION.md`: evolve the personality & positioning section
(append/refine, never rewrite history), add or re-order roadmap movements,
update enablers. Edit the specific sections in place; state each edit as you
make it.

## Step M3: Apply REQUIREMENTS / ARCHITECTURE deltas
For what this movement changes, edit `docs/REQUIREMENTS.md` and
`docs/ARCHITECTURE.md` in place — append new FR/NFR (don't renumber existing),
adjust the affected architecture sections. A movement may be **redirective**
(a 2.x shift): revise the affected sections, not just append.

## Step M4: Archive the prior movement's plan
Move `docs/PROJECT_PLAN.md` → `docs/project-plans/project-plan-NNN.md` and
`docs/CLAUDE_CODE_PROMPTS.md` → `docs/project-plans/claude-code-prompts-NNN.md`,
where NNN is the **retiring** movement's ordinal (the in-flight plan's
`movement:` header), zero-padded to 3 digits. Create `docs/project-plans/` if
absent. Archived files are historical — never edited again.

## Step M5: Author the new movement's plan + prompts
Author fresh `docs/PROJECT_PLAN.md` and `docs/CLAUDE_CODE_PROMPTS.md` from the
PRD's in-scope items, using the same shapes as the first-PRD versions (Step 4's
`docs/PROJECT_PLAN.md` and `docs/CLAUDE_CODE_PROMPTS.md` sections):
- PROJECT_PLAN frontmatter `movement: NNN` + `source-prd: PRD-<slug>-NNN`;
  phases with goal / deliverables / falsifiable exit criteria; first-class
  design-review phases where the movement needs them; the orientation paragraph.
- One prompt per phase with the deviation-footer convention; a design-review
  prompt for each design-review phase.

The PRD's deferred / roadmap items do **not** go into this plan — they stay in
PRODUCT_VISION's roadmap (or `docs/open-questions.md`).

## Step M6: Activate the PRD; supersede the prior
Flip the target PRD's frontmatter `status: DRAFT → ACTIVE` (if DRAFT), and mark
the prior movement's PRD `status: ACTIVE → SUPERSEDED <date>`. The latest
`ACTIVE` PRD is the definitive scope for the current movement.

## Step M7: TODO, hand off, wind down
Rewrite `TODO.txt`'s first entry: "Run `/design-review` to review the movement
NNN decomposition before pasting Prompt <first> from
`docs/CLAUDE_CODE_PROMPTS.md`." (No `/bootstrap` — the dev environment is
already set up.) If `docs/published/` holds a documentation manifest, add a
one-line reminder below the first entry: "User docs are now stale for movement
NNN — run `/write-documentation` once this movement's user-facing work ships."
(A reminder, not ahead of `/design-review`.) Then invoke `/wind-down` for the
commit handoff (rule 7): the new plan + prompts, the archived pair, the updated
PRODUCT_VISION / REQUIREMENTS / ARCHITECTURE, and the PRD status flips land
together. Report the movement opened, what was archived, that `/design-review`
is next, and (if docs exist) that `/write-documentation` is due once the
movement ships user-facing changes.

End with: "Movement NNN decomposed — PRODUCT_VISION updated, the prior plan
archived to `docs/project-plans/`, and a fresh `docs/PROJECT_PLAN.md` +
`docs/CLAUDE_CODE_PROMPTS.md` authored. Run `/design-review` to review it."

---

## Failure modes

- **No design doc found.** Stop. Tell Jamie to drop one in
  `docs/design/`. Don't proceed.
- **Design doc is too thin to derive REQUIREMENTS / ARCHITECTURE.**
  Surface specifically what's missing and ask Jamie to either expand
  the design or accept a thin first draft that she'll iterate on.
  Don't make up requirements.
- **Jamie picks parallel-worktrees but the project is a single-module
  CLI tool.** Push back once: "Worktrees add coordination overhead
  with little throughput gain on a single-module project. Are you
  sure?" Respect her decision after the pushback.
- **Existing files would be overwritten.** Surface the conflict, ask
  per file. Never silently overwrite Jamie's work.
- **Onboarding interrupted mid-way.** The status comment is the
  source of truth. If `UNCONFIGURED` is still present, onboarding can
  safely re-run; if it's been flipped, treat the project as
  partially-configured and ask Jamie what state to recover to.
- **Movement decompose, but the PRD's scope is unfinished** (a
  placeholder section remains). Stop; point back to `/product-visioning`
  to finish the PRD. Don't decompose a half-written PRD.
- **Movement decompose, but the in-flight plan isn't all `(COMPLETE)`.**
  Surface it and confirm before archiving — opening a movement over
  unfinished phases is unusual (a deliberate pivot is possible).
- **`COMPLETE` with no newer PRD.** Nothing to decompose; redirect to
  `/product-visioning` (plan the next movement) or `/design-review`.

---

## What this command does NOT do

- Does not plan the dev environment — `/bootstrap` does. That
  includes shell choice, venv path, build/run/test commands,
  required tooling beyond language, and dev secrets layout.
- Does not plan deployment — `/deployment-plan` does. That
  includes dev/test/prod topology, deploy mechanism, secrets
  across envs, and CD posture.
- Does not write a Developer setup section in `README.md` —
  `/bootstrap` does. The README this command writes is a stub.
- Does not write canonical command lines into
  `rules/environment-rules.md` — `/bootstrap` does.
- Does not run any git commands (rule 7).
- Does not run any tests (rule 8).
- Does not install dependencies, create venvs, or run package
  managers. Phase 0 of the generated PROJECT_PLAN.md handles that.
- Does not push to remotes.
- Does not write CI workflows. Phase 0 handles that for the project.
- Does not delete files Jamie created.
- Does not run design reviews — `/design-review` does. `/onboard`
  plants design-review phases in `docs/PROJECT_PLAN.md` and
  matching numbered prompts in `docs/CLAUDE_CODE_PROMPTS.md`, and
  seeds `TODO.txt` so the post-onboarding review runs first. It
  never writes a checkpoint file at
  `docs/design/design-review-checkpoint-NNN.md` — that's
  `/design-review`'s domain.
- Does not run the product-visioning conversation — `/product-visioning`
  produces the PRD; `/onboard` decomposes it (first PRD or a later
  movement), which is what writes/updates `docs/PRODUCT_VISION.md` and
  authors the plan + prompts. `/onboard` does not author a PRD's *body*
  (that comes from the design intake or `/product-visioning`); it records
  the first PRD and flips PRD status as part of decomposing.
- Does not author exit test plans — `/exit-test-plan` does.
  `/onboard` notes the recurring convention in
  `docs/PROJECT_PLAN.md`'s opening paragraph; it never creates
  files in `docs/test-plans/` (the directory itself doesn't exist
  until `/exit-test-plan` first runs).
