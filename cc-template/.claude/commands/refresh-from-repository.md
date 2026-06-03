---
description: Pull the latest commands and block-merge the latest rules + CLAUDE.md from the cc-template upstream into a downstream project, without re-onboarding. Source-mode syncs a local cc-template/ subdir to root.
---

# /refresh-from-repository

`/refresh-from-repository` updates a project that was seeded from
`cc-template` to the latest upstream template. It **wholesale-replaces**
the shipped slash commands and **block-merges** the universal rules
files and the template-owned sections of `CLAUDE.md`, while leaving
every consumer-personalized region untouched.

It is the pull half of the template's update model (the upstream never
pushes — see `docs/design-decisions.md` "Downstream updates use a pull
model, not push"). It runs **in the downstream project** and writes
files relative to the current working directory.

Two relationships use the same code path:

- **Public mode** — upstream is the public repo
  `github.com/JamieAnneHarrell/claude-code-sdlc-template`. This is the
  normal consumer case.
- **Source mode** — when a `cc-template/` subdirectory exists at the
  cwd, that local subdir *is* the upstream. This serves the
  source-of-truth repo's own root↔dist sync **and** the
  vendored-template-lock-in pattern (a consumer who vendors a pinned
  `cc-template/` copy into their repo). Same three-way algorithm;
  only the "fetch upstream / fetch baseline" steps read from disk
  instead of the network.

<!-- Refresh logic version: 1 -->
**Refresh logic version: 1**

> The integer above is the drift stamp (read by Step 1). Bump it only
> when a change would make an *older* locally-installed copy of this
> command mis-handle a *newer* upstream: marker open/close strings
> change, the state-file path or section names change, the hash recipe
> (Step 4) changes, or the reconciliation algorithm (Step 5) changes
> behavior. Cosmetic prose edits do **not** bump it.

**Required reading before Step 0:** read
`rules/coding-session-rules.md` and `rules/design-philosophy-rules.md`
in full. Rule 7 is load-bearing here — this command never commits;
it leaves a clean working tree for Jamie to review and routes the
commit through `/wind-down`.

---

## What gets refreshed, and what never does

Refresh manages three kinds of file:

1. **Slash commands** (`.claude/commands/*.md`) — wholesale-replaced
   from upstream every run. Commands carry no markers; they are not
   personalized.
2. **Rules files** (`rules/*.md`) and **`CLAUDE.md`** — block-merged.
   Only content inside a `<!-- CC-TEMPLATE-BLOCK: <id> -->` …
   `<!-- /CC-TEMPLATE-BLOCK -->` pair is template-owned and eligible
   for merge. Everything else is a **free region** refresh never
   touches.
3. **The refresh state file**
   (`.claude/claude-code-sdlc-template-refresh-state.md`) — machine
   state this command reads and writes. Consumers never hand-edit it.

Two consumer-owned primitives are **never** merged or overwritten:

- **`<!-- ONBOARD-FILL: <name> -->` … `<!-- /ONBOARD-FILL -->`
  blocks** — the inverse primitive. `/onboard` writes them; they hold
  project-specific content.
- **Free regions** — any text outside both marker kinds. In
  `CLAUDE.md` the free regions are the banner, status comments,
  `## Project-specific context`, and `## Load-bearing invariants`
  (per the consumer-boundary partition in `docs/design-decisions.md`);
  only `## Reading order at session start` and `## Collaboration
  rules` are template-owned.

Refresh identifies the template-owned region of each file by the
**block ids present in the upstream copy**, never by "everything
between the first and last marker." A consumer's unmarked sections
(e.g. a chosen multi-agent mode, an onboard-appended tooling tail)
are structurally safe.

---

## Arguments

`/refresh-from-repository` takes no positional arguments. The happy
path is the bare invocation: detect mode, detect drift, pull commands,
block-merge rules + CLAUDE.md.

Two progressive-disclosure flags exist for the cases that need them:

- `--refresh-skills-only` — pull `.claude/commands/*.md` only; attempt
  no merge. Use it to inspect upstream's new refresh logic before
  letting it touch your rules. This is also the path the drift
  auto-detection (Step 1) takes on your behalf.
- `--no-claudemd` — refresh commands and rules, but skip `CLAUDE.md`
  from the merge. For consumers who have heavily customized `CLAUDE.md`
  outside the marker model.

There is deliberately no third flag and no configurable upstream URL
(the URL is baked in; see `docs/design-decisions.md` "Public GitHub
URL").

---

## Step 0 — Detect mode and confirm the upstream

1. **Mode.** If a `cc-template/` subdirectory exists at the cwd →
   **source mode** (upstream = `./cc-template/`). Otherwise → **public
   mode** (upstream = the baked-in GitHub URL).

2. **Source-mode identity check (first time only).** On the first
   source-mode invocation in a given repo, content-inspect
   `./cc-template/` to confirm it is genuinely this template (it
   contains `cc-template/.claude/commands/` with the template's
   command set and `cc-template/rules/` with the universal rules — not
   an unrelated directory that happens to be named `cc-template`). If
   it is, record the determination in `CLAUDE.md` (a one-line
   machine note in a free region) so later invocations skip the
   re-check. If it is **not** this template, refuse: "`cc-template/`
   at cwd is not a claude-code-sdlc-template distribution; not
   treating it as upstream." (Per checkpoint 002 N1.)

3. **Confirm git is available.** This command relies on `git`
   (`git hash-object` for content hashing; git history for baseline
   content). If git is absent, refuse with a pointer to install it.

---

## Step 1 — Detect merge-logic drift (before pulling anything)

The locally-installed copy of *this* command may be older than
upstream's. An old copy must not try to interpret a newer marker
convention or state-file schema.

1. Read this command file's own **Refresh logic version** and the
   `## Refresh logic version` value in the state file (if present).
2. Fetch **upstream's** current `refresh-from-repository.md` (network
   for public mode, `./cc-template/.claude/commands/` for source mode)
   and read its Refresh logic version.
3. Read the state file's `## Upstream directives` →
   `force-skills-only-from`. An upstream commit can set this to force
   a skills-only pass on the next downstream run.
4. **If upstream's version is higher than the local version, or a
   force directive applies:** pull `.claude/commands/*.md` only (as in
   `--refresh-skills-only`), update the state file's refresh-logic
   version + clear the consumed directive, and stop with:
   "Your refresh logic was N version(s) behind upstream. The latest
   commands are now installed — re-invoke `/refresh-from-repository`
   to merge rules and CLAUDE.md with the updated logic." Do not
   attempt the merge this run.

This staging is automatic; the consumer never has to know to reach
for `--refresh-skills-only` (per `feedback_progressive_disclosure_escape_hatches`).

---

## Step 2 — Read the refresh state file

Path: `.claude/claude-code-sdlc-template-refresh-state.md`. Parse its
four sections:

- **`## Upstream baseline`** — `last-synced-commit` (the upstream
  commit, or local-subtree reference for source mode, whose content
  is the three-way baseline) and `last-synced-at`.
- **`## Block hashes`** — a table `| File | Block ID | Hash |` giving
  each template-owned block's hash **as of the last sync** (the
  baseline hash).
- **`## Refresh logic version`** — `local-refresh-logic-version`.
- **`## Upstream directives`** — `force-skills-only-from`.

State of the file determines the path:

- **State file present, rules files carry markers** → normal refresh
  (Steps 3–7).
- **No state file AND rules files carry no `CC-TEMPLATE-BLOCK`
  markers** → **pre-marker migration** (Step 6), then normal refresh.
- **No state file but markers exist** → treat as a re-seed: skip
  baseline comparison this run, seed the state file from current
  content as the new baseline (no merges applied, no conflicts
  surfaced), and report. The next run does a normal three-way merge.

---

## Step 3 — Fetch the three inputs

For each template-managed file, the reconciliation in Step 5 needs
three versions of every block:

- **downstream-current** — the file in this project (read directly).
- **upstream-current** — the file in upstream's current state
  (network fetch for public mode; `./cc-template/<path>` for source
  mode).
- **baseline** — upstream's content at `last-synced-commit`. Obtain
  it with `git show <last-synced-commit>:<upstream-path>` from
  upstream's history (the cloned upstream for public mode, the local
  repo for source mode). The stored per-block **baseline hash**
  (Step 2) is the cheap signal for "did this block change"; the full
  baseline content is only fetched when a hand-merge needs it.

Always pull `.claude/commands/*.md` wholesale from upstream-current
(Step 4) regardless of the merge outcome.

---

## Step 4 — Replace commands; compute block hashes

1. **Commands.** Overwrite every `.claude/commands/*.md` with
   upstream-current. (In source mode this is a root↔dist copy; in
   public mode a fetch.) If `--refresh-skills-only`, stop here, update
   the state file's refresh-logic version, and report.

2. **Hash recipe (deterministic, cross-platform).** To hash a block's
   content: take the text **between** its open and close markers
   (exclude the marker lines themselves), normalize all line endings
   to LF (`\n`), and pipe the result to `git hash-object --stdin`. The
   emitted object id is the block hash. This is reproducible on
   Windows, Linux, and macOS because the LF normalization removes the
   only platform-dependent byte difference before `git hash-object`
   sees the content. Use the same recipe everywhere hashes are
   compared or written.

---

## Step 5 — Reconcile (Option D hybrid, three-way, matched by id)

Match upstream blocks to downstream blocks by **`id`**, never by file
position — a consumer who reordered sections keeps their order
(checkpoint 002 R2c). For `CLAUDE.md`, skip this entire step if
`--no-claudemd`; otherwise reconcile only the `reading-order` and
`collaboration-rules` blocks.

For each upstream block id, compare hashes (baseline hash from the
state file; downstream and upstream hashes via Step 4's recipe):

- **Upstream unchanged** (upstream-current hash == baseline hash):
  nothing to push. Leave downstream as-is.
- **Upstream changed, downstream untouched** (downstream-current hash
  == baseline hash): apply upstream's new content cleanly.
- **Upstream changed, downstream also edited** (both differ from
  baseline): **inline-edit conflict** → resolve via Step 5a.
- **Block present upstream, absent downstream** (consumer deleted the
  whole marker pair): respect the deletion — do **not** re-add. Then
  run the cross-file migration check, Step 5b.
- **Block new in upstream** (id not in the state file and not in
  downstream): insert it, then run the semantic check, Step 5c.

After all blocks: any downstream block whose id is unknown to upstream
and unknown to the state file is a consumer-authored block in the
template region — leave it untouched and note it in the report,
suggesting `--refresh-skills-only` if many appear (they may indicate
the local logic is behind; checkpoint 002 R2a).

### Step 5a — Inline-edit conflict UX

Surface conflicts **inline to this session, one block at a time**.
For each: show the block id, the downstream-current content, and the
upstream-current content side by side, and ask Jamie to choose:

- **accept upstream** — replace the block with upstream's version
  (the local edit is lost);
- **keep downstream** — skip this block this run (no upstream update
  applied);
- **hand-merge** — fetch the full baseline content, show all three,
  and edit the resolved block together in the session.

No sidecar conflict files. No git-style `<<<<<<<` conflict markers in
the `.md` files — consumers read these files every session and should
never have to learn merge syntax (checkpoint 002 R2b).

### Step 5b — Cross-file migration upgrade-offer

When a block was deleted downstream, check whether its **baseline
hash** matches a known template block in **another** of the six
template-managed rules files (`coding-session-rules.md`,
`design-philosophy-rules.md`, `multi-agent-rules.md`,
`project-rules.md`, `environment-rules.md`, `testing-rules.md`). A
match means the consumer likely moved that rule into a personalized
file. If upstream changed that block, **offer** to apply the update
to wherever the consumer moved it (locate the migrated copy, or ask).
If they decline, respect it. Do not track arbitrary cross-file moves
beyond these six files (checkpoint 002 R2d).

### Step 5c — Semantic conflict surfacing

When upstream introduces a new block or substantially rewrites one
that lands next to the consumer's personalized content, **read both
and judge** whether they semantically conflict (e.g. upstream's new
rule says "always X"; a consumer rule says "never X"). Surface real
conflicts to Jamie in your own words and ask how to resolve. This is
your job as the executing session using language understanding — the
command does **not** delegate it to a string-matching heuristic
(per `docs/design-decisions.md` "Template marks its own content" and
`feedback_ai_not_heuristic`).

---

## Step 6 — Pre-marker migration (one-time)

Triggered when rules files exist but carry **no** `CC-TEMPLATE-BLOCK`
markers (the project was onboarded before Phase 2.1 shipped, or has
never refreshed). This includes the source-of-truth repo's own root
the first time refresh runs there.

1. For each of the six template-managed rules files and (unless
   `--no-claudemd`) `CLAUDE.md`, align the downstream's current
   sections against **upstream's marked copy**, matching by section
   heading / content. Insert the upstream block's open/close markers
   (with its id) around the corresponding downstream section. Use
   coarse-grained boundaries exactly as upstream defines them — one
   block per top-level rule section.
2. Where a downstream section's content **diverges** from upstream's
   block at the same boundary, surface it via the Step 5a inline-edit
   UX (the consumer may have edited a rule before markers existed).
3. Leave consumer-only sections (no upstream counterpart — e.g. a
   chosen multi-agent mode, onboard-appended tooling, `ONBOARD-FILL`
   regions) **unmarked and untouched**.
4. Seed `.claude/claude-code-sdlc-template-refresh-state.md` with the
   per-block hashes computed from the now-marked content as the
   baseline, `last-synced-commit` = upstream's current commit, and
   `local-refresh-logic-version` = this command's version.
5. Report what was marked and what diverged. Normal refresh
   (Steps 3–5) then applies on this and every later run.

Human-only rationale may live in single-line HTML comments at zero
context cost (the preprocessor strips them before Claude reads the
file as context, but this command reads raw file content and still
sees the markers — see `rules/environment-rules.md` "HTML comments
are free").

---

## Step 7 — Write the state file and report

1. Rewrite `.claude/claude-code-sdlc-template-refresh-state.md`:
   updated `## Block hashes` (recomputed for every template-owned
   block in its post-merge state), `## Upstream baseline` with
   upstream's current commit + today's date, the current
   `local-refresh-logic-version`, and an emptied
   `force-skills-only-from` if its directive was consumed.
2. **Report** to Jamie: commands replaced; blocks applied / kept /
   hand-merged / skipped; deletions respected; migrations offered;
   any unknown local blocks; and whether `CLAUDE.md` was included.
3. **Do not commit.** Leave the working tree for Jamie to review and
   route the commit through `/wind-down` (rule 7). If this refresh
   was a session-closing action, suggest `/wind-down`.

---

## Failure modes

- **`git` not available.** Refuse; point at installing git (Step 0.3).
- **Public mode, upstream unreachable** (offline, URL down). Refuse
  before changing any file; write no partial state.
- **Source mode, `cc-template/` at cwd is not this template.** Refuse
  (Step 0.2); do not treat an unrelated directory as upstream.
- **State file present but malformed / hand-edited** (missing a
  section, unparseable hash table). Refuse and explain the file is
  machine-managed; offer to re-seed from current content.
- **Merge-logic drift detected.** Not a failure — Step 1 stages
  commands-only and asks for a re-invoke. Never merge with stale
  logic.
- **An `ONBOARD-FILL` region overlaps or nests a `CC-TEMPLATE-BLOCK`
  marker.** Refuse to merge that file and surface it — the two
  primitives must never overlap; a file is partitioned into
  template-owned, ONBOARD-FILL, and free regions.
- **Ambiguous block boundary during pre-marker migration** (a
  downstream section can't be aligned to any upstream block). Surface
  it; do not guess a boundary. Mark what is unambiguous and ask about
  the rest.

---

## What this command does NOT do

- Does not run `git commit`, `git push`, or `git tag` (rule 7).
  Routes the commit through `/wind-down`.
- Does not run tests (rule 8).
- Does not edit `ONBOARD-FILL` regions or any free region — including
  the `CLAUDE.md` banner, status comments, `## Project-specific
  context`, and `## Load-bearing invariants`. Only template-owned
  blocks merge; `--no-claudemd` skips `CLAUDE.md` entirely.
- Does not re-add a block the consumer deleted (the absence is the
  consumer's decision), except via the explicit cross-file migration
  offer in Step 5b.
- Does not reorder a consumer's sections — blocks match by id, not
  position.
- Does not write git-style conflict markers or sidecar conflict files.
- Does not take a configurable upstream URL (baked in) and does not
  push upstream — the model is pull-only.
- Does not target the source from the source as a network upstream:
  in the source-of-truth repo it runs in source mode against the
  local `cc-template/`, never fetching its own GitHub copy.
- Does not own a status comment in `CLAUDE.md`. Its state lives in
  `.claude/claude-code-sdlc-template-refresh-state.md`.
