# `/deployment-plan`

Plan how the project ships — topology, environments, deploy mechanism, secrets,
and CD posture — into `docs/DEPLOYMENT.md`. Deferrable.

**Type:** reference · **Primary reader:** template adopter

## Synopsis

```
/deployment-plan
```

No flags or arguments. One pass with two completion paths: a full plan or a
deferred stub.

## When to run

After [`/bootstrap`](bootstrap.md) is `COMPLETE`. This is the third and last
configuration command, and the most deferrable — it works best once some
development experience has accumulated, typically after Phase 0. You can defer it
indefinitely without blocking other work.

## What it does

**Full plan.** Reads the planning docs, the developer-environment plan, and the
design intake, then asks tailored questions (topology, per-environment
definitions, deploy mechanism, secrets and config across environments, CD
posture, rollback). It writes `docs/DEPLOYMENT.md` with sections for Topology,
Environments, Deploy mechanism, Secrets and config, Continuous deployment, and
Rollback, updates the README "Deployment" section with a summary and link, and
flips `DEPLOYMENT-PLAN-STATUS` to `COMPLETE <date>`.

**Deferred stub.** If you choose to defer, it writes a single-page
`docs/DEPLOYMENT.md` marked `Status: Deferred` with an open-questions list,
appends those questions to `docs/open-questions.md`, notes the deferral in the
README, and flips `DEPLOYMENT-PLAN-STATUS` to `DEFERRED <date>`.

**Documentation build.** When a `docs/documentation-plans/` documentation plan
exists, `/deployment-plan` owns building the delivered docs: it reads the plan's delivery
recipe and release-readiness ledger, codes the render targets against this
environment's runtime, and adds a pre-release doc gate. The doc *content* and the
recipe belong to [`/write-documentation`](write-documentation.md); the build
belongs here.

## Reads

`CLAUDE.md` (all three status comments), `REQUIREMENTS.md`, `ARCHITECTURE.md`,
`PROJECT_PLAN.md`, the design intake, the README (especially Developer setup),
`rules/environment-rules.md`, and the newest
`docs/documentation-plans/documentation-plan-*.md` if one exists.

## Writes / owns

`docs/DEPLOYMENT.md`, the README "Deployment" section, the deferred questions it
appends to `docs/open-questions.md`, and the `DEPLOYMENT-PLAN-STATUS` comment
(values `UNCONFIGURED`, `DEFERRED <date>`, `COMPLETE <date>`).

## Refuses when

- **`ONBOARD-STATUS` is not `COMPLETE`** — points at [`/onboard`](onboard.md).
- **`BOOTSTRAP-STATUS` is `UNCONFIGURED`** — points at [`/bootstrap`](bootstrap.md).
- **`BOOTSTRAP-STATUS` is `INSTRUCTIONS-WRITTEN`** — "Re-run `/bootstrap` to enter
  stage 2 (validation) first — deployment plan needs a validated dev
  environment."
- **The architecture doesn't constrain deployment yet** — it offers the deferred
  path rather than pressing you to decide.
- **The design's deployment intent contradicts the README setup** — it surfaces
  the conflict and asks you to resolve it first.
- **`docs/DEPLOYMENT.md` was hand-edited** — it shows the difference and asks per
  section.

## Does not do

Deploy anything, write CI workflows, provision infrastructure, edit
`rules/environment-rules.md` ([`/bootstrap`](bootstrap.md)'s domain), write
documentation *content* ([`/write-documentation`](write-documentation.md)), run
git, or run tests.

## See also

[`/bootstrap`](bootstrap.md) · [`/write-documentation`](write-documentation.md) ·
[Keeping a project up to date](../keeping-up-to-date.md)
