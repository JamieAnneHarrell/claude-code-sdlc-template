---
description: Plan dev/test/prod topology, deploy mechanism, and CD strategy for an onboarded and bootstrapped project — produce docs/DEPLOYMENT.md and update README.
---

# /deployment-plan

The **third** of three sequential configuration commands.
Prerequisite: `/bootstrap` has run.

`/deployment-plan` answers *how does this ship?* It reads the
planning docs, the dev environment plan from `/bootstrap`, and the
design intake; asks tailored questions about topology, deploy
mechanism, secrets across environments, CD posture, and rollback;
and produces:

- `docs/DEPLOYMENT.md` — the authoritative deployment plan, either
  full (if all questions were answered) or a stub recording the
  deferral and the open questions.
- An updated **Deployment** section in `README.md` with a
  one-paragraph summary and a link to `docs/DEPLOYMENT.md`.
- A `DEPLOYMENT-PLAN-STATUS: COMPLETE <date>` (or `DEFERRED <date>`)
  flip in `CLAUDE.md`.
- When a `docs/published/` documentation manifest exists, the
  **documentation build** within `docs/DEPLOYMENT.md`: the render
  targets — coded against this environment's runtime from
  `/write-documentation`'s **delivery recipe** — plus a pre-release
  doc-gate. `/deployment-plan` owns building the delivered docs;
  `/write-documentation` owns the doc *content* and the recipe.

This command is **deferrable**. Ideally it runs after some dev
experience has accumulated — typically after Phase 0, or once dev
is stable and the deployment questions have informed answers. There
is no hard cost to deferring.

This command runs in the current working directory and writes files
relative to it.

**Required reading before Step 0:** read
`rules/coding-session-rules.md`, `rules/design-philosophy-rules.md`,
and `rules/environment-rules.md` in full. The first two are the
universal rules; the commit-handoff format lives in `/wind-down`'s
Step 4 (rule 7 routes commit handoffs through `/wind-down`). The
third is what `/bootstrap` populated with shell, venv, and tooling
decisions — your deployment plan must be consistent with those.

---

## Step 0: Verify prerequisites

Read `CLAUDE.md` and check the three status comments:

- `<!-- ONBOARD-STATUS: ... -->` and `<!-- BOOTSTRAP-STATUS: ... -->`
  must both be `COMPLETE <date>`. If either is `UNCONFIGURED`,
  refuse and point at the missing command. If
  `BOOTSTRAP-STATUS` is `INSTRUCTIONS-WRITTEN <date>`, refuse and
  tell Jamie to re-run `/bootstrap` to enter stage 2 (validation)
  first — the deployment plan should only build on a validated
  dev environment. Bootstrap is the prerequisite because the
  deployment plan needs to know shell, secrets layout, and how
  the app runs locally — those answers shape every deployment
  decision.
- `<!-- DEPLOYMENT-PLAN-STATUS: ... -->`:
  - `UNCONFIGURED` → first run, proceed.
  - `DEFERRED <date>` → treat as "first real run." Proceed and
    overwrite the stub `DEPLOYMENT.md`.
  - `COMPLETE <date>` → ask Jamie whether to re-run (which will
    overwrite `docs/DEPLOYMENT.md` and the README Deployment
    section). Proceed only if she confirms.

## Step 1: Read project context

Load these in order:

1. `docs/REQUIREMENTS.md`
2. `docs/ARCHITECTURE.md`
3. `docs/PROJECT_PLAN.md`
4. Every file in `docs/design/` (other than the folder's own
   `README.md`)
5. `README.md` — particularly the Developer setup section, since
   the production deployment must be consistent with how the app
   actually runs in dev
6. `rules/environment-rules.md` — the project-specific section
   filled by `/bootstrap`
7. `docs/published/documentation-plan-*.md` (newest), if it exists —
   `/write-documentation`'s manifest. Its **delivery recipe** and
   **release-readiness ledger** drive the documentation build and the
   pre-release doc-gate (Step 5).

If a non-stub `docs/DEPLOYMENT.md` already exists (re-run case),
read it too and treat the existing structure as a starting point.

## Step 2: Reconnaissance

State back to Jamie a 3–4 bullet summary covering:
- What the architecture implies for deployment (single VPS? cloud
  service? container? serverless? client-side install? hybrid?)
- What the design doc says explicitly about deployment, vs. what's
  silent
- What's likely to drive the topology decision (compliance, cost,
  team size, expected scale)
- Any conflicts between the dev environment plan and what
  production deployment is likely to require — these need flagging
  early

Don't proceed until Jamie confirms.

## Step 3: Tailored questions

Ask via `AskUserQuestion`. Tailor to the architecture from Step 2 —
don't ask about Kubernetes for a static site, don't ask about CDNs
for a CLI tool.

**Version recency.** Before suggesting a version in any answer
back to Jamie or in any artifact, follow **Rule 10** (never assume
training-time versions are current) in
`rules/coding-session-rules.md` — surface the staleness caveat,
offer to look up current stable/LTS, wait for go-ahead before
`WebSearch`/`WebFetch`. This applies to base image tags
(`python:3.12-slim`, `node:20-alpine`), database engine versions
(`postgres:16`, `mysql:8.4`), cloud SDK versions, and any runtime
version repeated from the dev environment. It applies even if a
version is already pinned: ask whether the pin was intentional or a
training-time residual, and note the production target may move on a
different cadence than dev.

### Mandatory decision categories

**a. Topology.** One of:
- `dev → prod` — single non-dev environment, simplest case
- `dev → test → prod` — full three-tier
- `dev → prod, with test added later` — pragmatic deferred-test
- `defer everything` — write a stub, exit, status flips to
  `DEFERRED`

If Jamie picks `defer everything`, skip to Step 4 with deferred
mode.

**b. Per-environment definitions.** For each non-dev environment
in the chosen topology, ask:
- Where it runs (which provider/host/account)
- What data it uses (real prod data? scrubbed copy? synthetic?)
- Who has access (Jamie only? team? customers?)
- How it's provisioned (manual setup, Terraform, Pulumi, hosted
  managed service, etc.)

**c. Deploy mechanism.** Per non-dev environment:
- How code reaches it (manual command, GitHub Action, container
  push, push-to-deploy, etc.)
- What artifact is deployed (built binary, container image, source
  + run-on-target, etc.)
- Where the build happens (local, CI, on-target)

**d. Secrets and config across environments.** What differs from
the dev `.env` strategy:
- Where prod / test secrets live (managed service, encrypted
  store, ops handoff)
- How the app discovers them at runtime
- Rotation and access policy if relevant
- **Database credentials.** Follow NFR-X from `REQUIREMENTS.md`
  (or the "Database credentials policy" backstop in
  `rules/environment-rules.md` if the NFR is absent): prod and
  test app DB users are project-scoped non-privileged names —
  **never `root`, `admin`, `sa`, or any privileged default**. Any
  CI ephemeral DB user is a two-word-with-number pattern (e.g.
  `swift-otter-42`) that doesn't telegraph project identity.
  Confirm the prod app DB username with Jamie before it lands in
  `docs/DEPLOYMENT.md`.

**e. Continuous deployment posture.** (Skip if topology was
deferred.)
- Manual promotion at every step?
- Auto-deploy to test on green CI, manual to prod?
- Auto-deploy to prod on green CI?
- Hybrid (e.g. auto to test, manual canary to prod)?

**f. Rollback.** (Skip if deferred.)
- How a bad deploy gets reverted (redeploy previous artifact,
  feature flag, DB-aware rollback procedure)
- Rollback constraints (DB migrations one-way? blue/green
  available?)

## Step 4: Confirm answers

Read back to Jamie before writing artifacts:
- The topology
- Each environment's definition
- The deploy mechanism for each non-dev env
- Where secrets live in each environment
- CD posture and rollback approach

If anything is unclear or contradicts the architecture, surface
the conflict now — don't write it into `DEPLOYMENT.md`.

## Step 5: Generate artifacts

### `docs/DEPLOYMENT.md` (new, or rewrite)

**Version freshness checkpoint (write time).** Before any version
number lands in `docs/DEPLOYMENT.md` — base image tag, database
engine major, runtime version in deploy scripts, cloud SDK pin —
re-confirm per **Rule 10** (`rules/coding-session-rules.md`). Even
if Step 3 captured a version with Jamie's go-ahead, ask once more
before it goes into a file Jamie will commit. The Environments and
Deploy mechanism subsections are the version-heavy ones.

**If everything was answered:** write a full doc with these
sections, in this order:

1. **Topology** — which pattern, one-paragraph rationale.
2. **Environments** — one subsection per environment (Dev, Test if
   applicable, Prod). Each lists: where it runs, what data it
   uses, who has access, how it's provisioned.
3. **Deploy mechanism** — one subsection per non-dev environment,
   with the exact command or workflow that promotes code.
4. **Secrets and config** — strategy across environments, with
   pointers to the dev `.env` strategy in `README.md` for
   continuity.
5. **Continuous deployment** — current posture, what triggers
   auto-deploy where it applies, where manual gates exist.
6. **Rollback** — procedure per environment, constraints (DB
   migrations etc.).
7. **Documentation build** *(only when a `docs/published/`
   documentation manifest exists)* — how the shipped docs are built
   and gated at release time (see below).

**Documentation build (Section 7).** Include this section only when
`/write-documentation` has produced a `docs/published/` manifest.
`/deployment-plan` **owns building the delivered docs** — there is no
parallel render defined elsewhere. From the manifest's **delivery
recipe** (it names the target *forms*, never a toolchain) and the
runtime this deployment actually has, write:

- **Render targets** — the concrete build commands that turn the
  markdown sources into the delivered forms the recipe calls for (e.g.
  an HTML help site, a PDF in the release bundle), coded against this
  environment's runtime. The recipe prescribes no toolchain; you choose
  and pin one here, verifying current versions per **Rule 10** before it
  lands in the file.
- **Where built docs ship** — the output path and whether built output
  is committed, gitignored, or attached to the release artifact.
- **Pre-release doc-gate** — the release does not proceed unless the
  manifest is **CURRENT** (its `documented-through` matches the active
  movement/phase), the ledger has **no OPEN conformance gaps**, and **no
  required visual is unfilled**. A failing gate blocks the release and
  routes back to `/write-documentation`.

**If deferred:** write a single-page stub:

> # Deployment plan
>
> **Status:** Deferred (\<YYYY-MM-DD\>)
>
> Deployment topology and mechanism are not yet decided for this
> project. Re-run `/deployment-plan` when test/prod planning is
> needed (typically after Phase 0 or once dev experience has
> accumulated).
>
> ## Open questions
> \<bulleted list of the deferred decisions\>
>
> See also: `docs/open-questions.md` (these questions are also
> logged there).

Append the same questions to `docs/open-questions.md` under a
"Deployment" heading so they surface in normal review.

### `README.md` (modify)

Replace the placeholder "Deployment" section (the stub written by
`/onboard`) with one of two versions:

- **Completed:** one paragraph summarizing the topology and where
  to read details — "This project deploys via \<mechanism\> from
  \<source\> to \<target\>. See `docs/DEPLOYMENT.md` for full
  topology, secrets, and rollback details."
- **Deferred:** "Deployment plan is deferred. See
  `docs/DEPLOYMENT.md` for the open questions; re-run
  `/deployment-plan` when ready to decide."

Do not touch the Developer setup section — that is `/bootstrap`'s
domain.

## Step 6: Update CLAUDE.md

Rewrite `CLAUDE.md`:

1. Replace `<!-- DEPLOYMENT-PLAN-STATUS: UNCONFIGURED -->` (or
   `DEFERRED <date>` if a re-run) with one of:
   - `<!-- DEPLOYMENT-PLAN-STATUS: COMPLETE <YYYY-MM-DD> -->` if
     fully answered
   - `<!-- DEPLOYMENT-PLAN-STATUS: DEFERRED <YYYY-MM-DD> -->` if
     deferred

   Leave the other two status comments alone.

2. Update the post-bootstrap banner to include a Deployment line:
   - Completed: add `**Deployment:** see `docs/DEPLOYMENT.md`` to
     the orientation block, and update the lead sentence to drop
     the "`/deployment-plan` is deferrable" note.
   - Deferred: leave the lead sentence as-is (still deferrable),
     but add `**Deployment:** deferred — see
     `docs/DEPLOYMENT.md` for open questions`.

Keep the reading order, collaboration rules, and references
unchanged.

## Step 7: Surface git commands (rule 7)

Do not run any git commands.

```
git status
```

Show what to stage:

```
git add CLAUDE.md README.md docs/DEPLOYMENT.md docs/open-questions.md
```

```
git status
```

Then a single-block commit. Pick the message based on completion vs
deferral:

```
git commit -m "Deployment plan for <project name>

Topology: <topology choice or 'deferred'>
- <one-line summary of mechanism or deferred status>"
```

## Step 8: Final report

If completed:
- Files modified
- Topology and mechanism summary
- A pointer to `docs/DEPLOYMENT.md`

If deferred:
- The list of deferred questions (verbatim from the stub)
- A recommendation on when to re-run, e.g. "after Phase \<N\>
  when \<X\> is stable, or whenever you need to ship to a real
  user."

End with one of:
- "Deployment plan complete. The full project is configured."
- "Deployment plan deferred. Re-run `/deployment-plan` when ready."

---

## Failure modes

- **`/bootstrap` hasn't run.** Refuse and point at `/bootstrap`.
- **Architecture doesn't constrain deployment enough to ask
  meaningful questions** (e.g. "we'll figure out hosting later",
  no mention of where the app runs). Offer the deferred path
  explicitly rather than fishing for answers Jamie doesn't have
  yet. Don't pressure her into deciding now.
- **Deployment intent in design doc contradicts the README
  Developer setup.** For example: design says "containerized
  deploy" but Developer setup is "run `php artisan serve` from a
  WAMP install." Surface the conflict, ask Jamie to resolve before
  writing `DEPLOYMENT.md`. The deployment plan should be a
  promotion path from the dev environment, not a parallel universe.
- **Existing `docs/DEPLOYMENT.md` was hand-edited.** Surface the
  diff, ask per-section. Never silently overwrite.
- **Re-run on a project that's currently `DEFERRED`.** Treat as
  "first real run." Confirm what's changed since the deferral and
  ask all questions fresh.

---

## What this command does NOT do

- Does not actually deploy anything — it documents how to.
- Does not write CI workflows. The deployment plan describes the
  CD posture and what would auto-deploy where; the actual
  workflow files are written in a project phase (typically a
  late-stage hardening phase).
- Does not provision infrastructure. Provisioning steps are
  documented; running them is Jamie's call.
- Does not modify `rules/environment-rules.md` — that's
  `/bootstrap`'s domain. If a deployment decision implies a
  change to dev environment, surface it; don't reach across.
- Does not write documentation *content* or edit the
  `docs/published/` manifest and sources — `/write-documentation`
  owns those. `/deployment-plan` reads the delivery recipe + ledger
  and owns only the *build* of the delivered docs and the pre-release
  doc-gate (NFR-8 — one owner each).
- Does not run any git commands (rule 7).
- Does not run any tests (rule 8).
- Does not push to remotes.
- Does not run design reviews — `/design-review` does. By the time
  `/deployment-plan` runs (typically after Phase 0 and accumulated
  dev experience), design-review checkpoints are already part of
  the project's workflow; this command stays out of that domain.
