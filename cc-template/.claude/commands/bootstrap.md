---
description: Plan the developer environment for an onboarded project — shell, stack bootstrap, tooling, dev secrets — and produce a working README Developer setup section.
---

# /bootstrap

The **second** of three sequential configuration commands.
Prerequisite: `/onboard` has run and produced the planning docs.

`/bootstrap` answers *how do I start coding?* It reads the planning
docs and design intake, asks tailored questions about the dev
environment for this stack, and produces:

- A **Developer setup** section in `README.md` that any new
  developer (or future-Jamie on a new machine) can follow to go
  from `git clone` to a running app and a passing test command,
  including a `### Verify your setup` subsection of one-line
  commands proving each required tool is on PATH.
- An append into `rules/environment-rules.md` capturing language
  version, shell expectation, venv/activation, build/run/test
  commands, and tooling beyond the language runtime.
- A `BOOTSTRAP-STATUS: INSTRUCTIONS-WRITTEN <date>` flip in
  `CLAUDE.md` after stage 1; a flip to `COMPLETE <date>` after
  stage 2 verifies the tools are actually on PATH.

After stage 2 succeeds, the project is ready for Phase 0.

This command runs in the current working directory and writes files
relative to it.

**Required reading before Step 0:** read
`rules/coding-session-rules.md`, `rules/design-philosophy-rules.md`,
and `rules/environment-rules.md` in full. The first two are the
universal rules; the commit-handoff format Step 7 depends on lives
in `/wind-down`'s Step 4 (rule 7 routes commit handoffs through
`/wind-down`). The third is what this command appends into — you
need to know the existing structure to add to it cleanly.

---

## Two stages

`/bootstrap` runs in two stages keyed off the current
`BOOTSTRAP-STATUS`:

- **Stage 1 — write instructions** (`UNCONFIGURED`). The bulk of
  this command. Asks tailored questions, writes the README
  Developer setup section and the environment-rules append, flips
  status to `INSTRUCTIONS-WRITTEN`. Steps 0–8 below cover stage 1.
- **Stage 2 — validate** (`INSTRUCTIONS-WRITTEN`). After the
  developer follows the README and installs the listed tools,
  re-running `/bootstrap` enters stage 2: it runs the
  `### Verify your setup` commands from the README, and on success
  flips status to `COMPLETE`. See "Stage 2: Validate" near the end
  of this file.

Stage 1 produces instructions; stage 2 confirms the developer
followed them. Phase 0 expects status `COMPLETE`.

---

## Step 0: Verify prerequisites

Read `CLAUDE.md` and check the three status comments:

- `<!-- ONBOARD-STATUS: ... -->` must be `COMPLETE <date>`. If it's
  `UNCONFIGURED`, refuse: "Bootstrap requires `/onboard` to have run
  first. Run `/onboard`, then come back." If it's missing, surface
  the unexpected state and ask Jamie.

Then check `docs/design/` for any
`design-review-checkpoint-*.md` file. Three cases, each a soft
recommendation (do not refuse):

- **No checkpoint file at all.** The post-onboarding sanity-check
  review was skipped. Surface: "No design review has run yet. The
  recommended sequence after `/onboard` is `/design-review` for a
  post-onboarding sanity check before `/bootstrap`. Run
  `/design-review` first, or push ahead with `/bootstrap`?" Wait
  for Jamie's choice.
- **Newest is `AWAITING-DECISIONS`.** Surface: "A design review is
  awaiting your decisions at
  `docs/design/design-review-checkpoint-NNN.md`. The recommended
  sequence is: mark it up, rerun `/design-review` to land it, then
  run `/bootstrap`. Continue with `/bootstrap` anyway, or pause and
  mark up the review first?" Wait for Jamie's choice.
- **Newest is `LANDED`.** Proceed silently — the review has been
  done.

If Jamie opts to push ahead in either soft-recommendation case,
proceed without further mention.
- `<!-- BOOTSTRAP-STATUS: ... -->` selects the stage:
  - `UNCONFIGURED` → stage 1. Continue with this command from
    Step 1.
  - `INSTRUCTIONS-WRITTEN <date>` → stage 2. Skip Steps 1–8 and
    jump to "Stage 2: Validate" below.
  - `COMPLETE <date>` → already validated. Tell Jamie the project
    is already bootstrapped and ask whether to re-run stage 1
    (which will overwrite the README Developer setup section and
    the environment-rules append block, and downgrade status to
    `INSTRUCTIONS-WRITTEN` so stage 2 needs to run again). Proceed
    only if she confirms.
- `<!-- DEPLOYMENT-PLAN-STATUS: ... -->` is not this command's
  concern — leave it as-is.

## Step 1: Read project context

Load these in order to build a stack-aware mental model:

1. `docs/REQUIREMENTS.md`
2. `docs/ARCHITECTURE.md`
3. `docs/PROJECT_PLAN.md` — especially Phase 0
4. Every file in `docs/design/` (other than the folder's own
   `README.md`)
5. The current `rules/environment-rules.md` — specifically the
   `<!-- ONBOARD-FILL: environment -->` block, so you know exactly
   where you'll be appending

Do not start asking questions yet.

## Step 2: Reconnaissance

State back to Jamie a 3–4 bullet summary covering:
- What stack the design implies (language already known from
  onboarding; what runtime services it needs around the language —
  DB, web server, queue, container engine, etc.)
- What the dev environment for this stack typically looks like, and
  what's specific to this project's constraints
- What's ambiguous in the design — call out the genuine open
  questions that the question batches in Step 3 will need to resolve

Don't proceed until Jamie confirms the picture.

## Step 3: Tailored questions

Ask via `AskUserQuestion`, batched into 2–3 dialogs of 3–4
questions each. **Tailor questions to the stack from Step 2** —
don't ask questions whose answer the design doc already gives, and
don't ask irrelevant questions.

**Version recency.** Before suggesting a version in any answer
back to Jamie or in any artifact, follow **Rule 10** (never assume
training-time versions are current) in
`rules/coding-session-rules.md` — surface the staleness caveat,
offer to look up current stable/LTS, wait for go-ahead before
`WebSearch`/`WebFetch`. This applies even if `/onboard`, the design
doc, or `docs/REQUIREMENTS.md` already named a version: ask whether
the pin was intentional or a training-time residual before
perpetuating it. Step 5 below re-checks at write time; this Step 3
check is the first gate, not the only one.

Examples (Claude picks what's relevant; this is not a checklist to
ask in full):

- **Python projects:** venv path (`.venv/`?), package manager (pip /
  poetry / uv), how Jamie activates the venv on Windows
- **PHP / LAMP / WAMP:** which local stack (XAMPP, WAMP, Laravel
  Herd, Docker compose), DB engine and version, where Apache /
  nginx config lives, whether to use WSL2 or native Windows
- **Node / web:** package manager (npm / pnpm / yarn / bun), port
  assignments, whether a backend service is bundled, frontend
  build pipeline
- **Cloud-touching:** which cloud account / profile, local
  emulators vs real services, credential strategy

### Mandatory decision categories

Skip any already answered by the design doc. Do not skip silently
— briefly note that a category was already covered.

**a. Shell & editor environment.** PowerShell + VSCode native,
WSL2 + VSCode, plain bash on macOS/Linux. This decision is
load-bearing — it shapes every command example in the README and
the rule files. Cross-reference `rules/environment-rules.md` "Shell
handling" section so the answer is consistent with how Claude
already runs commands in this project.

**b. Stack-specific bootstrap.** The concrete command sequence to
go from "fresh clone" to "running the app locally." This becomes
the README Developer setup steps verbatim — ask in enough detail
to write working commands, not vague gestures.

**c. Tooling beyond the language runtime.** Docker, cloud CLIs,
DB engines, infra tools, browser drivers — anything Phase 0 will
need installed but the language-and-package-manager choice from
`/onboard` doesn't capture.

**d. Secrets & config in dev.** Where dev secrets live (`.env`
file, OS keychain, secret manager), how local config is loaded,
what's safe to commit vs gitignore. Production secrets are
`/deployment-plan`'s problem — stay focused on dev here.

If the stack includes a database, also ask:
- **App DB username** — propose a project-scoped default
  (e.g. `<slug>_app`, `<slug>_dev`). NFR-X (secrets hygiene) in
  `REQUIREMENTS.md` forbids `root`, `admin`, `sa`, and other
  privileged defaults; surface the constraint to Jamie alongside
  the suggestion. **Never propose `root`.** If
  `REQUIREMENTS.md` is missing the NFR (older project, or design
  has no DB and one was added later), fall back to the
  "Database credentials policy" backstop in
  `rules/environment-rules.md` — same rule.
- **App DB password handling** — generated locally, stored in
  `.env`, `.env` gitignored.

CI ephemeral DB usernames are *not* asked here. Phase 0+ generates
them per the NFR (two-word-with-number pattern) when the CI
workflow lands.

## Step 4: Confirm answers

Before writing any artifacts, read back to Jamie:
- The exact bootstrap command sequence (will land in README
  verbatim)
- The shell expectation
- The tool list
- The verify commands (one per tool; these become the
  `### Verify your setup` subsection and are what stage 2 runs)
- Where dev secrets live and what's gitignored

This is the most important confirmation in the command — a wrong
bootstrap step list produces a Developer setup section that won't
work on a fresh clone.

## Step 5: Generate artifacts

### `README.md` (modify)

The README was written as a stub by `/onboard`. Replace the
contents of the "Developer setup" section (currently a one-line
placeholder) with the real content below, in place. The section
sits after the one-paragraph description and before "Deployment".

**Version freshness checkpoint (write time).** Before any version
number lands in this section, re-confirm per **Rule 10**
(`rules/coding-session-rules.md`). Even if Step 3 captured a version
with Jamie's go-ahead, ask once more before it goes into a file
Jamie will commit. The Prerequisites and Verify your setup
subsections below both contain version pins — both are in scope.

Required subsections under "Developer setup":

- **Prerequisites** — OS support, shell choice, required tools
  (from category c) with version expectations and install
  pointers.
- **Verify your setup** — a numbered list of one-line shell
  commands proving each tool from Prerequisites is on PATH and
  at the expected major version. One command per tool (e.g.
  `python --version`, `docker --version`, `psql --version`).
  These are the same commands stage 2 runs to flip status to
  `COMPLETE`; humans can copy-paste them too. Use the project's
  chosen shell — don't bash-chain in a Windows-primary project.
- **Bootstrap** — the verbatim command sequence from category b.
  One command per line. No bash chaining if the project is
  Windows-primary.
- **Run locally** — the command(s) to start the app for dev.
- **Test** — the command(s) to run the test suite (full and any
  fast subset, e.g. unit-only).
- **Secrets and config** — from category d. Where the `.env`
  template lives, what to copy, what is gitignored. If a database
  is in the stack, record the **app DB username** chosen in
  category d (e.g. `<slug>_app`) and note that CI ephemeral DB
  usernames are generated per NFR-X (two-word-with-number pattern)
  when the CI workflow lands in Phase 0+ — not committed here.

Leave the "Deployment" stub section as-is — `/deployment-plan`
fills that in. (If `DEPLOYMENT-PLAN-STATUS` is already `COMPLETE`,
that's a re-run case — leave the existing Deployment section
alone.)

### `rules/environment-rules.md` (append)

**Version freshness checkpoint (write time).** Same gate as the
README write step above — re-confirm every version per **Rule 10**
(`rules/coding-session-rules.md`) before it goes into the
`<!-- ONBOARD-FILL: environment -->` block. The "Language and
runtime" and "Tooling beyond language" bullets below both pin
versions; both are in scope.

Inside the existing `<!-- ONBOARD-FILL: environment -->` block,
replace the placeholder bullets with concrete content:

- **Language and runtime** — version pinned (e.g. "Python 3.12",
  "PHP 8.3 / Laravel 11", "Node 20 / TypeScript 5.4").
- **Shell expectation** — which shell Jamie uses for this project
  (PowerShell + VSCode, WSL2, etc.). Reference back to the
  "Shell handling" section above so it's consistent.
- **Venv / environment activation** — the specific path and
  activation pattern. For PowerShell projects, also note the
  Claude-bash and Claude-PowerShell prefixes (per the existing
  "Commands Claude runs via its own Bash or PowerShell tools"
  rule).
- **Build / run / test commands** — canonical one-liners. These
  are what Claude pastes into rule-7 commit handoffs and what
  Jamie pastes into terminals. Make them work in the chosen
  shell.
- **Tooling beyond language** — Docker, cloud CLIs, DB engines,
  etc., with version expectations.

Keep the surrounding `<!-- ONBOARD-FILL: environment -->` /
`<!-- /ONBOARD-FILL -->` markers intact.

## Step 6: Update CLAUDE.md

Rewrite `CLAUDE.md`:

1. Replace `<!-- BOOTSTRAP-STATUS: UNCONFIGURED -->` with
   `<!-- BOOTSTRAP-STATUS: INSTRUCTIONS-WRITTEN <YYYY-MM-DD> -->`
   using today's date. Leave the other two status comments alone.
   Stage 2 will flip this to `COMPLETE <date>` once the developer
   has installed the listed tools and validation passes.
2. Replace the post-onboard banner (the ✅ block written by
   `/onboard`) with a stage-1 bootstrap banner:

   > ⚠️  **Bootstrap instructions written, not yet validated.**
   > Follow `README.md` § Developer setup to install the listed
   > tools, then re-run `/bootstrap` to verify them on PATH. On
   > success status flips to `COMPLETE` and the project is ready
   > for Phase 0.
   >
   > **Project:** \<name\> — \<one-paragraph description\>
   > **Language / framework:** \<from onboarding\>
   > **Multi-agent mode:** \<from onboarding\>
   > **Shell:** \<from Step 3a\>
   > **Run:** `<run command>` &nbsp;·&nbsp; **Test:** `<test command>`
   > **Developer setup:** see `README.md`

3. Add a session-start gate immediately after the reading-order
   block and before "## Collaboration rules":

   > **Bootstrap not yet validated.** While `BOOTSTRAP-STATUS` is
   > `INSTRUCTIONS-WRITTEN`, before any work this session surface:
   > "Bootstrap instructions are written but the dev environment
   > hasn't been validated. Run `/bootstrap` to verify the
   > required tools are on PATH (it will auto-enter stage 2). Want
   > to do that first, or push ahead?" If Jamie pushes ahead,
   > Phase 0 will likely fail on missing tools — the failure is a
   > legitimate signal to come back and validate.

Keep the reading order, collaboration rules, and references
unchanged. Stage 2 removes both the gate and the warning banner
once status flips to `COMPLETE`.

## Step 7: Surface git commands (rule 7)

Do not run any git commands. Surface a copy/paste-ready block per
the commit-handoff format in `/wind-down`'s Step 4.

```
git status
```

Show what to stage:

```
git add CLAUDE.md README.md rules/environment-rules.md
```

```
git status
```

Then a single-block commit:

```
git commit -m "Bootstrap dev environment for <project name>

Developer setup written to README.md; environment-rules.md filled.
- Shell: <shell choice>
- Stack: <one-line summary>"
```

## Step 8: Final report

Summarize for Jamie:
- Files modified (one bullet per file with one-line description)
- The shell and stack choices
- What the developer needs to do next: install the tools listed
  in `README.md` § Developer setup § Prerequisites, then re-run
  `/bootstrap` to enter stage 2 (validation). Once status flips
  to `COMPLETE`, run Prompt 0 from
  `docs/CLAUDE_CODE_PROMPTS.md` to start Phase 0.
- If a `design-review-checkpoint-*.md` file in `docs/design/` is
  still `AWAITING-DECISIONS`, remind Jamie to mark it up and rerun
  `/design-review` before pasting Prompt 0 — the review's findings
  may revise REQUIREMENTS / ARCHITECTURE / Prompt 0 before scaffolding
  begins.
- A reminder that `/deployment-plan` is the third configuration
  command and can be run whenever test/prod planning is needed
  — typically after Phase 0 or once dev experience has accumulated.

End with: "Bootstrap stage 1 complete. Follow `README.md` §
Developer setup to install the listed tools, then re-run
`/bootstrap` to validate."

---

## Stage 2: Validate

This stage runs when `BOOTSTRAP-STATUS` is
`INSTRUCTIONS-WRITTEN`. The developer has (presumably) followed
the README and installed the listed tools. This stage proves it.

### Step S1: Read the verify commands

Open `README.md` § Developer setup § `### Verify your setup`.
Read each one-line command. If the subsection is missing,
surface as unexpected state and ask Jamie how to proceed. Do
not invent commands.

### Step S2: Run each command

Run each command using the project's chosen shell (per
`rules/environment-rules.md`). Capture the exit code and
stdout/stderr for each.

### Step S3: Report and flip

**On success** (every command exits 0):

1. Rewrite `CLAUDE.md`:
   - Flip `<!-- BOOTSTRAP-STATUS: INSTRUCTIONS-WRITTEN <date> -->`
     to `<!-- BOOTSTRAP-STATUS: COMPLETE <YYYY-MM-DD> -->` using
     today's date.
   - Replace the stage-1 warning banner with the validated
     banner:

     > ✅ **Onboarded and bootstrapped.** Ready for Phase 0. Run
     > Prompt 0 from `docs/CLAUDE_CODE_PROMPTS.md` when ready.
     > `/deployment-plan` is deferrable — run it when test/prod
     > planning is needed.
     >
     > **Project:** \<name\> — \<one-paragraph description\>
     > **Language / framework:** \<from onboarding\>
     > **Multi-agent mode:** \<from onboarding\>
     > **Shell:** \<from environment-rules\>
     > **Run:** `<run command>` &nbsp;·&nbsp; **Test:** `<test command>`
     > **Developer setup:** see `README.md`

   - Remove the session-start "Bootstrap not yet validated" gate
     added in stage 1.

2. Surface git commands per rule 7. Stage 2 only modifies
   `CLAUDE.md`:

   ```
   git status
   ```

   ```
   git add CLAUDE.md
   ```

   ```
   git status
   ```

   ```
   git commit -m "Validate bootstrap for <project name>

   All required tools verified on PATH; BOOTSTRAP-STATUS → COMPLETE."
   ```

3. Final report: a one-line confirmation per verified tool with
   its detected version, plus a reminder that `/deployment-plan`
   is the third configuration command and can be run whenever
   test/prod planning is needed.

End with: "Bootstrap validated. Ready for Phase 0. Run Prompt 0
from `docs/CLAUDE_CODE_PROMPTS.md` when ready."

**On any failure** (any command exits non-zero or the tool isn't
found):

1. Do NOT modify any files. Do NOT flip the status.
2. Report each failed check: the command, the exit code, the
   tool it was checking, and a pointer to the corresponding
   entry in `README.md` § Developer setup § Prerequisites.
3. End with: "Bootstrap validation failed. Install the missing
   tools per `README.md` § Developer setup § Prerequisites, then
   re-run `/bootstrap` to validate again."

---

## Failure modes

- **`/onboard` hasn't run.** Refuse and point at `/onboard`. Don't
  try to do both jobs in one command — they're split for a reason.
- **Stack is too unclear in the design doc to ask tailored
  questions.** Ask Jamie directly what the stack actually is
  before proceeding. Don't invent. If the design genuinely doesn't
  pin a stack (e.g. "we'll figure out language later"), refuse:
  bootstrapping is premature.
- **Bootstrap commands won't actually work on a fresh clone.**
  Push back: "These steps don't look complete — \<specifically what
  is missing\>. What's missing?" Do not write a Developer setup
  section that won't work. A non-working setup section is worse
  than no setup section.
- **Existing `README.md` Developer setup section was hand-edited.**
  Surface the diff, ask per-section. Never silently overwrite.
- **`<!-- ONBOARD-FILL: environment -->` markers are missing from
  `rules/environment-rules.md`.** The file has been hand-edited in
  a way this command can't safely append to. Surface the issue and
  ask Jamie how to proceed.
- **Stage 2 invoked but `### Verify your setup` is missing from
  `README.md`.** The README was hand-edited to remove it, or
  stage 1 didn't run cleanly. Surface as unexpected state and
  ask Jamie. Do not invent verification commands.
- **Stage 2 found a tool missing or wrong version.** Report the
  failed checks with command, exit code, and Prerequisites
  pointer. Do not flip the status.

---

## What this command does NOT do

- Does not plan deployment — `/deployment-plan` does.
- Does not run any git commands (rule 7).
- Does not run any tests (rule 8).
- Does not actually install dependencies, create venvs, or run
  package managers — it documents how to. Phase 0 (or Jamie
  manually) does the actual install.
- Does not install dependencies in stage 2 either. Stage 2 only
  checks whether tools are on PATH; it never installs. If a tool
  is missing, the developer installs it per the README and
  re-runs `/bootstrap`.
- Does not push to remotes.
- Does not write CI workflows. Phase 0 handles that.
- Does not run design reviews — `/design-review` does. `/bootstrap`
  surfaces a soft recommendation in Step 0 and a reminder in Step 8
  if a review is missing or `AWAITING-DECISIONS`, but never refuses
  on that basis and never edits checkpoint files.
