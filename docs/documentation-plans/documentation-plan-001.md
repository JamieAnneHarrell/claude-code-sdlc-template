---
documentation-plan: 001
project: claude-code-sdlc-template
created: 2026-06-25
status: ACTIVE
documented-through:
  movement: initial
  phase: "2.10"
---

# Documentation Plan 001 — claude-code-sdlc-template

## Context

**Product type.** `claude-code-sdlc-template` is a *Claude-Code-driven project
scaffold*, not a CLI, SaaS, library, or service. The product is a copyable
directory (`cc-template/`) of Claude Code **slash commands** plus curated
**collaboration rules**. An adopter installs it by copying `cc-template/` into a
new project, renaming it, and opening it in Claude Code; they "run" it by
invoking the slash commands (`/onboard`, `/bootstrap`, `/product-visioning`,
`/design-review`, `/exit-test-plan`, `/write-documentation`, `/deployment-plan`,
`/wind-down`, `/refresh-from-repository`) across the project's life. The product
surface is the command set and the SDLC lifecycle those commands orchestrate.

**Audiences (both in scope).**
- **Template adopter** — a Claude Code user seeding and running a new project
  from the template. Power-user register: comfortable with git, SDLC concepts,
  and Claude Code. *Primary.*
- **Template maintainer / contributor** — someone working on the
  source-of-truth repo itself (the source/dist split, the universal-content
  duplication, the load-bearing invariants). *Secondary.* The audience-facing
  maintainer track orients and routes; the authoritative internal detail stays
  in the source-only `docs/REQUIREMENTS.md` / `docs/ARCHITECTURE.md` /
  `CLAUDE.md`, which it links rather than duplicates.

**Documentation set this plan covers.** A canonical, detailed authored set filed
by audience — the adopter track under `docs/user/`, the one maintainer doc under
`docs/maintainer/` — plus in-place improvement of the two READMEs that orient and
route into it:
- Root `README.md` — the source-of-truth orientation README (its user-facing
  sections only; the `/bootstrap`-owned **Developer setup** and
  `/deployment-plan`-owned **Deployment** sections are left untouched).
- `cc-template/README.md` — the README that *ships* in the copied directory and
  reaches adopters.

**Deliberately NOT documented (minimalism, rule 4).**
- **API / SDK reference** — there is no public API (NFR-6, no runtime).
- **Admin / install / topology guide** — there is no server, service, or deploy
  topology to operate.
- **Release notes** — no versioned release cadence yet; revisit when one exists.
- **A standalone getting-started tutorial separate from the quick-start** — the
  surface has one happy path; a single quick-start serves it without a second
  learning-oriented doc.
- **`CONTRIBUTING.md`** — the source-only docs ship CC BY-NC-ND from a personal
  namespace; this is not an open-contribution project today. The maintainer
  track is served by a maintainer guide + the root README, not a PR-flow
  contributor doc. (Mark the maintainer-guide block `adjust` to add a
  `CONTRIBUTING.md` if that changes.)

**Currency.** First documentation pass, reconciled through phase `2.10` after the
audience-folder migration; movement `initial` (the project predates
`/product-visioning`, so there are no PRDs and no `PRODUCT_VISION.md`).

## How to mark up this plan

For each proposed doc below, replace the placeholder line under
`DOC DECISION — JAH:` with one of:

- `approve` — author it as proposed.
- `adjust: <note>` — author it with this change (re-scope, re-audience,
  re-outline).
- `drop` — do not author it.

To request a doc that isn't proposed, append a new doc block and mark it
`approve`. Then re-run `/write-documentation`; Stage 2 authors the approved set.
Mark each independently.

## Proposed documentation set

| Doc | Path | Primary audience | Mode(s) |
|-----|------|------------------|---------|
| Docs index / landing | docs/user/README.md | Adopter (+ maintainer) | orientation / routing |
| Quick-start | docs/user/quick-start.md | Adopter (new) | how-to (tutorial-leaning) |
| Command reference (per command) | docs/user/commands/ | Adopter (power user) | reference |
| The SDLC lifecycle | docs/user/lifecycle.md | Adopter | explanation + how-to |
| Keeping a project up to date | docs/user/keeping-up-to-date.md | Adopter (power user) | how-to |
| Troubleshooting | docs/user/troubleshooting.md | Adopter | how-to + reference |
| Maintaining the template | docs/maintainer/maintaining-the-template.md | Maintainer / contributor | explanation + how-to |
| Root README (in place) | README.md | Maintainer + adopter orientation | orientation / routing |
| Shipping README (in place) | cc-template/README.md | Adopter | orientation / routing + how-to |

### Docs index / landing — `docs/user/README.md`

- **Primary audience:** Adopter, with a labelled maintainer pointer (voice:
  lean, peer-to-peer).
- **Diátaxis mode(s):** orientation / routing (a landing page — names the
  audiences and routes to each doc; contains no deep content itself).
- **Job served:** a reader who lands in `docs/user/` (browsing the repo)
  needs to find the right doc for their job in one screen.
- **Outline:**
  - One-paragraph "what this folder is" *(orientation)*
  - "Start here" table: job → doc, split Adopter vs Maintainer *(routing)*
  - Pointer back to the two READMEs and the source-only internal docs
    *(routing)*
- **Visuals:** none.

> DOC DECISION — JAH:
> _approved_

### Quick-start — `docs/user/quick-start.md`

- **Primary audience:** Adopter, first run (voice: warm-but-lean; guarantees a
  first success).
- **Diátaxis mode(s):** how-to, tutorial-leaning (one ordered happy path that
  always succeeds; no branching, no theory).
- **Job served:** get from "I have the template" to "my project is onboarded and
  I'm in the loop" without reading everything first.
- **Outline:**
  - Before you begin: Claude Code installed; a design doc (or the intent to
    build one with `/product-visioning`) *(prerequisites)*
  - 1. Copy `cc-template/` into a new directory and rename it *(step)*
  - 2. Drop a design doc into `docs/design/` (or run `/product-visioning`)
    *(step)*
  - 3. Open in Claude Code; skim the rules *(step)*
  - 4. Run `/onboard` *(step, with what it produces)*
  - 5. Run `/design-review`, then `/bootstrap` *(step)*
  - 6. You're in the loop — pointer to the lifecycle doc *(next steps)*
- **Visuals:** none (text steps stand alone; a chat-session screenshot would
  age fast and show little — fails the decorative test).

> DOC DECISION — JAH:
> _adjust: skim the rules before running any skills. User may not agree with my design philosophy, give them the opportunity to orient themselves first._

### Command reference — `docs/user/commands/`

- **Primary audience:** Adopter, power user (voice: austere, factual; consulted
  not read).
- **Diátaxis mode(s):** reference (mirrors the product's own structure — one
  entry per command; no instruction, no narrative).
- **Job served:** "what does command X do, when do I run it, what does it
  produce, and what won't it do" — answered per command, authoritatively.
- **Outline (per JAH adjust — split into one document per command):**
  - `commands/README.md` — index with a "which command when" routing table.
  - One file per command (`commands/<name>.md`), each with a consistent shape
    adapted from the CLI-reference doctrine to slash commands: Synopsis
    (invocation + flags) · When to run · What it does (stages + artifacts) ·
    Reads · Writes/owns · Refuses when · Does not do · See also.
  - Files: `onboard`, `bootstrap`, `deployment-plan`, `product-visioning`,
    `design-review`, `exit-test-plan`, `write-documentation`, `wind-down`,
    `refresh-from-repository`.
- **Authoring note:** built from the **real command files** in
  `.claude/commands/`, not from memory — enumerate each command's actual flags
  and stages; flag any prose/command disagreement as a conformance gap.
- **Visuals:** none.

> DOC DECISION — JAH:
> _adjust: each command should have it's own document._

### The SDLC lifecycle — `docs/user/lifecycle.md`

- **Primary audience:** Adopter (voice: peer-to-peer, lightly opinionated).
- **Diátaxis mode(s):** explanation (the model and the *why*) + how-to (the
  recurring moves). Sections are single-mode; the conceptual model and the
  step-sequences are not blended within a section.
- **Job served:** understand the recurring loop a configured project settles
  into, and know which move comes next.
- **Outline:**
  - Configuration ritual vs recurring lifecycle *(explanation)*
  - Movements vs tactical work; the PRD → `/onboard` → `/design-review` loop
    *(explanation)*
  - Per-session: `/wind-down` *(how-to)*
  - Per-phase exit: `/exit-test-plan` *(how-to)*
  - High-risk transitions: `/design-review` checkpoints *(how-to)*
  - Milestones: `/product-visioning` → next movement *(how-to)*
  - Documentation currency: `/write-documentation` in the loop *(explanation)*
- **Visuals:** one inline lifecycle diagram of the movement loop, authored as a
  Mermaid/ASCII diagram in-markdown (not an image file — nothing to capture).
  Its content is also stated in prose.

> DOC DECISION — JAH:
> _adjust: This is a highly opinionated and structured SDLC. Provide a good introduction about WHY this exists and how it helps write good software. (Note the doc doesn't need to be highly opinionated, but it might help to intruduce the SDLC as such.)_

### Keeping a project up to date — `docs/user/keeping-up-to-date.md`

- **Primary audience:** Adopter, power user (voice: lean, precise).
- **Diátaxis mode(s):** how-to (the refresh task) + a thin reference slice (the
  marker-state vocabulary and flags).
- **Job served:** pull later upstream improvements into a seeded project without
  losing local customizations.
- **Outline:**
  - When and why to run `/refresh-from-repository` *(how-to)*
  - What it touches and what it never touches (`ONBOARD-FILL`, content outside
    `CC-TEMPLATE-BLOCK`) *(reference)*
  - The ask-once model: take upstream / keep mine (`forked`) / hand-merge;
    tombstones (`removed`) *(how-to + reference)*
  - The two flags: `--refresh-skills-only`, `--no-claudemd` *(reference)*
  - Vendored lock-in / source mode *(how-to)*
  - Bootstrapping refresh into a project that predates it *(how-to)*
- **Authoring note:** the strong existing "Keeping a project up to date"
  content in `cc-template/README.md` is the seed; this doc is the detailed home
  and the README routes to it.
- **Visuals:** none.

> DOC DECISION — JAH:
> _adjust: This is about keeping the SDLC *template* up to date. Frame as such._

### Troubleshooting — `docs/user/troubleshooting.md`

- **Primary audience:** Adopter (voice: terse, scannable).
- **Diátaxis mode(s):** how-to + reference. Each entry is Symptom → Cause → Fix
  (+ Prevention), symptom in the user's words including the verbatim message.
- **Job served:** recover fast when a command refuses or a stage doesn't behave
  as expected.
- **Outline (seed entries, drawn from real refusal modes / NFR-5):**
  - "`/bootstrap` refuses — onboarding isn't complete" *(refusal)*
  - "`/design-review` / `/exit-test-plan` re-ran but did the wrong stage"
    (marked vs unmarked placeholders) *(stage detection)*
  - "`/write-documentation` won't author — a doc is unmarked" *(refusal)*
  - "`/refresh-from-repository` keeps asking about a block I already own"
    (`forked` not recorded) *(refresh)*
  - "I edited a rule and a refresh offered to overwrite it" *(markers)*
  - placeholder for additional entries surfaced during authoring
- **Visuals:** none.

> DOC DECISION — JAH:
> _adjust: include a section about common drifts such as logging what was done instead of deviations in claude code prompts, TODO becoming a log or holder of user stories that aren't really blockers, rediscovery of environment, etc._

### Maintaining the template — `docs/maintainer/maintaining-the-template.md`

- **Primary audience:** Maintainer / contributor (voice: candid peer; honest
  trade-offs).
- **Diátaxis mode(s):** explanation (the architecture and *why*) + how-to (the
  safe-change workflow). Routes to the authoritative internal docs rather than
  duplicating them.
- **Job served:** orient someone working on the source-of-truth and let them
  change the template without breaking a load-bearing invariant.
- **Outline:**
  - The source/dist split and why it exists *(explanation, routes to
    ARCHITECTURE)*
  - Universal-content duplication discipline: edit in `cc-template/` first, copy
    up *(how-to)*
  - The two marker systems (`CC-TEMPLATE-BLOCK` / `ONBOARD-FILL`) at a working
    level *(explanation)*
  - The load-bearing invariant chain — where it lives, how to audit it when you
    touch a command *(how-to, routes to `CLAUDE.md`)*
  - Dogfooding: the source repo runs its own commands; exercise a change on the
    source before it ships *(how-to)*
  - "Don't add another command" bar; KISS / progressive disclosure for template
    changes *(explanation)*
- **Visuals:** none.

> DOC DECISION — JAH:
> _approved_

### Root README (in place) — `README.md`

- **Primary audience:** Maintainer + adopter orientation (voice: lean,
  orienting).
- **Diátaxis mode(s):** orientation / routing.
- **Job served:** the repo's front door — say what this is, route a maintainer
  to the work and an adopter to the quick-start and the published set.
- **Scope (edit in place, user-facing sections only):** overview, project shape,
  seeding, working-on-the-template, why-this-shape, usage, license, reference;
  **add routing into `docs/user/` + `docs/maintainer/`**. **Do not touch** the
  `/bootstrap`-owned *Developer setup* section or the `/deployment-plan`-owned
  *Deployment* section.
- **Visuals:** none.

> DOC DECISION — JAH:
> _approved_

### Shipping README (in place) — `cc-template/README.md`

- **Primary audience:** Adopter (voice: warm-but-lean; it travels in the copied
  directory and is often the first thing a new project's session reads).
- **Diátaxis mode(s):** orientation / routing + how-to (the seeding walkthrough).
- **Job served:** the README a freshly-seeded project opens with — orient the
  adopter and route them into the quick-start and the lifecycle.
- **Scope (edit in place):** tighten "How to use", "After setup: the recurring
  lifecycle", "Keeping a project up to date", and "What the template ships
  with"; route into the published set; align voice and terminology with it.
- **Known tension to record (not resolved here):** when an adopter runs
  `/onboard`, the consumer's README is rewritten — so improvements to the
  *shipping* `cc-template/README.md` reach the adopter only at seed time, before
  onboarding overwrites their copy. How the published set and the shipping
  README survive and travel is a delivery problem captured for `/deployment-plan`
  (see Delivery recipe).
- **Visuals:** none.

> DOC DECISION — JAH:
> _approved_

## Audience × feature exposure

Columns: **A** = Adopter, **M** = Maintainer / contributor. A check means the
feature is documented for that audience; a gated feature is not documented for
the audience it's hidden from.

| Feature / capability | A | M |
|---|---|---|
| Seeding a new project (copy / rename) | ✓ | ✓ |
| Configuration ritual (`/onboard`, `/bootstrap`, `/deployment-plan`) | ✓ | ✓ |
| Recurring commands (`/product-visioning`, `/design-review`, `/exit-test-plan`, `/wind-down`) | ✓ | ✓ |
| `/write-documentation` | ✓ | ✓ |
| Keeping a project current (`/refresh-from-repository`, flags, source mode) | ✓ | ✓ |
| Customizing the collaboration rules | ✓ | ✓ |
| Movement / SDLC lifecycle model | ✓ | ✓ |
| Source/dist architecture & duplication discipline | — | ✓ |
| Load-bearing invariants & marker internals | refresh markers only | ✓ |
| Dogfood-on-source workflow | — | ✓ |

## Delivery recipe

*What well-delivered docs look like for this product — recorded for
`/deployment-plan`, which owns the render/build and will code the targets against
whatever runtime exists at release time. Names no toolchain or runtime (NFR-6).*

- **Canonical detailed set** = the markdown under `docs/user/` and
  `docs/maintainer/`. It is authored and freshness-gated here, browsable as-is on
  the repo host. A future `/deployment-plan` decides the *delivered* form and how
  the audience doc set travels into a distribution — it is **source-only** today
  (FR-12) and does not copy into `cc-template/`, so adopters do not receive it
  by default. **Open delivery question for `/deployment-plan`:** how (and which
  subset of) the audience doc set reaches adopters.
- **Adopter-facing shipping surface** = `cc-template/README.md` (+
  `cc-template/docs/design/README.md`), which travels inside the copied
  directory. The root `README.md` is the source-of-truth orientation front door.
  Both READMEs are kept current by `/write-documentation`; `/wind-down` may also
  touch them, but `/write-documentation` is the freshness gate that makes them
  shippable at a release checkpoint.
- **Second open delivery question for `/deployment-plan`:** `cc-template/README.md`
  is rewritten when an adopter runs `/onboard`, so the shipping README only
  reaches them at seed time. The delivered docs strategy must account for the
  pre-seed vs post-onboard README lifecycle.
- **Build output:** none today — markdown is the deliverable; nothing is
  rendered or committed as build output until `/deployment-plan` defines it.

## Release-readiness ledger

*Stage 2 fills this. Outstanding items a release process may gate on; each is
OPEN or RESOLVED.*

| Type | Item | State | Detail |
|------|------|-------|--------|
| Stale | Root `README.md` "Developer setup" pins "Opus 4.7 (current latest as of 2026-05-24)" | OPEN | Rule-10 stale-version claim. That section is owned by `/bootstrap`; `/write-documentation` does not edit it. Re-run `/bootstrap` or correct the pin. |

No unfilled visuals — every authored doc is text-only. No conformance gaps — the
per-command reference was built from the real command files and agrees with them.

## Authoring log

*Stage 2 fills this. Append-only; one row per authoring round.*

| Round | Date | documented-through | Docs touched | Notes |
|-------|------|--------------------|--------------|-------|
| Original | 2026-06-25 | initial / 2.6 | 16 sources under `docs/published/` (index, quick-start, lifecycle, keeping-up-to-date, troubleshooting, maintaining-the-template, `commands/` index + 9 per-command docs); root `README.md` (added Documentation routing); `cc-template/README.md` (self-contained, `ds-*` fix) | First pass. Command reference split into per-command docs per JAH adjust. Shipping README kept self-contained — relative links into source-only `docs/published/` would break on seed; delivery routing recorded for `/deployment-plan`. |
| Reconcile 1 | 2026-06-26 | initial / 2.10 | Migrated the 16 sources from `docs/published/` to audience folders (`docs/user/`, `docs/user/commands/`, `docs/maintainer/`) and moved the manifest to `docs/documentation-plans/`; re-stamped all paths + currency. Refreshed the four reshaped command docs (`write-documentation`, `deployment-plan`, `wind-down`, `onboard`) to current behavior; fixed root `README.md` routing links. | Phase 2.10 reconcile after the Phase 2.8 skill reshape + Phase 2.9 root propagation: audience-folder filing, three Stage-2 modes (incl. revise), and `docs/documentation-guidance.md` ownership. Same movement — no new sign-off. |
