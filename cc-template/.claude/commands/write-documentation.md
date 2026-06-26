---
description: Author and maintain audience-facing product documentation. Stage 1 surveys the project and proposes a documentation plan for sign-off; stage 2 writes or reconciles the docs (README, quick-start, guides, reference, troubleshooting) applying a documentation-craft doctrine. Writes markdown sources plus a product-type-aware delivery recipe; the render/build is owned by /deployment-plan.
---

# /write-documentation

`/write-documentation` authors the **audience-facing documentation** a
product ships — for its end users, administrators, support staff,
developers, and contributors. It is distinct from the internal SDLC
planning docs (REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN): those describe
*what we're building and why*; this command writes *how to install, use,
operate, and extend the thing we built*.

Unlike the other commands, its output has no fixed shape — it depends on
the product. So it is a **reasoning** command: it surveys the project,
**infers** the product type and the audiences, **proposes** a documentation
set for sign-off, then **authors** it. It never assumes a doc structure from
a template; it reasons one out per project and confirms it.

It is also an **ecosystem citizen**. It exposes a machine-readable currency
signal and a release-readiness ledger that `/deployment-plan`'s release
procedure can gate on; it owns the user-facing sections of `README.md`
(which `/wind-down` then routes around rather than clobbering); and it reads
`/product-visioning`'s PRDs to know which *movement* the docs are current
through.

This command runs in the current working directory and writes files
relative to it.

**Required reading before Step 0:** read `rules/coding-session-rules.md` and
`rules/design-philosophy-rules.md` in full.

- **Rule 4** governs scope — propose the *minimal* doc set that covers real
  user jobs; never spin up a doc type with no audience.
- **Rule 8** governs the boundary with product code — when the docs can't be
  made correct without a *software* change (a wrong `--help`, an undocumented
  option), surface it as a **conformance gap**; never silently edit the
  product to match the docs.
- **Rule 10** governs version pins — flag any version written into a doc as
  possibly-stale and confirm before pinning.

---

## What it produces

- A numbered **documentation plan** (the manifest):
  `docs/published/documentation-plan-NNN.md`. Newest governs; it holds the
  doc-set definition, the audience map, the `documented-through` currency
  stamp, the **delivery recipe**, and the release-readiness ledger.
- The **doc sources**: markdown under `docs/published/`, plus images under
  `docs/published/images/` (real screenshots and labelled placeholders).
- The **user-facing sections of `README.md`** (and `CONTRIBUTING.md` when the
  project takes contributors) — authored by offer, never clobbering sections
  another command owns.

Markdown is the deliverable. Rendering or building the delivered docs is
`/deployment-plan`'s job — it reads this command's delivery recipe + ledger and
codes the render targets against whatever runtime exists at release time. This
command owns no renderer.

### Arguments

Progressive disclosure — bare invocation handles the happy path.

- `/write-documentation` — auto-detect stage per Step 0.
- `/write-documentation --doc "<name>"` — scope a Stage 2 pass to one doc
  (regenerate or reconcile just that source).
- `/write-documentation --audience "<role>"` — scope the proposed set to one
  audience.
- `/write-documentation --new` — force Stage 1 (a fresh re-scoped plan) even if
  the current plan is `ACTIVE` and current. Rare.

---

## Two stages, auto-detected; movement-aware currency

- **Stage 1 — Survey & Plan.** Read the project (internal docs *and* the real
  implementation), run the **doc-set decision procedure**, infer product type
  + audiences + feature exposure, and write a new numbered manifest with one
  approval placeholder per proposed doc. Status `AWAITING-APPROVAL`. Authors no
  doc sources yet (the sign-off gate precedes expensive prose). Runs on the
  **first** documentation pass and on a **re-scope** — a new movement changed
  which docs/audiences exist.
- **Stage 2 — Author/Update.** Two entry modes:
  - **Author** — the latest manifest's proposed docs are all marked: write the
    approved sources applying the writing doctrine, fill the release-readiness
    ledger, stamp `documented-through`, flip the manifest to `ACTIVE`.
  - **Reconcile** — the current `ACTIVE` manifest is stale within the *same*
    movement (a later phase shipped): refresh the affected docs against what
    changed, append an authoring-log round, re-stamp. No new sign-off — the
    doc set was already approved. (If the change needs *new* docs or audiences,
    it's a re-scope — route to Stage 1.)

**No "LANDED" — currency is derived, and movement-aware.** Docs track a moving
product. The stamp is a tuple: `documented-through: { movement, phase }`.

- **Current movement** = the newest `docs/design/PRD-<slug>-*.md` with
  `status: ACTIVE` (its id); `initial` if the project predates
  `/product-visioning` (no PRDs). This is the same detection `/onboard` uses —
  a read-only dependency on `/product-visioning`'s artifacts.
- **Staleness** (any consumer derives it from the manifest): the active
  movement ≠ the stamped movement → **STALE** (a new movement → re-scope);
  else a later phase has completed in the *same* movement → **STALE** (a
  same-movement reconcile). Phase numbers are compared **only within one
  movement** — this
  defuses the trap where a new movement's Phase 1 looks "behind" a prior
  movement's Phase 5.

Stage detection never asks Jamie to flip a status by hand — the manifest's
markup and currency state signal readiness.

---

## Step 0: Detect state and stage

1. **Determine the current movement + phase.** Read `docs/PRODUCT_VISION.md`
   (or the newest PRD's `project:`) for the `<slug>`, glob
   `docs/design/PRD-<slug>-*.md`, and take the newest `status: ACTIVE` PRD as
   the current movement id; `initial` if there are no PRDs. Read
   `docs/PROJECT_PLAN.md` for the latest completed phase within that movement.

2. **Find the governing manifest.** Glob `docs/published/documentation-plan-*.md`
   (zero-padded NNN sorts numerically through 999). The newest governs.
   - **No manifest** → **Stage 1**, `N = 001`.
   - **Newest `AWAITING-APPROVAL`**:
     - Any proposed doc in the latest round is still the unmarked placeholder
       → **refuse**: list the unmarked doc names and the markup legend; "mark
       the plan, then re-run."
     - Every proposed doc in the latest round is marked → **Stage 2** (author
       the marked set).
   - **Newest `ACTIVE`** → derive currency:
     - **CURRENT** (stamped movement/phase == current) → nothing to do. Report
       "documentation is current through movement `<id>` / phase `<id>`." If
       `--doc` was passed, do just that scoped work; `--new` forces Stage 1.
     - **STALE, movement changed** → **Stage 1** (re-scope), `N = previous + 1`
       (the prior is superseded at Stage 2, once the new set is authored).
     - **STALE, same movement, phase advanced** → **Stage 2 reconcile**
       (refresh the affected docs in the current manifest; no new sign-off).
   - **Newest any other status** → unexpected; surface and ask.

The placeholder line that distinguishes an unmarked proposed doc is exactly:

```
> _[UNMARKED — approve / adjust: <note> / drop, per the legend above]_
```

Any DOC DECISION block whose body is still that line counts as unmarked.
Anything else (`approve`, `adjust: …`, `drop`) counts as marked.

---

# Documentation-craft doctrine

Apply this throughout. Stage 1's decision procedure shapes the *plan*; the
writing rules shape the *prose* in Stage 2. The doctrine is grounded in
Diátaxis, the Google and Microsoft writing style guides, minimalism, and the
"every page is page one" topic model.

## Two governing meta-rules

1. **Classify, then write (the Diátaxis compass).** Every section is *exactly
   one* mode — tutorial, how-to, reference, or explanation. A document may
   contain several modes, but a single section must not blend them. Blending
   modes within a section is the single biggest cause of bad documentation.
2. **Fix the audience and expertise, then hold the register constant for the
   whole document.** Never average across audiences: if two readers both need
   the content, write two documents, or hard-segment with a named *primary*
   audience. (Writing for "everyone" under-explains for the novice and
   over-explains for the expert at the same time.)

## The four modes

- **Tutorial** (learning-oriented): take a beginner by the hand to a
  *guaranteed first success*. Ordered, always succeeds, shows results early.
  MUST NOT explain at length or branch into options.
- **How-to** (task-oriented): direct a *competent* reader through a real task
  to a goal. Action only; goal-stating title ("How to configure TLS"). MUST
  NOT teach fundamentals or digress into theory.
- **Reference** (information-oriented): austere, factual description that
  *mirrors the product's own structure*. Exact, complete, consulted not read.
  MUST NOT instruct or explain.
- **Explanation** (understanding-oriented): the *why* — context, trade-offs,
  alternatives; opinion is allowed. MUST NOT give steps or close-up reference
  detail.

## Doc-set decision procedure (Stage 1)

Minimalism gates everything — documentation is a cost; a small, fresh,
accurate set beats a large set in disrepair. Decide the set like this:

1. Classify the **product type** (CLI / GUI / SaaS / library / service / …)
   and the **audiences** (from roles, auth tiers, admin-only features in the
   specs and the code).
2. Enumerate the **real user jobs**. *No job, no doc.*
3. Map jobs to the **minimal Diátaxis coverage** that serves them.
4. Include a **quick-start** (the first-success happy path) for any product a
   newcomer must run — it is a near-default; skip it only for a trivial
   surface.
5. **Scale reference depth to surface area.** A three-flag CLI needs a README
   plus `--help`, not a five-chapter manual.
6. Add type-specific docs only on a trigger: admin/install guide ⇐ an admin or
   operator role; API reference ⇐ a public API; release notes ⇐ a versioned
   service; troubleshooting ⇐ known failure modes; `CONTRIBUTING` / PR guide ⇐
   the project takes contributors.
7. The **README orients and routes** — it does not contain everything.

Map modes to artifacts: README = orientation + a quick-start how-to + thin
reference that *links out*; quick-start = how-to leaning (one happy path);
getting-started = tutorial; task guides = how-to; CLI/API reference =
reference; troubleshooting = how-to + reference; admin/install = how-to +
reference; explainer/architecture-for-users = explanation. A "user manual" is
a *navigable container of single-mode chapters*, never a blended narrative.

## Audience voice profiles

Pick the audience, write to its profile, hold it for the whole document.

| Audience | Reading goal | Assumes | Tone | Examples / code |
|---|---|---|---|---|
| **End user (non-technical)** | finish one task; recover from an error | general app literacy only | warmest, plain, reassuring; no jargon | none; visuals + numbered steps |
| **Power user** | go faster; automate; configure deeply | knows the product cold | friendly but lean; peer-to-peer | medium: config, CLI, scripting |
| **IT admin / installer** | deploy, secure, integrate reliably | infra-expert, novice about *this* product | precise, neutral, formal | high infra code; prereq/port tables |
| **Support rep** | diagnose a customer issue fast; escalate | product-broad, issue-shallow | terse, scannable under pressure | symptom→cause→fix; decision logic |
| **Developer (API/SDK)** | zero-to-working-call, then reference | expert coder, novice about *this* API | "a knowledgeable friend, not pedantic" | highest: runnable, multi-language |
| **Maintainer / contributor** | change the system safely; contribute | strong engineer, maybe new to this code | candid peer; honest trade-offs | architecture, rationale, build/test, PR flow |

## Writing and readability rules

- Active voice; second person ("you"); present tense; imperative mood for
  steps.
- Plain language; get to the point, then stop; cut "you can" and "there is".
- **Guard the curse of knowledge** — an author who knows the system
  under-explains by default. Spell out the "too-obvious" intermediate steps;
  define every term and acronym on first use; calibrate to the named
  audience's actual prior knowledge.
- One idea per sentence (~20-word average); one topic per paragraph (3–7
  lines).
- **Front-load** (inverted pyramid) at the document, section, and paragraph
  scale — the reader can stop anywhere and still have the main point.
- Descriptive, keyword-first headings (never "Introduction" / "Overview");
  sentence case.
- Numbered list = a sequence; bulleted list = an unordered set; **table =
  facts, specs, comparisons**; prose = concepts only. Never bury a sequence of
  actions inside a paragraph.
- Prerequisites go in a "Before you begin" section *above* the steps; steps in
  execution order; a step never references something not yet obtained.
- One term per concept, every time (no elegant variation).
- Accessibility: alt text ≤ ~125 characters summarizing the image's intent;
  meaningful link text (never "click here"); no directional language
  ("above" / "below" / "on the right"); never rely on color alone.
- Forbid condescension: "simply", "just", "easy", "obviously", "of course".

## CLI reference doctrine

For a command-line product, the reference is load-bearing. Per command:

> **NAME** · **SYNOPSIS** · **DESCRIPTION** (what + when) · **OPTIONS** ·
> **ARGUMENTS** · **EXAMPLES** (progressive, with expected output) ·
> **EXIT CODES** · **SEE ALSO**

- SYNOPSIS uses POSIX notation: `[ ]` optional, `|` mutually exclusive, `...`
  repeatable, `--` ends options.
- **Enumerate every option from the real implementation** — parse `--help` or
  read the argument parser; never invent an option and never omit one. Each
  option states **what it does / why-or-when to use it / its default**; show
  both short and long forms; use typed placeholders (`<FILE>`, `<PORT>`);
  group options when numerous.
- **Prose and `--help` must agree.** A disagreement is a *conformance gap*
  (see below), not something to paper over in prose.
- Enumerate exit codes and map each to a failure mode; flag destructive
  options explicitly.

## Examples, troubleshooting, visuals

- **Code samples:** minimal but complete; copy-pasteable (mark an omission
  with a language comment, never an ellipsis; don't mark a block
  click-to-copy if it contains an omission); use realistic values where they
  clarify; show expected output in a *separate, non-copyable* block; progress
  simplest-first.
- **Troubleshooting:** each entry is **Symptom → Cause → Fix (+ Prevention)**.
  State the symptom in the user's words, **including the verbatim error
  string** (that is what makes it findable). Order causes most-common-first;
  order fixes simplest-first with an explicit "if that fails, next…".
- **Visuals:** propose an image only if it passes the *decorative test* —
  what information is lost without it? If nothing, it is clutter that will
  age badly. Any information in an image must also be in the prose. See the
  screenshot-placeholder convention in Stage 2.

## Anti-patterns to forbid

Wall of text · the curse of knowledge · passive-voice instructions · mixing
two modes in one section · undefined jargon · screenshots used as a crutch
(the text steps must stand alone) · condescending filler · stale or
un-runnable examples.

---

# Stage 1 — Survey & Plan

Stage 1 surveys the project, runs the decision procedure (above), and writes a
new numbered manifest for sign-off. It authors no doc sources — that is
Stage 2's job. Stage 1 runs on the first documentation pass and on a re-scope.

## Step S1.1: Survey the project

Read what exists; skip absent files. Read both the *intent* docs and the
*real implementation* — specs drift, and the docs must describe what the
product actually does.

1. `docs/REQUIREMENTS.md`, `docs/ARCHITECTURE.md` — the contract and shape.
2. `docs/PROJECT_PLAN.md`, `docs/CLAUDE_CODE_PROMPTS.md` — what has shipped
   (which phases are complete), and through which movement.
3. The active `docs/design/PRD-<slug>-*.md` and `docs/PRODUCT_VISION.md` —
   the product's positioning and personality (these set the documentation's
   voice).
4. `docs/DEPLOYMENT.md` — install / topology / runtime, for install and
   operator docs. If `DEPLOYMENT-PLAN-STATUS` is `UNCONFIGURED` / `DEFERRED`,
   stub or skip install-and-ops sections; **never invent topology** (rule 8).
5. Prior `docs/design/design-review-checkpoint-*.md` and
   `docs/test-plans/phase-*-exit.md` — known limitations and behaviours users
   will hit.
6. **The implementation itself.** For a CLI, read the argument parser / the
   real `--help`. For an API or library, read the public surface. For a GUI,
   read the screens/flows. This is where the option-level truth lives; a
   broad read here is exactly what subagents are for (see
   `rules/multi-agent-rules.md`).

Build a picture of: product type, the audiences and which features each is
exposed to, and what shipped through (the `documented-through` coordinate). On
a re-scope, also note how the new movement changes the doc set from the prior
manifest.

## Step S1.2: Reconnaissance

State back a 4–6 bullet picture: the product type and how it's installed and
run; the audiences you infer and why; the candidate doc set from the decision
procedure (with anything you propose to *skip* and why — minimalism is
visible); the product's voice/personality from PRODUCT_VISION; and the
**delivery recipe** you'll record — what well-delivered docs look like for this
product type (researched or discovered at run time, not a fixed menu). Ask Jamie
to confirm or correct before you write the plan. Use `AskUserQuestion` for
genuine forks (an ambiguous audience split, an uncertain product-type call).

## Step S1.3: Write the manifest

Filename `docs/published/documentation-plan-NNN.md`, zero-padded 3-digit.
Status `AWAITING-APPROVAL`. One DOC DECISION block per proposed doc. Use this
template:

````markdown
---
documentation-plan: NNN
project: <slug>
created: YYYY-MM-DD
status: AWAITING-APPROVAL
documented-through:
  movement: <active PRD id, or "initial">
  phase: <latest completed phase in that movement, or "none">
---

# Documentation Plan NNN — <project>

## Context

<Product type; how it's installed and run; the audiences and which features
each sees; the full documentation set this plan covers. Names what is
deliberately NOT documented, and why.>

## How to mark up this plan

For each proposed doc below, replace the placeholder line under
`DOC DECISION — JAH:` with one of:

- `approve` — author it as proposed.
- `adjust: <note>` — author it with this change (re-scope, re-audience,
  re-outline).
- `drop` — do not author it.

To request a doc that isn't proposed, append a new doc block and mark it
`approve`. Then re-run `/write-documentation`; Stage 2 authors the approved
set. Mark each independently.

## Proposed documentation set

| Doc | Path | Primary audience | Mode(s) |
|-----|------|------------------|---------|
| <name> | docs/published/<file>.md | <audience> | <modes> |

### <doc name> — `docs/published/<file>.md`

- **Primary audience:** <audience> (voice profile applied)
- **Diátaxis mode(s):** <tutorial / how-to / reference / explanation>
- **Job served:** <the real user job — why this doc exists>
- **Outline:** <section list; each section tagged with its single mode>
- **Visuals:** <planned image placeholders with capture notes, or "none">

> DOC DECISION — JAH:
> _[UNMARKED — approve / adjust: <note> / drop, per the legend above]_

## Audience × feature exposure

<Matrix: feature (rows) × audience (columns); mark which audiences a feature
is documented for. Features gated from an audience are not documented for it.>

## Delivery recipe

<What well-delivered docs look like *for this product* — the form the docs
should reach their audiences in. This is for `/deployment-plan`, which owns the
render/build: it reads this recipe and codes the targets against the runtime
that exists at release time.

Research or discover the product type at run time — from the specs, the
implementation, and (when it's genuinely unclear) by asking the developer. Do
**not** pick from a fixed menu; SaaS / downloadable zip / library are only
illustrations of how delivery form follows product type:

- a **SaaS / web app** → in-app or hosted HTML help, versioned with the app;
- a **downloadable tool** → a README + quick-start that travel in the package,
  maybe a single rendered document;
- a **library / API** → a generated reference site next to the package.

Describe *what* good delivery is here — the target forms, where they live, what
ships in a release, whether build output is committed or gitignored. Name **no**
toolchain, engine, or runtime: NFR-6 keeps the template runtime-free, and the
*how* is `/deployment-plan`'s to code at build time. If the product type is
still unsettled, say so and record what's known.>

## Release-readiness ledger

*Stage 2 fills this. Outstanding items a release process may gate on; each is
OPEN or RESOLVED.*

| Type | Item | State | Detail |
|------|------|-------|--------|
| <Stale / Unfilled visual / Conformance gap> | | | |

## Authoring log

*Stage 2 fills this. Append-only; one row per authoring round.*

| Round | Date | documented-through | Docs touched | Notes |
|-------|------|--------------------|--------------|-------|
````

If a section has nothing yet, write a one-line italic note rather than
omitting it. Author **no doc sources** in Stage 1 (rule 4 — the sign-off gate
precedes the prose).

## Step S1.4: TODO and report

Rewrite `TODO.txt` so the first entry under "What's next" is:

```
- Mark up docs/published/documentation-plan-NNN.md (approve / adjust / drop
  each proposed doc per the legend), then re-run /write-documentation. Stage 2
  authors the approved set.
```

Carryovers preserved below. Report: the manifest path, the proposed doc count
(and what you proposed to skip), and that marking up + re-running enters
Stage 2. The manifest is a created artifact — `/wind-down` owns the commit
handoff (rule 7); don't surface git here.

End with: "Documentation plan NNN written. Mark up the proposed docs, then
re-run `/write-documentation` to author them."

---

# Stage 2 — Author / Update

Two entry modes (per Step 0): **author** the approved set, or **reconcile** the
current set for a same-movement phase advance. Both apply the doctrine and
update the manifest.

## Step S2.1: Settle the set to author

**Author mode** (the latest manifest's DOC DECISION blocks are all marked):
for each proposed doc, read its marking — `approve` (author as proposed),
`adjust: <note>` (author with the change), `drop` (skip). Surface only genuine
ambiguities (a malformed marking, an `adjust` note that conflicts with the
doc's stated mode/audience).

**Reconcile mode** (`ACTIVE` + stale, same movement): walk the **doc-rot
signals** against what shipped since the stamp — vanished or newly-undocumented
options, dead samples, mismatched screenshots, drifted facts, stale links — and
list the docs to refresh. If the change needs a *new* doc or audience (not just
content), stop and route to Stage 1 (a re-scope); reconcile is for the approved
set only.

Either mode: state back the set you will author/refresh and confirm before
writing.

## Step S2.2: Author or reconcile each approved doc

For each approved doc, apply the doctrine:

- **Classify every section** into exactly one Diátaxis mode; never blend.
- **Hold the audience's voice profile** for the whole document.
- Apply the writing and readability rules. For a CLI, build the reference from
  the **real implementation**, documenting every option (what / why-or-when /
  default).
- On an **update** (the source already exists), reconcile rather than rewrite:
  refresh what the doc-rot signals flagged, and leave human-edited prose whose
  scope is unchanged intact.
- **Screenshot placeholders.** Where a visual earns its place (decorative
  test), write an inline reference and a greppable placeholder so the
  maintainer has a shoot-list and unfilled images are caught before release:

  ```
  ![<alt text, ≤125 chars, summarizing intent>](images/<stable-name>.png)
  <!-- VISUAL TODO: capture <what screen/state, how to reach it in the product>.
       Caption: "<caption>". Why it earns its place: <what text alone can't show>. -->
  ```

  If the image already exists in `docs/published/images/`, the reference
  renders it; the placeholder comment is removed once filled. The image's
  information must also be present in the prose.

Write sources under `docs/published/` per the manifest's recorded paths
(create `docs/published/images/` as needed).

## Step S2.3: README / CONTRIBUTING (by offer)

`/write-documentation` owns the **user-facing sections** of `README.md` —
overview, install, usage, getting-started, and a pointer to the quick-start —
and authors `CONTRIBUTING.md` when the project takes contributors. It does
**not** touch sections other commands own: **Developer setup** (`/bootstrap`)
and **Deployment** (`/deployment-plan`). On an existing README, **ask before
editing**, and edit only the user-facing sections. State what you'll add or
change and confirm.

## Step S2.4: Update the manifest

Edit the manifest (the file this command owns):

1. **Release-readiness ledger** — record OPEN items:
   - **Unfilled visual** — every placeholder written but not yet captured,
     with its capture instructions.
   - **Conformance gap** — every doc/code discrepancy that needs a *software*
     change (a wrong or missing `--help`, prose that can't be made to agree
     with the real CLI). Describe the gap and the change it implies; **do not
     edit the product** (rule 8). These queue as coding tasks.
   - Mark a prior OPEN item RESOLVED when this pass closed it.
2. **Authoring log** — append one row: round (Original / Reconcile N), date,
   the new `documented-through`, the docs touched, notes.
3. **`documented-through`** — re-stamp to the current `{ movement, phase }`.
4. **Frontmatter status & supersession** — in author mode, flip
   `AWAITING-APPROVAL` → `ACTIVE`; in reconcile mode it stays `ACTIVE`. If this
   was a re-scope (a new manifest number) and a prior manifest is still
   `ACTIVE`, flip the prior to `SUPERSEDED <date>` — newest governs.

## Step S2.5: Cross-doc surfacing, TODO, report

- If this pass created a doc that ships in releases, surface a TODO:
  "`docs/DEPLOYMENT.md`'s shipped-artifacts list may need `<doc>` — that's
  `/deployment-plan`'s to land." Do not edit DEPLOYMENT.md.
- If the ledger has OPEN **conformance gaps**, surface them as coding TODOs
  (one per gap) — they may gate a release until resolved.
- Rewrite `TODO.txt`'s first "What's next" entry to whatever the session's
  state implies (often: capture the listed screenshots; resolve conformance
  gaps; or the next planning step). Carryovers preserved.
- Report: the docs written/updated, the manifest's new `documented-through`,
  OPEN ledger items (unfilled visuals + conformance gaps), and anything
  surfaced for `/deployment-plan`. This is an artifact close — `/wind-down`
  owns the commit handoff (rule 7); don't surface git here.

End with: "Documentation authored through movement `<id>` / phase `<id>`.
`<N>` images to capture, `<M>` conformance gaps to resolve" (omit a count when
zero).

---

## Failure modes

- **No `docs/published/` and no implementation to read.** Refuse: the command
  documents a product; there must be something built to document.
- **A proposed doc is left unmarked.** Refuse Stage 2; list the unmarked docs
  and the legend; do not author a partially-approved set.
- **A marking is malformed** (not `approve` / `adjust:` / `drop`). Surface and
  ask; don't guess.
- **The docs can't be made correct without changing the product** (`--help`
  wrong, an option undocumented in `--help`). Record a **conformance gap** in
  the ledger and surface a coding TODO; never edit the product to match the
  docs (rule 8).
- **`DEPLOYMENT.md` is absent or deployment is UNCONFIGURED.** Stub or skip
  install/ops sections; note them as pending the deployment plan; don't invent
  topology.
- **Frontmatter `status:` or `documented-through:` is unexpected**
  (hand-edited, missing). Surface and ask how to recover.
- **A version needs pinning into a doc.** Flag it as possibly-stale, offer to
  confirm the current release, and let Jamie pick (rule 10).

---

## What this command does NOT do

- Does not own a status comment in `CLAUDE.md`. Per-file manifest frontmatter
  (`AWAITING-APPROVAL` / `ACTIVE` / `SUPERSEDED`) plus the derived currency are
  the source of truth.
- Does not edit product source code. Doc/code discrepancies are recorded as
  conformance gaps and surfaced as coding tasks (rule 8).
- Does not edit the internal SDLC docs (REQUIREMENTS / ARCHITECTURE /
  PROJECT_PLAN / CLAUDE_CODE_PROMPTS / design intake / PRDs) — it reads them.
- Does not edit `docs/DEPLOYMENT.md` — it reads it for install/ops docs and
  surfaces release-bundle changes for `/deployment-plan` to land.
- Does not clobber README sections owned by `/bootstrap` (Developer setup) or
  `/deployment-plan` (Deployment); it writes only the user-facing sections,
  by offer.
- Does not render or build the delivered docs — it writes the markdown and a
  product-type-aware delivery recipe; `/deployment-plan` owns the render/build.
- Does not run a release or gate one — it maintains the release-readiness
  ledger; a release process consumes it.
- Does not author doc sources in Stage 1 — the sign-off gate precedes the
  prose (rule 4).
- Does not run git commands (rule 7), tests (rule 8), or push to remotes.
- Does not run other commands automatically — it hands off via `TODO.txt`.
