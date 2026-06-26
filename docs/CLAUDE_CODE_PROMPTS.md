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

Phases that completed before this file was written (Phase 0) are
recorded as historical references; their prompts were not authored
under this template's convention.

Reading order before any prompt: this project's
[`docs/REQUIREMENTS.md`](REQUIREMENTS.md),
[`docs/ARCHITECTURE.md`](ARCHITECTURE.md),
[`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md), root
[`CLAUDE.md`](../CLAUDE.md), and the rules files
([`rules/coding-session-rules.md`](../rules/coding-session-rules.md),
[`rules/design-philosophy-rules.md`](../rules/design-philosophy-rules.md),
[`rules/multi-agent-rules.md`](../rules/multi-agent-rules.md),
[`rules/project-rules.md`](../rules/project-rules.md),
[`rules/environment-rules.md`](../rules/environment-rules.md),
[`rules/testing-rules.md`](../rules/testing-rules.md)).

---

## Prompt 0: Source/dist restructure (historical, COMPLETE)

This phase predates the planning docs and was scoped from the
design intake at
[`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md).
Recorded here for completeness. No actionable prompt — the work is
already done.

**What landed.** Repository restructured into source-of-truth
root + `cc-template/` distributable. All six commands duplicated at
both locations. Rules and design philosophy duplicated where
universal. Root `CLAUDE.md` and `README.md` written as
template-improvement guides. Design-decisions records the
source/dist split rationale.

**Revisions since this prompt ran:** none tracked.

---

## Prompt 1: Onboard the source-of-truth (COMPLETE)

This prompt is the one currently running. The `/onboard` command
itself implements the work — it reads
[`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md),
produces these planning docs, fills rules `ONBOARD-FILL` blocks,
rewrites `rules/multi-agent-rules.md` to explore-plus-plan, writes
the README stub, and flips `ONBOARD-STATUS`.

**Read first.**
- [`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md)
- Root [`CLAUDE.md`](../CLAUDE.md) (the template-improvement guide
  preserved via the `CLAUDE.md.bak` merge step)
- [`docs/design-decisions.md`](design-decisions.md) (existing
  source/dist + license + pull-model decisions)
- [`docs/open-questions.md`](open-questions.md) (existing
  `/refresh-from-repository` open question and CLAUDE.md drift
  notes)

**Scope.**
1. Run `/onboard` against
   [`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md).
2. Confirm the four planning docs reflect the design intake and
   the design-decisions already on record.
3. Confirm dual licensing: root `LICENSE` is CC BY-NC-ND 4.0;
   `cc-template/LICENSE` is MIT.
4. Run `/bootstrap` next to plan the dev environment (PowerShell
   primary, no language runtime — most of `/bootstrap`'s work is
   confirming the documented shell and writing the README
   "Developer setup" section to reflect the no-runtime reality).
5. After Phase 1 closes, run `/design-review` for the
   post-onboarding sanity check (per PROJECT_PLAN.md checkpoint).

**Constraints (what NOT to do).**
- Do NOT overwrite `docs/design-decisions.md` or
  `docs/open-questions.md` wholesale — append entries, leave
  existing content intact.
- Do NOT lose template-improvement content from `CLAUDE.md` —
  the wholesale rewrite must be followed by a merge step that
  pulls "What this template is", "Load-bearing invariants",
  "Design principles for template changes", "Testing template
  changes", and "Migrating projects from older template versions"
  from `CLAUDE.md.bak`.
- Do NOT push to any remote until Jamie explicitly asks.

**Exit criteria.**
- `<!-- ONBOARD-STATUS: COMPLETE <date> -->` set in root
  `CLAUDE.md`.
- Four planning docs exist at `docs/REQUIREMENTS.md`,
  `docs/ARCHITECTURE.md`, `docs/PROJECT_PLAN.md`,
  `docs/CLAUDE_CODE_PROMPTS.md`.
- `rules/multi-agent-rules.md` describes explore-plus-plan.
- `<!-- ONBOARD-FILL: project-scope -->` block in
  `rules/project-rules.md` filled.
- README has the required stub sections (Developer setup,
  Deployment) with pointers to the right command.
- `CLAUDE.md.bak` merge complete; bak file deleted.
- Both `LICENSE` files in place.

**Revisions since this prompt ran:** none tracked.

---

## Prompt 1.1: `/exit-test-plan` BLOCKED disposition

**Before running this prompt:** Phase 1's post-onboarding
`/design-review` has landed (checkpoint 001). Phase 1.1 runs
first per the sequencing decided in that checkpoint (R4a);
Phase 1.2 (rules + CLAUDE.md cleanup) fires after Phase 1.1
closes and before Phase 2.x.

**Read first.**
- [`.claude/commands/exit-test-plan.md`](../.claude/commands/exit-test-plan.md)
  (current spec for Stage 1 / Stage 2, §4 / §6.N run log shape, §5
  disposition table)
- [`cc-template/.claude/commands/exit-test-plan.md`](../cc-template/.claude/commands/exit-test-plan.md)
  (dist mirror — must stay identical after the change)
- [`rules/testing-rules.md`](../rules/testing-rules.md) (the
  rules-file contract for §3 / §4 / §5)
- [`docs/REQUIREMENTS.md`](REQUIREMENTS.md) FR-15
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 1.1

**Scope.**
1. Decide the exact `BLOCKED` shape: is it `[BLOCKED]` (parallel
   to `[PENDING]`) in the run log, or a different marker?
2. Decide how Stage 2 walks a `BLOCKED` row — does it queue a
   cleanup coding pass, a `/design-review` checkpoint, or
   additional phases? Document the decision tree in
   `exit-test-plan.md` Step S2.
3. Decide where blocked items re-appear — automatically in the
   next §6.N addendum's test plan, or only on operator request?
4. Update `.claude/commands/exit-test-plan.md` with the new
   disposition. Mirror to
   `cc-template/.claude/commands/exit-test-plan.md`.
5. Update `rules/testing-rules.md` if the §3 / §4 / §5 contract
   changes shape.
6. Add a deviation footer entry to this prompt summarizing the
   final shape (since the spec inevitably evolves through
   implementation).

**Constraints (what NOT to do).**
- Do NOT change the existing `[PENDING]` placeholder string —
  stage detection depends on it (see NFR-4 in REQUIREMENTS.md).
  `[BLOCKED]` is an addition, not a replacement.
- Do NOT add `BLOCKED` to the configuration-status comment
  vocabulary — it's a per-row marker, not a per-document status.
- Do NOT collapse `BLOCKED` and `FAIL` — `FAIL` means the test
  ran and produced wrong output; `BLOCKED` means the test could
  not run at all.

**Exit criteria.**
- Both `exit-test-plan.md` copies (root and dist) describe
  `BLOCKED` identically and pass a copy-diff check.
- A re-read of both files end-to-end confirms the chain (Step 0
  refusal logic, Step S1.5, Step S2.x walk) handles `BLOCKED`
  consistently.
- Validated by `/exit-test-plan` Stage 1 on an artificial blocker
  case during the Phase 1.1 exit walkthrough.

**Revisions since this prompt ran:**
- 2026-05-24 — "Before running this prompt" reworded to reflect
  Phase 1.2 sequencing decided in design-review checkpoint 001
  (R4a). Prompt body unchanged.
- 2026-05-25 — Landed. Deviations:
  - Did not run `/exit-test-plan` Stage 1 against an artificial
    blocker case as the prompt's exit criteria specified. This
    project has no traditional test suite (NFR-6) and does not
    run `/exit-test-plan` against itself.
  - `rules/testing-rules.md` not edited — scope item 5 was
    conditional and the §3 / §4 / §5 contract didn't change
    shape.
- 2026-05-25 — Intersession friction-fix pass between Prompt 1.1
  landing and Prompt 1.2 starting (orthogonal to 1.1's scope;
  recorded here as the most-recently-landed prompt):
  - Rule 7: pre-flight re-read trigger wired into `/wind-down`
    Step 4, `/design-review` S1.8 + S2.7, `/exit-test-plan` S1.7
    + S2.7.
  - Rule 9: TODO.txt tightened (reference prompts by ID;
    immediate-next + blocking only); wind-down state-intent-then-
    edit pattern replaces pre-pasted approvals.
  - Rule 4 + KISS: simpler-alternative-in-same-message self-check.
  - Root + `cc-template/` CLAUDE.md: MUST-DO block emphasizing the
    rules files aren't auto-loaded.
  - `/onboard`: deviation-block written verbatim into seeded
    `docs/CLAUDE_CODE_PROMPTS.md`.
  - `docs/open-questions.md`: new OQ "Sharpen rule 7 commit-message
    brevity constraints" deferred to Phase 1.2 R4b.

---

## Prompt 1.2: Rules and CLAUDE.md cleanup pass

**Before running this prompt:** Phase 1.1 (BLOCKED disposition)
has landed. This prompt addresses checkpoint 001 findings R4a
(rules + CLAUDE.md context-size optimization), R4b (rule 7
commit-message guidance tightening), R4c (cc-template/CLAUDE.md
placeholder qualifier cleanup), and R2 (cc-template/CLAUDE.md
Reading-order entry for `docs/design/`). Must land before Phase
2.1 so the cleaner content ships in the dist first.

**Read first.**
- [`docs/design/design-review-checkpoint-001.md`](design/design-review-checkpoint-001.md)
  (findings R2, R4a, R4b, R4c and their dispositions)
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 1.2
- [`docs/REQUIREMENTS.md`](REQUIREMENTS.md) NFR-6 (no runtime),
  NFR-9 (source/dist duplication is deliberate)
- Root [`CLAUDE.md`](../CLAUDE.md) — every load-bearing
  invariant named in the "Load-bearing invariants" section must
  survive the cleanup
- [`cc-template/CLAUDE.md`](../cc-template/CLAUDE.md) — the
  pre-onboard placeholder; R4c and R2 edits land here
- All six rules files at root: `coding-session-rules.md`,
  `design-philosophy-rules.md`, `project-rules.md`,
  `multi-agent-rules.md`, `environment-rules.md`,
  `testing-rules.md`
- Global `~/.claude/CLAUDE.md` content to identify rules that
  overlap and could be trimmed with explicit "see global" notes

**Scope.**
1. **Baseline measurement.** Capture pre-cleanup line counts for
   each rules file, root `CLAUDE.md`, and `cc-template/CLAUDE.md`.
   Record in a working note (not committed) so the ≥25% reduction
   target is measurable.
2. **Classify content.** Walk each file section-by-section.
   Categorize: load-bearing (named in CLAUDE.md invariants or
   parsed literally by a command) → keep verbatim; content
   referenced by a command spec → keep but compress; rationale /
   anecdote / repeated guidance → candidate for compression or
   removal.
3. **Rule 7 rework (R4b).** Restructure
   `rules/coding-session-rules.md` rule 7 so brevity is
   foregrounded: (a) "subject + zero or one body sentence" is the
   default; (b) body bullets cite doc IDs (FR-N, NFR-N,
   ARCHITECTURE §N, Prompt N) over restating context;
   (c) multi-paragraph bodies are the rare case, not the default.
   Move the PowerShell-quoting mechanics into a "Mechanics
   reference" sub-section at the end, smaller font weight in the
   reader's eye.
4. **CLAUDE.md cleanup (R4c).** Apply OQ option A to
   `cc-template/CLAUDE.md`: drop the "(configured projects)"
   parenthetical from the Reading-order header; rephrase "filled
   in by `/bootstrap`" / "filled in by `/onboard`" qualifiers to
   read correctly pre- and post-onboard.
5. **Reading-order parity (R2).** Add a `docs/design/`
   Reading-order entry to `cc-template/CLAUDE.md` mirroring root
   `CLAUDE.md` item 5 ("design intake plus any `/design-review`
   checkpoints in flight").
6. **Global-overlap decision.** For each section of
   `rules/design-philosophy-rules.md` (and any other rules with
   significant global overlap), decide: keep self-contained
   (downstream consumers don't inherit Jamie's globals) or trim
   with explicit "see global `~/.claude/CLAUDE.md` for X" notes.
   Document the call inline in the rules file.
7. **Mirror to dist.** Every rules-file edit applies identically
   to root and `cc-template/`. CLAUDE.md edits are file-specific
   (root and dist diverge by design).
8. **OQ cleanup.** Mark the `placeholder qualifiers persist`
   entry in `docs/open-questions.md` as resolved (or move to
   `design-decisions.md`).

**Constraints (what NOT to do).**
- Do NOT remove or rename any load-bearing invariant named in
  root `CLAUDE.md` "Load-bearing invariants" section.
- Do NOT change the `[PENDING]` or
  `> _[UNMARKED — replace this line with your decision per the legend above]_`
  placeholder strings (NFR-4).
- Do NOT change any status comment vocabulary or
  `<!-- ONBOARD-FILL: ... -->` marker names.
- Do NOT collapse rule 7 to the point that the PowerShell
  quoting / HEREDOC guidance is lost — relegate, don't delete.
  The mechanics matter when commits land in the VSCode PowerShell
  terminal.
- Do NOT touch files outside the cleanup scope: design intake
  docs, REQUIREMENTS, ARCHITECTURE, PROJECT_PLAN, CLAUDE_CODE_PROMPTS,
  README. The cleanup is rules + CLAUDE.md only.

**Exit criteria.**
- Post-cleanup aggregate line count across rules + CLAUDE.md
  (root and dist) is ≥25% lower than the pre-cleanup baseline.
- Every load-bearing invariant named in root `CLAUDE.md` is
  still present and sourced.
- A spot-check of rule 7 produces a commit handoff that hits the
  new brevity bar.
- `cc-template/CLAUDE.md` reads correctly both as a pre-onboard
  placeholder and as the post-onboard CLAUDE.md template (no
  stale qualifiers).
- `cc-template/CLAUDE.md` Reading-order list includes the
  `docs/design/` entry.
- Validated by `/exit-test-plan` walkthrough covering: rule 7
  brevity bar on a sample commit handoff; sandbox copy of
  `cc-template/` reads correctly pre- and post-onboard.

**Revisions since this prompt ran:**

- LANDED 2026-05-25. Deviations: aggregate reduction -11.7% (-269
  lines), short of ≥25%; load-bearing content density was the
  limit. Validation was per `rules/testing-rules.md`
  "Project-specific tooling" — the `/exit-test-plan` bullet was a
  drafting carry-over (this project doesn't use that command).
- 2026-05-27 — Intersession cleanup (T2 from checkpoint 002
  Follow-up; recorded here as the most-recently-landed prompt):
  - `/onboard`'s spec retired the "Before running this prompt"
    header-block-note form (root + cc-template mirror).
  - Convention shift landed alongside: design reviews are now
    first-class numbered phases in PROJECT_PLAN.md and matching
    numbered prompts in CLAUDE_CODE_PROMPTS.md, sharing the
    surrounding minor-version numbering (no X.Y.5 sub-numbering;
    insert renumbers downstream). The between-phases marker form
    and the freestanding `## Design Review Checkpoint` block form
    are both retired with R5's header-block form. See
    `docs/design-decisions.md` "Design reviews are first-class
    phases and prompts".
  - This project's own PROJECT_PLAN.md and CLAUDE_CODE_PROMPTS.md
    were NOT retroactively reshaped — landed Phase 2.1 design
    review block stays as historical record.
- 2026-05-26 — Intersession cleanup between Prompt 1.2 landing and
  Prompt 2.1 starting (orthogonal to 1.2's scope; recorded here as
  the most-recently-landed prompt):
  - Prompt 2.1 rewritten ahead of its build session to drop
    two-stage framing, rule-3-restating "Do NOT" constraints, the
    `/exit-test-plan` exit criterion (NFR-6), and the
    "Before running this prompt" header-block.
  - Five new `docs/design-decisions.md` entries land the
    `/refresh-from-repository` contract (single-stage + drift
    staging; template-owned markers; source-mode sync; progressive-
    disclosure flags; vendored-lock-in pattern). FR-13 rewritten;
    NFR-4 extended.
  - `docs/open-questions.md` closes 4 entries (CLAUDE.md merge
    strategy, universal-rules customization partitions,
    source-only release helper, multi-agent-rules output shape)
    and adds 3 (reconciliation-algorithm choice, header-block
    pattern user story, plan-mode → wind-down user story).
  - Prompt 2.2 exit criterion's stray `/exit-test-plan` reference
    fixed (same NFR-6 carry-over).

---

## Prompt 1.3: root↔cc-template bring-forward

**Read first.**
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 1.3
- [`docs/design/design-review-checkpoint-003.md`](design/design-review-checkpoint-003.md)
  "Out-of-scope follow-ups / Cleanup bundle" list — the canonical
  record of what landed in `cc-template/` and where (architectural
  spine, R4-A1, R5, R6-A1, R7, N1, N3-A1, N4-A1, N5, N6, R11)
- [`docs/design-decisions.md`](design-decisions.md): "`/refresh-from-repository`
  consumer-boundary partition" (the CLAUDE.md template-owned vs
  consumer-owned map — which sections to bring forward), plus the
  recent "Rules-read reliability" and "`/wind-down` owns ...
  reconciliation" entries
- [`docs/REQUIREMENTS.md`](REQUIREMENTS.md) NFR-9 (source/dist
  duplication is deliberate; universal content stays identical)
- Both copies of each rules file, both `CLAUDE.md` files, and all six
  `.claude/commands/*.md` files in both command directories

**Scope.**
1. **Rules files (surgical).** For each `rules/*.md`, diff root vs
   `cc-template/rules/`. Bring forward `cc-template/`'s universal
   (template-owned) improvements to root; preserve root's
   `ONBOARD-FILL` block content (`project-rules.md` § project-scope,
   `environment-rules.md` § environment, `multi-agent-rules.md`'s
   explore-plus-plan content). The three fully-universal files
   (`coding-session-rules.md`, `design-philosophy-rules.md`,
   `testing-rules.md`) should end up identical root↔`cc-template/`.
2. **CLAUDE.md (surgical).** Diff root `CLAUDE.md` vs
   `cc-template/CLAUDE.md`. Bring forward improvements only to the
   template-owned sections — Collaboration rules and Reading order,
   per the checkpoint 002 R3 partition in `docs/design-decisions.md`
   "consumer-boundary partition" (rule summaries stripped per B1;
   rules-read directive foregrounded per B2-A1). Preserve root's
   project-specific sections verbatim: the banner, Project-specific
   context, Load-bearing invariants. Root `CLAUDE.md` is NOT a
   verbatim copy of the dist — the banner and invariants differ by
   design.
3. **Command files (hard-copy).** Pre-diff each of the six
   `.claude/commands/*.md`, root vs `cc-template/`. Flag any root
   content that never landed in `cc-template/` (root-ahead surprise)
   before overwriting. Then hard-copy
   `cc-template/.claude/commands/*.md` → root `.claude/commands/*.md`
   so all six are byte-identical. (`wind-down.md` was synced during
   the Phase-1.3-authoring session — expect a no-op there.)

**Constraints (what NOT to do).**
- Do NOT overwrite root's `ONBOARD-FILL` block content — those are
  this project's filled-in specifics, not template content.
- Do NOT overwrite root `CLAUDE.md`'s banner, Load-bearing
  invariants, or Project-specific context — only the template-owned
  Collaboration-rules + Reading-order sections get `cc-template/`
  improvements.
- Do NOT hard-copy the rules files — they carry `ONBOARD-FILL`
  divergence; rules are surgical, commands are hard-copy.
- Do NOT introduce `CC-TEMPLATE-BLOCK` markers — that is Phase 2.1's
  job. This phase aligns content only.
- Do NOT blindly overwrite a command file if the pre-diff shows
  root-ahead content — surface it and decide before copying.

**Exit criteria.**
- `coding-session-rules.md`, `design-philosophy-rules.md`,
  `testing-rules.md` identical root↔`cc-template/`.
- `project-rules.md`, `environment-rules.md`, `multi-agent-rules.md`
  match `cc-template/` on universal content; root `ONBOARD-FILL`
  blocks intact.
- All six `.claude/commands/*.md` byte-identical root↔`cc-template/`
  (verified by diff).
- Root `CLAUDE.md` Collaboration-rules + Reading-order reflect the
  `cc-template/` improvements; banner / invariants / context
  preserved.
- A re-read confirms this project's next session would load the
  cleaned-up rules from root.

**Revisions since this prompt ran:**

- LANDED 2026-06-02. Deviations:
  - `testing-rules.md` was grouped with the "fully-universal /
    identical" files, but it carries `/onboard`-appended content;
    treated like the fill-block files (universal portion 1–119 already
    identical; appended "Project-specific tooling" section preserved).
    No content lost.
  - `wind-down.md` was not the expected no-op — root was behind
    `cc-template/`'s `### CLAUDE.md` banner-drift block; the hard-copy
    resolved it.
  - Decision B (beyond strict bring-forward): `project-rules.md`
    "rule 1–9" → "1–10" fixed in **both** copies, since
    `coding-session-rules.md` now defines 10 rules.
  - Decision A: `multi-agent-rules.md` briefing rule adopted
    `cc-template/`'s terser form (in-scope choice).

---

## Prompt 2.1: `/refresh-from-repository`

**Read first.**
- [`docs/REQUIREMENTS.md`](REQUIREMENTS.md) FR-13, NFR-4
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 2.1
- [`docs/design-decisions.md`](design-decisions.md): the five
  refresh-from-repository entries (single-stage with skills-first
  staging; template marks its own content; source-repo refresh
  syncs cc-template/ → root; progressive-disclosure flags;
  vendored-template lock-in)
- [`docs/design/design-review-checkpoint-002.md`](design/design-review-checkpoint-002.md)
  (the pre-Phase-2.1 review — pinned algorithm, marker syntax,
  state file, R6 granularity, R3 partition, R2b conflict UX,
  R2d cross-file upgrade-offer, B1-A1 source-mode three-way)
- [`docs/open-questions.md`](open-questions.md) — verify the
  reconciliation-algorithm and "Before running this prompt"
  header-block entries have moved to `design-decisions.md` as
  closed (per checkpoint 002 follow-up TODOs); flag if not

**Scope.**
1. Author
   `cc-template/.claude/commands/refresh-from-repository.md`
   per FR-13. Single-stage by default; auto-detects merge-logic
   drift before pulling and stages skills-first when warranted,
   asking the consumer to re-invoke.
2. Use the template-marker syntax pinned by checkpoint 002:
   `<!-- CC-TEMPLATE-BLOCK: <id> --> ... <!-- /CC-TEMPLATE-BLOCK -->`
   with `<id>` a stable human-readable kebab-case identifier (no
   inline metadata; per-block hashes live in the state file).
3. Implement **Option D (hybrid)** reconciliation per checkpoint
   002: per-block content hash for inline-edit detection +
   file-level baseline reference for deletion-vs-never-existed
   disambiguation. Three-way comparison (downstream-current /
   upstream-current / baseline-from-state-file).
4. Wrap refresh-managed regions of rules files and (post-onboard)
   CLAUDE.md in the pinned markers. **Coarse-grained wrapping**
   (per checkpoint 002 R6): one TEMPLATE-BLOCK per top-level rule
   section, not per paragraph. Compact kebab-case ids (file-scoped,
   file prefix implicit). `ONBOARD-FILL` blocks remain as the
   inverse primitive — consumer-owned, refresh never touches.
   **CLAUDE.md merge partition** (per checkpoint 002 R3):
   Collaboration-rules + Reading-order sections wrap in
   TEMPLATE-BLOCK; banner / project-specific context / Load-bearing
   invariants stay consumer-owned (free regions, refresh never
   touches).
5. Create `.claude/claude-code-sdlc-template-refresh-state.md`
   per checkpoint 002 R4-A1 — single machine-managed markdown
   file with sections: Upstream baseline (last-synced-commit,
   last-synced-at), Block hashes (table of file → id → hash),
   Refresh logic version (drift detection integer), Upstream
   directives (force-skills-only-from). Refresh reads/writes;
   consumers don't hand-edit. Format sketch in checkpoint 002
   R4-A1's recommendation.
6. Source-mode behavior: if `cc-template/` exists as a subdir at
   cwd, treat it as upstream. Same code path serves the
   source-of-truth root↔dist sync and the vendored-template
   lock-in pattern. **Preserves three-way reconciliation
   semantics** (per checkpoint 002 B1-A1) — baseline reference
   for source-mode comes from the state file (subtree
   content-hash or source-repo HEAD at last sync). On first
   source-mode invocation in a repo, content-inspect the
   `cc-template/` subdir to confirm it's this template; cache
   determination in CLAUDE.md (per checkpoint 002 N1).
7. Flags: `--refresh-skills-only` (manual override for skills-
   first staging); `--no-claudemd` (skip CLAUDE.md from merge).
   Upstream-directive in the state file can also force
   `--refresh-skills-only` on next downstream run (per checkpoint
   002 R3 caveat).
8. **Inline-edit conflict UX** (per checkpoint 002 R2b): when
   refresh detects an inline-edit divergence in a TEMPLATE-BLOCK,
   surface inline to the running session, one block at a time,
   showing downstream-current vs upstream-current; ask the
   consumer to choose accept upstream / keep downstream /
   hand-merge. No sidecar conflict files; no git-style conflict
   markers.
9. **Cross-file migration upgrade-offer** (per checkpoint 002
   R2d): when a consumer-deleted block's hash matches a known
   template block in one of the six template-managed rules files
   (`coding-session-rules.md`, `design-philosophy-rules.md`,
   `multi-agent-rules.md`, `project-rules.md`,
   `environment-rules.md`, `testing-rules.md`), refresh detects
   the migration and offers to apply upstream's update.
10. Semantic conflict surfacing between upstream's new/changed
    blocks and downstream's personalized content is the executing
    Claude session's job — the command spec instructs the session
    to do this analysis directly, not via a heuristic engine.
11. Mirror to `.claude/commands/refresh-from-repository.md` at
    root.
12. Documentation: shipping `cc-template/README.md` covers the
    default invocation, both flags, and the vendored-lock-in
    pattern; root `README.md` adds a one-paragraph pointer.
13. After markers exist on-disk, update root `CLAUDE.md`
    "Load-bearing invariants" section to add the marker syntax
    and the state-file path alongside `ONBOARD-FILL`. (The
    markers are real at this point, not a future plan — pinning
    them is appropriate here, not earlier.)
14. **Pre-marker migration.** When `/refresh-from-repository` is
    invoked in a downstream project that has rules files but no
    `CC-TEMPLATE-BLOCK` markers (the consumer was onboarded
    pre-Phase-2.1), the command performs a one-time migration:
    insert coarse-grained markers (per checkpoint 002 R6) at
    top-level rule-section boundaries; seed
    `.claude/claude-code-sdlc-template-refresh-state.md` with
    per-block hashes computed from the consumer's current
    content; surface block-by-block any boundaries where the
    consumer's content diverges from upstream's block boundaries,
    using the standard inline-edit conflict UX (per scope item
    8). After migration, normal refresh applies. Per checkpoint
    003 N5 (HTML-comment content is stripped from Claude
    context-load before reading), the fine-vs-coarse-grained
    block decision is relaxed — human-only rationale prose can
    live in HTML comments at zero context cost, so block-boundary
    placement need not minimize comment overhead.

**Exit criteria.**
- Sandbox downstream (copy of an earlier `cc-template/` commit)
  runs default refresh cleanly: command files wholesale-replaced;
  rules and CLAUDE.md block-merged; `ONBOARD-FILL` content
  preserved; consumer-edited template blocks surface as
  divergences rather than silent overwrites; consumer-deleted
  blocks are not re-added.
- Refresh state file populated correctly across sandbox + source-
  mode + vendored runs.
- Source-mode invocation at this project's root syncs
  `cc-template/` up to root correctly with three-way semantics
  preserved.
- Drift auto-detection stages skills-first when locally-loaded
  refresh is meaningfully behind upstream's current refresh.
- `--refresh-skills-only` and `--no-claudemd` each work as named.
- Root `CLAUDE.md` Load-bearing invariants section reflects the
  new markers and state-file path by phase close.

**Revisions since this prompt ran:**

- 2026-05-26 — Scope and Read-first rewritten to reflect pinned
  decisions from checkpoint 002 (Round 1 + 1 addendum): Option D
  algorithm; CC-TEMPLATE-BLOCK marker syntax (id-only);
  `.claude/claude-code-sdlc-template-refresh-state.md` state file;
  coarse-grained TEMPLATE-BLOCK granularity (R6); CLAUDE.md merge
  partition (R3); inline-edit conflict UX (R2b); cross-file
  migration hash-upgrade-offer (R2d); source-mode three-way
  semantics (B1-A1); source-mode content-inspection caching (N1).
- 2026-05-29 — Scope item 14 added per checkpoint 003 R10:
  pre-marker migration for downstream projects onboarded before
  Phase 2.1 ships. Includes the N5 HTML-comment-stripping caveat
  that relaxes the fine-vs-coarse-grained block-boundary
  decision.
- 2026-06-04 — **Superseded by checkpoint 004 (B1 Option A).** This
  prompt's body describes the Option D (hash/baseline/state-file)
  mechanism as built; checkpoint 004 reframed reconciliation to the
  stateless marker-state + ask-once model. This run body is kept as
  the historical record (it already ran); the rewrite is specified
  by **Prompt 2.1.A** and tracked as Phase 2.1.A. Do not re-run this
  prompt.

---

## Prompt 2.1.A: `/refresh-from-repository` Option A rewrite

**Read first.**
- [`docs/REQUIREMENTS.md`](REQUIREMENTS.md) FR-13 (Option A), NFR-4
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 2.1.A
- [`docs/design/design-review-checkpoint-004.md`](design/design-review-checkpoint-004.md)
  — B1 (Option A), R1, N1, N2, N3, and the investigation section
- The as-built command
  [`cc-template/.claude/commands/refresh-from-repository.md`](../cc-template/.claude/commands/refresh-from-repository.md)
  (Option D — being rewritten)
- [`docs/open-questions.md`](open-questions.md) § Abandoned
  Approaches (once the Option-D entry is written)

**Scope.**
1. Rewrite the command to **stateless marker-state + ask-once**
   reconciliation per FR-13 / checkpoint 004 B1 Option A. Markers
   carry `template-owned` / `forked` / `removed` state; refresh is a
   two-way compare matched by id, asking once and recording the
   answer in-file; the executing session performs the surgical
   merge.
2. **Remove** the state file, per-block hashes, `git hash-object`
   recipe, `last-synced` reference, and baseline. Settle the
   `forked` / `removed` marker-state encoding (NFR-4 pins the state
   vocabulary; this build pins the exact syntax).
3. Make the `Refresh-logic-version` stamp the **sole** drift lever;
   remove `force-skills-only-from`, the `## Upstream directives`
   section, and the Step 7.1 clear (R1). Bump the stamp.
4. Converge source-mode and public-mode (disk vs network read only;
   no baseline fetch; public mode no longer needs git history).
   First refresh is a one-time guided yours/take/merge walk, then
   quiet.
5. Supersede the affected `design-decisions.md` entries; write the
   Abandoned Approaches entry for the Option D mechanism.
6. Update `cc-template/README.md` "Keeping a project up to date".
7. Mirror to root `.claude/commands/` via the source-mode dogfood
   (N2), after `/onboard`'s briefing-rule preservation is confirmed
   (N1 — landed at checkpoint 004 close).
8. After markers exist on-disk at root, pin the marker syntax in
   root `CLAUDE.md` Load-bearing invariants and close Phase 2.1.A.

**Exit criteria.** As Phase 2.1.A: source-mode dogfood runs the
ask-once migration cleanly with no state file; FR-13 (Option A)
satisfied; surviving 002 decisions intact.

**Revisions since this prompt ran:**

- 2026-06-04 — Block 1 landed (scope items 1–6: command rewrite + dist
  docs). Split into two sessions: Block 2 (scope items 7–8 — source-mode
  dogfood to root + root invariant pins + phase close) is next session.
  Deviations beyond the written scope:
  - Added a fetch→review→apply **security gate** (Step 2 of the
    command) after Jamie flagged the supply-chain risk of importing
    executable command files from a network upstream. Public mode does
    an adversarial review + change summary before any live write; source
    mode shows a change summary. New `design-decisions.md` entry.
  - Pinned the public-mode fetch as a **shallow clone**
    (`git clone --depth 1`) — checkpoint 004 left the fetch mechanism
    unspecified. New `design-decisions.md` entry.
  - Marker-state syntax pinned: `state=forked` / `state=removed` inline
    on the open marker; `template-owned` is the bare default;
    `removed` is a closerless tombstone.
  - Per Jamie's wind-down instruction, the two fully-superseded Option-D
    `design-decisions.md` entries (reconciliation mechanism,
    content-hashing) were **removed** to `open-questions.md` § Abandoned
    Approaches rather than left with supersession notes; the
    partially-superseded entries were rewritten to surviving content.
  - Stripped non-self-contained citations (checkpoint/finding IDs,
    memory slugs) from the shipped command per the
    distributable-self-contained rule.
  - Queued a "/design-review security-review lens" deferred user story.

- 2026-06-04 — Block 2 landed (scope items 7–8). The source-mode
  dogfood ran the Step 6 pre-marker migration at this repo's root: 30
  template-owned blocks inserted in sync; `briefing-rule` kept
  (`state=forked`); `collaboration-rules` took upstream (clickable
  links → plain code spans). A second refresh verified quiet (zero
  re-asks). Marker encoding pinned in ARCHITECTURE + REQUIREMENTS
  NFR-4. Deviations from written scope:
  - The CLAUDE.md marker-encoding pin (scope item 8) was **not**
    applied — Jamie ruled the byte-level encoding belongs with the
    owning skill + design docs, not duplicated into CLAUDE.md
    invariants. CLAUDE.md left clean; a source-mode identity machine
    note (`REFRESH-SOURCE-MODE-VERIFIED`) is the only CLAUDE.md
    addition.
  - Noted, not fixed: root files are CRLF, `cc-template/` is LF —
    pre-existing cross-tree EOL drift, queued for a future sweep.
  - Phase 2.1 marked SUPERSEDED; Phase 2.1.A COMPLETE.

---

## Prompt 2.3: `/write-user-documentation`

**Read first.**
- [`docs/REQUIREMENTS.md`](REQUIREMENTS.md) FR-14
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 2.3
- Whichever existing command file has the closest shape (likely
  [`.claude/commands/onboard.md`](../.claude/commands/onboard.md)
  given it also produces user-facing artifacts)
- This project's design intake at
  [`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md)
  — Prompt 2.3 will eat its own dog food by running on this
  project to produce user docs for the template itself.

**Scope.**
1. Spec the command's shape: what inputs does it read (design
   docs, REQUIREMENTS, ARCHITECTURE, the actual implementation)?
   What outputs does it produce — a README for end users? A
   `docs/USER_GUIDE.md`? Multiple files? Single file?
2. Decide whether it has stages (like `/bootstrap` write-then-
   verify) or runs once and emits.
3. Author
   `cc-template/.claude/commands/write-user-documentation.md`.
4. Mirror to `.claude/commands/write-user-documentation.md` at
   root.
5. Run the new command on the source-of-truth project to produce
   user documentation for the template itself. The dog-food output
   serves as the validation artifact.

**Constraints (what NOT to do).**
- Do NOT make the command depend on a language runtime — keep it
  consistent with the rest of the template (markdown-only).
- Do NOT overwrite existing user docs without asking — same
  pattern as `/onboard`'s "if file exists with non-template
  content, ask first".

**Exit criteria.**
- Command runs against this project and produces a user-facing
  README plus any supplementary docs (decided in step 1).
- The output reads as written for end-users (not developers) —
  no internal-jargon, references to invariants are absent or
  abstracted, etc.
- Sandbox-consumer dry-read confirms a fresh consumer could read
  the output and use the template without consulting internal docs.
  (`/exit-test-plan` is not used in this project per NFR-6.)

**Revisions since this prompt ran:**

- 2026-05-26 — Removed "Before running this prompt" header-block
  per checkpoint 002 R5 (retire the gate-assertion/next-session-
  reminder conflated pattern; rely on PROJECT_PLAN.md first-class
  checkpoint entries instead).
- 2026-06-25 — Command shipped as `/write-documentation` (renamed
  from `/write-user-documentation`); Block 1 (core skill + craft
  doctrine) built. Checkpoint 005 settled render/build ownership
  (Option A: writing + delivery recipe here, build at
  `/deployment-plan`). Follow-on work is Prompts 2.4–2.7; this
  prompt's body is the Block 1 record and is not rewritten.

---

## Prompt 2.4: Apply checkpoint 005 — render/build ownership

**Read first.**
- [`docs/design/design-review-checkpoint-005.md`](design/design-review-checkpoint-005.md)
  — finding B1 (Option A, Accepted with caveats) and its
  Disposition log row; the caveat on a product-type-**open**
  delivery recipe is load-bearing.
- [`docs/REQUIREMENTS.md`](REQUIREMENTS.md) FR-14 (already updated to
  the recipe-handoff model) and NFR-6 / NFR-8.
- [`docs/ARCHITECTURE.md`](ARCHITECTURE.md) manifest-shape and
  "No language runtime" passages (already updated).
- [`docs/design-decisions.md`](design-decisions.md) the
  `/write-documentation` entry's "render architecture under review"
  scope note (L1256–1263 at authoring) — the note to rewrite.

**Scope.**
1. Reshape `cc-template/.claude/commands/write-documentation.md`:
   remove the `--render` argument and Step S2.5 "Render"; convert the
   manifest's "Render configuration" section into a product-type-open
   **"Delivery recipe"** — what well-delivered docs look like for this
   product, where the skill researches or discovers the product type
   at run time (real-time research and/or conversational discovery
   with the developer), never a fixed type enum (SaaS / zip / library
   are illustrations only); drop the "single source of truth for how
   the project renders" claim; point render/build at `/deployment-plan`;
   keep the release-readiness ledger as the handoff signal.
2. Flip the root [`CLAUDE.md`](../CLAUDE.md) `/write-documentation`
   load-bearing invariant: render is no longer single-owned by
   `/write-documentation` — `/deployment-plan` owns the doc
   render/build, consuming the delivery recipe + ledger; align the
   render-single-owned and `/deployment-plan`-references-render lines
   accordingly. Mirror to `cc-template/CLAUDE.md` if the invariant is
   carried there.
3. Rewrite the `design-decisions.md` "render architecture under
   review" note to the surviving landed decision (Option A) — writing
   here, build at `/deployment-plan`, no runtime prescribed. Do not
   leave a "superseded" tombstone (the design-decisions ↔
   abandoned-approaches hygiene rule).
4. Correct the `open-questions.md` 2026-06-24 render story: remove the
   false claim that ds-zip-tax wrote a render-to-HTML script (it is
   single-target PDF), and the false assumption that this project
   already has a Python runtime (it does not — that was ds-zip-tax's).
   Migrate the now-resolved story to its home (a `design-decisions.md`
   record, or rewrite to the chosen architecture).

**Constraints (what NOT to do).**
- Do NOT name a runtime, engine, or toolchain in the delivery recipe
  or anywhere the template ships — NFR-6 forbids prescribing a runtime
  downstream; the recipe describes *what* good delivery is, never *how*
  to build it.
- Do NOT hand-edit the root `.claude/commands/write-documentation.md`
  copy — the reshaped command reaches root via the Phase 2.6
  `/refresh-from-repository` source-mode dogfood, not a manual copy.
- Do NOT alter already-run prompt or plan bodies — deviation notes
  only (checkpoint 005 R1).

**Exit criteria.**
- `cc-template/.claude/commands/write-documentation.md` has no
  `--render` and no Step S2.5, and its manifest section is a
  product-type-open "Delivery recipe" naming no toolchain.
- The root `CLAUDE.md` `/write-documentation` invariant reads
  "`/deployment-plan` owns the doc render/build, consuming the
  delivery recipe + ledger"; no doc still says
  `/write-documentation` single-owns the render.
- `design-decisions.md` records the landed Option A decision with no
  tombstone; `open-questions.md`'s render story is corrected and
  migrated.
- The root command copy is intentionally still the old shape until
  Phase 2.6 (refresh) propagates it.

**Revisions since this prompt ran:**

- LANDED 2026-06-25. Clean execution. Deviation: scope item 2's
  "mirror to `cc-template/CLAUDE.md`" was a no-op — that file does not
  carry the `/write-documentation` invariant.

---

## Prompt 2.5: Block 2 — ecosystem wiring

**Read first.**
- [`docs/design/design-review-checkpoint-005.md`](design/design-review-checkpoint-005.md)
  B1 — the `/deployment-plan` piece grew from *referencing* a render
  to *owning* the doc build.
- The three sibling commands in `cc-template/.claude/commands/`:
  `deployment-plan.md`, `wind-down.md`, `onboard.md`.
- Root [`CLAUDE.md`](../CLAUDE.md) "Load-bearing invariants" — the
  ownership map (NFR-8) the wiring must keep non-overlapping.

**Scope.** Edit the three sibling commands in `cc-template/` (root
copies follow in Phase 2.6):
1. `deployment-plan.md` — gains ownership of the doc render/build:
   when authoring `DEPLOYMENT.md`'s release procedure and a doc
   manifest exists, read its delivery recipe + ledger and code the
   render targets against the environment's runtime at release time;
   add a pre-release **doc-gate** (docs CURRENT + no OPEN conformance
   gaps + no unfilled required visuals); ship the built docs. It owns
   the build outright — there is no parallel render defined elsewhere.
2. `wind-down.md` — its coherence sweep respects
   `/write-documentation`'s README / CONTRIBUTING section ownership
   (route user-facing drift to the owner, never clobber inline), and
   surfaces stale docs / OPEN ledger items at session close.
3. `onboard.md` — when it decomposes a new movement, add a one-line
   reminder that user docs are now stale for the new movement → run
   `/write-documentation`.

**Constraints (what NOT to do).**
- Do NOT let `/deployment-plan` and `/write-documentation` both define
  a render — `/write-documentation` writes the recipe, `/deployment-plan`
  owns the build; keep ownership non-overlapping (NFR-8).
- Do NOT hand-edit root command copies — they arrive via Phase 2.6.
- Edit only the three sibling commands; `/product-visioning` needs no
  change.

**Exit criteria.**
- `deployment-plan.md` owns the doc build (reads recipe + ledger,
  codes targets at release time, gates the release); no doc claims a
  second render owner.
- `wind-down.md` routes user-facing-doc drift to `/write-documentation`
  and surfaces stale-doc / OPEN-ledger state at close.
- `onboard.md` reminds about stale docs on movement decomposition.
- The ownership map in root `CLAUDE.md` stays coherent (each file one
  owner).

**Revisions since this prompt ran:**

- LANDED 2026-06-25. Beyond written scope (per Jamie mid-session):
  added a one-line `/write-documentation` pointer to onboard's
  PROJECT_PLAN orientation paragraph (Step 4, inherited by the
  later-movement path), and queued a Deferred User Story in
  `open-questions.md` on `/onboard` capturing documentation intent so
  `/design-review` can review it.

---

## Prompt 2.6: Dogfood `/refresh-from-repository` (source mode)

**Read first.**
- [`.claude/commands/refresh-from-repository.md`](../.claude/commands/refresh-from-repository.md)
  — source-mode behavior.
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phases 2.4–2.5 (the command
  changes to propagate).

**Scope.**
1. Run `/refresh-from-repository` in source mode to propagate the
   Phase 2.4 / 2.5 command changes from `cc-template/.claude/commands/`
   to root `.claude/commands/` — the reshaped `write-documentation.md`
   plus the three reshaped siblings.
2. Review the download/diff before applying (the command's own
   fetch→review→apply discipline).
3. Run a second refresh and confirm it is quiet (no diffs).

**Constraints (what NOT to do).**
- Do NOT hand-copy files — the dogfood IS the propagation mechanism,
  and exercising it is part of the validation.

**Exit criteria.**
- Root `.claude/commands/` copies of `write-documentation.md`,
  `deployment-plan.md`, `wind-down.md`, `onboard.md` match
  `cc-template/`.
- A second refresh reports no changes.
- `/write-documentation` (reshaped) is now runnable at root — the
  precondition for Phase 2.7.

**Revisions since this prompt ran:**

- LANDED 2026-06-25.

---

## Prompt 2.7: Dogfood `/write-documentation` + close

**Read first.**
- [`.claude/commands/write-documentation.md`](../.claude/commands/write-documentation.md)
  — now present at root via Phase 2.6.
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 2.7.

**Scope.**
1. Run `/write-documentation` on this template (the source-of-truth
   project) to produce `docs/published/` — the numbered manifest, the
   audience-facing markdown sources, the delivery recipe, and the
   release-readiness ledger — documenting the template itself for its
   consumers.
2. Close out the write-documentation work in `PROJECT_PLAN.md` and
   `CLAUDE_CODE_PROMPTS.md`.

**Constraints (what NOT to do).**
- Do NOT build or render the docs here — this project has no runtime
  and `/deployment-plan` is `UNCONFIGURED`; markdown is the
  deliverable, and the build (when a project wants one) is
  `/deployment-plan`'s. The dogfood validates the *writing*.
- Do NOT invent install / topology sections — `DEPLOYMENT.md` does not
  exist here; stub or skip them (rule 8).

**Exit criteria.**
- `docs/published/` holds an `ACTIVE` manifest plus audience-facing
  markdown that reads as written for the template's consumers, with a
  product-type-open delivery recipe recorded.
- The write-documentation work is marked closed in the tracking docs.
- A sandbox-style dry-read confirms a consumer could use the output
  without the internal docs (`/exit-test-plan` is not used here per
  NFR-6).

---

## Prompt 2.8: Reshape the `/write-documentation` skill spec (cc-template)

**Read first.**
- [`docs/design/design-review-checkpoint-006.md`](design/design-review-checkpoint-006.md)
  — the reshape's findings + dispositions (B1/B2/B3/R1/R2-A1).
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 2.8.
- [`docs/REQUIREMENTS.md`](REQUIREMENTS.md) FR-14, FR-18.

**Scope.** Edit in `cc-template/` only (root propagates in Step 2.9):
1. `.claude/commands/write-documentation.md` — Step-0 glob →
   `docs/documentation-plans/`; read `docs/documentation-guidance.md` before
   routing; add the audience-folder derivation rule (`docs/<audience-slug>/`,
   slug per the voice-profile audiences, no `docs/guides/`); add the three
   Stage-2 modes incl. interactive **revise**; add the documentation-guidance
   subsection (FR-18, downstream-of-behavior boundary); add the "After Stage 2 —
   who owns the docs / how to iterate" section.
2. `.claude/commands/deployment-plan.md`, `wind-down.md`, `onboard.md` — manifest
   glob → `docs/documentation-plans/`; doc-source refs → audience folders;
   `wind-down` gains guidance capture + retirement; `onboard` gains the
   `docs/documentation-guidance.md` skeleton seeding + skeleton-overwrite-safe
   entry.
3. `cc-template/docs/documentation-guidance.md` — new shipped skeleton (purpose
   header + entry format + legend; no entries).

**Constraints (what NOT to do).**
- Do NOT edit the root copies — Step 2.9's source-mode refresh propagates them.
- Do NOT move the render/build boundary (checkpoint 005): `/write-documentation`
  writes markdown + delivery recipe; `/deployment-plan` builds.
- Keep the DOC DECISION placeholder string unchanged (NFR-4 stage detection).

**Exit criteria.**
- The four skill files + the skeleton are edited in `cc-template/`.
- The DOC DECISION placeholder is intact; ownership stays non-overlapping (NFR-8).

**Revisions since this prompt ran:**

- (none yet)

---

## Prompt 2.9: Propagate to root + root invariants

**Read first.**
- [`.claude/commands/refresh-from-repository.md`](../.claude/commands/refresh-from-repository.md)
  — source-mode behavior.
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 2.9.
- Root [`CLAUDE.md`](../CLAUDE.md) "Load-bearing invariants" — the
  `/write-documentation` block, Cross-cutting filename block, File-ownership list.

**Scope.**
1. Run `/refresh-from-repository` (source mode, review-before-apply) to propagate
   Step 2.8's reshaped skill files + the skeleton from `cc-template/` to root;
   run a second refresh and confirm it is quiet.
2. Update the root `CLAUDE.md` Load-bearing invariants (source-only; not
   refreshed): the `/write-documentation` lifecycle block (glob, guidance-store
   bullet, audience-folder bullet, three Stage-2 modes), the Cross-cutting
   filename block (manifest path; `documentation-guidance.md` durable-global, not
   zero-pad-N), the File-ownership list (`/onboard` creates guidance; `/wind-down`
   captures; `/write-documentation` reads/applies).

**Constraints (what NOT to do).**
- Do NOT hand-copy the command files — the source-mode refresh IS the propagation.
- Do NOT add a `cc-template/CLAUDE.md` invariant block — it carries no parallel
  Load-bearing-invariants section.

**Exit criteria.**
- Root `.claude/commands/` copies match `cc-template/`; the second refresh is quiet.
- The root invariants reflect the reshape.

**Revisions since this prompt ran:**

- (none yet)

---

## Prompt 2.10: Migrate live docs + reconcile + close

**Read first.**
- [`.claude/commands/write-documentation.md`](../.claude/commands/write-documentation.md)
  — reshaped at root via Step 2.9 (reconcile + revise modes).
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 2.10 + Phase 2.7.
- [`docs/design/design-review-checkpoint-006.md`](design/design-review-checkpoint-006.md)
  Disposition log (R3/R4/N1 reconciliation caveat).

**Scope.**
1. `git mv` `docs/published/` → `docs/user/` + `docs/maintainer/` +
   `docs/documentation-plans/` per each doc's Primary audience; remove the empty
   `docs/published/`. Create `docs/documentation-guidance.md` (the skeleton).
2. Re-stamp `documentation-plan-001` (paths + an authoring-log migration row);
   fix intra-doc relative links and the root `README.md` links.
3. Run a `/write-documentation` **reconciliation** pass to refresh the migrated
   docs (incl. the content-stale command-docs) to the reshaped skill behavior +
   phase/step terminology.
4. NFR-6 sandbox dry-read against the new `docs/user/` + `docs/maintainer/`
   layout; mark the write-documentation work closed in `PROJECT_PLAN.md` (Phase
   2.7 + 2.8–2.10) and this file.

**Constraints (what NOT to do).**
- Do NOT build/render — markdown is the deliverable; `/deployment-plan` is
  `UNCONFIGURED` (rule 8 / NFR-6).
- Do NOT leave any `docs/published/` reference except the frozen
  `design-review-checkpoint-005.md`.

**Exit criteria.**
- No stray `docs/published/` references; the audience layout reads as
  audience-facing.
- The dry-read confirms a consumer could use it without the internal docs.

**Revisions since this prompt ran:**

- (none yet)
