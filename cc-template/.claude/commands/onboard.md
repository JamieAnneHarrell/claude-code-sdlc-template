---
description: Onboard a new project from cc-template based on a design doc Jamie dropped into docs/design/.
---

# /onboard

Walk Jamie through configuring this project from the cc-template seed.
This is the **first** of three sequential configuration commands:

1. **`/onboard`** (this command) — *what are we building?* Produces
   the planning docs from the design intake.
2. **`/bootstrap`** — *how do I start coding?* Plans the dev
   environment. Hard prerequisite to Phase 0.
3. **`/deployment-plan`** — *how does this ship?* Optional and
   deferrable.

The output of `/onboard` is a working set of project documents —
REQUIREMENTS, ARCHITECTURE, PROJECT_PLAN, CLAUDE_CODE_PROMPTS — plus
the design-decisions seed and project-specific scope/test-tooling
sections in the rules files. After `/onboard` completes, Jamie can
review the docs at her own pace; `/bootstrap` is the next required
step before Phase 0 work begins.

This command runs in the current working directory and writes files
relative to it.

**Required reading before Step 0:** read `rules/coding-session-rules.md`
and `rules/design-philosophy-rules.md` in full. These are the universal
rules that govern this session — including the rule-7 commit handoff
format that the final step of onboarding depends on. Skipping them
leads to drift (e.g. surfacing a bash heredoc commit block in a
PowerShell session).

---

## Step 0: Verify the project is unconfigured

Read `CLAUDE.md` and grep for the status comment:
`<!-- ONBOARD-STATUS: UNCONFIGURED -->`.

- If the comment is `UNCONFIGURED`: proceed.
- If the comment is `COMPLETE <date>`: tell Jamie the project is
  already onboarded and ask whether she wants to re-run onboarding
  (which will overwrite generated docs). If yes, proceed; if no, exit.
- If the comment is missing entirely: surface the unexpected state and
  ask Jamie how to proceed. Do not assume.

The other two status comments (`BOOTSTRAP-STATUS`, `DEPLOYMENT-PLAN-STATUS`)
are not this command's concern — leave them as-is.

## Step 1: Verify a design intake exists

List files in `docs/design/`. Ignore any `README.md` in that folder
(it's the placeholder telling Jamie where to drop her design).

- If no design files found: stop and tell Jamie to drop her design
  doc(s) into `docs/design/` first, then re-run `/onboard`.
- If one or more design files found: read all of them in full. They
  are the source of truth for what we're building.

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
back to Jamie or in any artifact, follow the **Version freshness**
section in `rules/environment-rules.md` — surface the staleness
caveat to Jamie, offer to look up the current stable/LTS release,
wait for go-ahead before running `WebSearch`/`WebFetch`. This
applies even if the design doc names a version. The illustrative
versions below (e.g. "Python 3.12", "Node 20") are placeholders
only — never copy them as pins.

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

### `docs/PROJECT_PLAN.md`
Phased plan derived from architecture. Each phase has:
- Phase letter or number (A, B, C, … or 0, 1, 2, …)
- One-sentence goal
- Bullet list of deliverables
- Exit criteria (what "done" looks like)

Phase 0 is always project scaffolding. Phase 1+ are functional
deliverables. Final phase is launch hardening.

Open PROJECT_PLAN.md with a short orientation paragraph that
names the two recurring close-out commands:

> Phases close with `/exit-test-plan` (manual walkthrough against
> the phase's exit criteria) when the phase ships user-observable
> behavior. High-risk transitions between phases also get a
> `/design-review` checkpoint — see the markers below.

Phases that ship no user-observable surface (docs-only,
pure-infrastructure, pure-refactor) can skip the manual
walkthrough — don't force one. The orientation paragraph is
enough; per-phase exit-test-plan markers would be noise.

**Design-review checkpoints between phases.** For each high-risk
transition identified above (the same set used for
CLAUDE_CODE_PROMPTS.md checkpoint touchpoints), insert a one-line
checkpoint entry after the prior phase's exit criteria:

```
**Design review checkpoint:** before Phase N+1 begins. Run
`/design-review`.
```

PROJECT_PLAN.md is the at-a-glance phase queue, so the checkpoint
shows up alongside phase boundaries — Jamie sees the gate when she
scans the plan, separately from the prompt body.

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

**Design-review checkpoints between prompts.** Identify high-risk
transitions in the phase queue — schema/migrations, multi-tenancy,
auth, anything Jamie flagged in Step 3 as expensive-to-retrofit, or
the post-Phase-0 transition where scaffolding hands off to the first
functional phase. At each transition, plant a review touchpoint in
`docs/CLAUDE_CODE_PROMPTS.md` in one of two forms (default to the
header-block note; reserve first-class entries for transitions where
skipping the review would be expensive):

- **Header-block note** (default). Add a line at the top of the next
  prompt's body, before its first numbered scope item:

  ```
  **Before running this prompt:** this is a good place to run
  `/design-review` to surface findings against the architecture as
  it stands now.
  ```

- **First-class entry** (major transitions only). Insert a standalone
  block between two prompts:

  ```
  ## Design Review Checkpoint — pre-Prompt-N

  Run `/design-review`. Trigger: <one-line — e.g. "Pre-Prompt-1
  scrutiny: tenant scoping and migration ordering">.
  ```

Routine phases (UI polish, docs-only work) get no checkpoint. The
goal is gating the high-risk transitions, not every boundary.

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
rule from the global `~/.claude/CLAUDE.md`.

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
  post-onboarding sanity check (REQUIREMENTS / ARCHITECTURE
  coherence, dependency justifications, design-intake fidelity)
  before `/bootstrap`. Mark up the resulting checkpoint inline
  (one decision per AUDIT NOTE block), then re-run
  `/design-review` to walk dispositions. Stage 2 will ask whether
  to land the doc or open an addendum round."
- Second entry: "After the review lands, run `/bootstrap` to plan
  the dev environment before starting Phase 0."
- Third entry: "After `/bootstrap`, paste Prompt 0 from
  `docs/CLAUDE_CODE_PROMPTS.md` to run Phase 0."
- Leave Carryover and Deferred decisions empty (rule 9 wind-down
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

   > ✅ **Onboarded.** Planning docs are written. Next step is
   > `/design-review` for a post-onboarding sanity check, then
   > `/bootstrap` to plan the dev environment, then Prompt 0.
   > `/deployment-plan` is deferrable. At every phase close,
   > `/exit-test-plan` authors the manual walkthrough; Stage 2
   > lands dispositions once Jamie has run it.
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
rule 7's commit-handoff format:

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
- Which phase transitions got design-review checkpoints (in
  CLAUDE_CODE_PROMPTS.md and PROJECT_PLAN.md), and which form was
  used (header-block note vs first-class entry)
- What the next session should do: run `/design-review` for the
  post-onboarding sanity check, then `/bootstrap`, then Prompt 0
  from `CLAUDE_CODE_PROMPTS.md`. The first action in `TODO.txt` is
  the design review.
- Any open questions that surfaced during onboarding (write these into
  `docs/open-questions.md` as well)

End with: "Onboarding complete. Read `docs/PROJECT_PLAN.md` to
review the phase queue, then run `/design-review` for a
post-onboarding sanity check before `/bootstrap`. `/deployment-plan`
is deferrable — run it when test/prod planning is needed."

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
  plants checkpoint touchpoints in `docs/CLAUDE_CODE_PROMPTS.md`
  (header-block notes and/or first-class entries) and in
  `docs/PROJECT_PLAN.md` (one-line checkpoint entries), and seeds
  `TODO.txt` so the post-onboarding review runs first. It never
  writes a checkpoint file at
  `docs/design/design-review-checkpoint-NNN.md` — that's
  `/design-review`'s domain.
- Does not author exit test plans — `/exit-test-plan` does.
  `/onboard` notes the recurring convention in
  `docs/PROJECT_PLAN.md`'s opening paragraph; it never creates
  files in `docs/test-plans/` (the directory itself doesn't exist
  until `/exit-test-plan` first runs).
