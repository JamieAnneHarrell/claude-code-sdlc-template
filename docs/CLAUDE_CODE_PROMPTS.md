# Claude Code prompts

One prompt per phase. Paste the prompt body into a Claude Code
session to run that phase. Each prompt has a "revisions since this
prompt ran" footer where deviations from the original plan
accumulate during the actual coding session.

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

**Revisions since this prompt ran:** none tracked.

---

## Prompt 2.1: `/refresh-from-repository`

**Before running this prompt:** pre-Phase-2.1 `/design-review` has
run per the PROJECT_PLAN.md first-class checkpoint. The
`CLAUDE.md` merge-strategy open question in
[`docs/open-questions.md`](open-questions.md) is resolved before
this prompt fires — that resolution is a load-bearing input here.

**Read first.**
- [`docs/REQUIREMENTS.md`](REQUIREMENTS.md) FR-13
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 2.1
- [`docs/design-decisions.md`](design-decisions.md) entries:
  "Downstream updates use a pull model, not push" and "Public
  GitHub URL for the source repo"
- [`docs/open-questions.md`](open-questions.md) "CLAUDE.md merge
  strategy for `/refresh-from-repository --merge`"
- The pre-Phase-2.1 design review checkpoint (its disposition log
  has the resolutions)
- Existing command files for reference patterns:
  [`.claude/commands/onboard.md`](../.claude/commands/onboard.md),
  [`.claude/commands/bootstrap.md`](../.claude/commands/bootstrap.md)

**Scope.**
1. Author
   `cc-template/.claude/commands/refresh-from-repository.md` as a
   two-stage self-modifying command.
2. Stage 1: selectively pull latest `.claude/commands/*.md` and
   `rules/*.md` from
   `github.com/JamieAnneHarrell/claude-code-sdlc-template`'s
   `cc-template/` subtree. Replaces the command files themselves,
   including this one.
3. Stage 2 (`--merge`): run the freshly-updated logic. Diff
   downstream `CLAUDE.md` and `rules/*.md` against the upstream
   `cc-template/` subtree. Apply the CLAUDE.md merge strategy
   decided in the design-review checkpoint. Preserve
   `<!-- ONBOARD-FILL: ... -->` content in rules files.
4. Implement refusal logic: if invoked at the source repo (detect
   by the presence of `docs/design/cc-template-product-spec.md`
   or equivalent source-only sentinel), refuse with a clear
   message.
5. Mirror the new command file to
   `.claude/commands/refresh-from-repository.md` at root so the
   source-of-truth project can also exercise it (subject to the
   refusal in step 4).
6. Update the dist's `cc-template/README.md` and root README to
   describe the command.

**Constraints (what NOT to do).**
- Do NOT push from upstream to known downstream paths — pull
  model only (per the "Downstream updates use a pull model"
  design decision, which is rejected-permanent under rule 3).
- Do NOT replace `<!-- ONBOARD-FILL: ... -->` content during
  `--merge`. That content is downstream-owned by definition.
- Do NOT add a configurable upstream URL unless the design review
  said to — bake in the
  `github.com/JamieAnneHarrell/claude-code-sdlc-template` URL per
  the design decision.
- Do NOT skip the test-downstream sandbox before declaring done.
  The blast radius of a broken `--merge` is "all downstream
  projects on next refresh."

**Exit criteria.**
- Sandbox downstream project (copy of an earlier `cc-template/`
  commit) runs both stages cleanly. Diff between downstream
  before-and-after shows upstream improvements landed and
  `ONBOARD-FILL` content preserved.
- Refusal triggers correctly when run inside the source repo.
- Manual `/exit-test-plan` walkthrough covers: happy path;
  upstream unreachable; merge conflict in `ONBOARD-FILL`-adjacent
  text; downstream that diverged from upstream beyond
  `ONBOARD-FILL` (e.g. consumer hand-edited a rules file
  outside the markers).

**Revisions since this prompt ran:** none tracked.

---

## Prompt 2.2: `/write-user-documentation`

**Before running this prompt:** this is a good place to run
`/design-review` to surface findings against the architecture as
it stands now — Phase 2.1's `/refresh-from-repository` is the
first network-touching command in the template, and Phase 2.2's
user-documentation command is the first content-generating one.
Worth a checkpoint before scope freezes.

**Read first.**
- [`docs/REQUIREMENTS.md`](REQUIREMENTS.md) FR-14
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 2.2
- Whichever existing command file has the closest shape (likely
  [`.claude/commands/onboard.md`](../.claude/commands/onboard.md)
  given it also produces user-facing artifacts)
- This project's design intake at
  [`docs/design/cc-template-product-spec.md`](design/cc-template-product-spec.md)
  — Prompt 2.2 will eat its own dog food by running on this
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
- Manual `/exit-test-plan` walkthrough confirms a fresh consumer
  could read the output and use the template without consulting
  internal docs.

**Revisions since this prompt ran:** none tracked.

---

## Prompt 3: Regression-test automation

**Before running this prompt:** this is a good place to run
`/design-review` to surface findings against the architecture as
it stands now — Phase 3 introduces tooling (a script that runs
diffs and parses invariants out of markdown), which is a
qualitative shift from the markdown-only stack to date.

**Read first.**
- [`docs/REQUIREMENTS.md`](REQUIREMENTS.md) FR-16
- [`docs/PROJECT_PLAN.md`](PROJECT_PLAN.md) Phase 3
- Root [`CLAUDE.md`](../CLAUDE.md) "Load-bearing invariants"
  section — the script's job is to detect drift in exactly these.
- [`rules/environment-rules.md`](../rules/environment-rules.md)
  "Project-specific environment" section — tooling for the script
  has to be consistent with what's documented there.

**Scope.**
1. Pick a baseline tag. Likely `dist-baseline-v0.1` capturing the
   current `cc-template/` state immediately after Phase 1's
   onboarding closes.
2. Author a small source-only script
   (`scripts/regression-check.ps1` or equivalent) that diffs
   current `cc-template/` against the baseline tag and flags:
   - Status comment renames (`ONBOARD-STATUS` → anything else)
   - `ONBOARD-FILL` marker drift
   - Zero-pad-width changes in `checkpoint-NNN` /
     `phase-NNN-exit` filenames or globs
   - Stage-detection placeholder text drift (`[PENDING]`,
     `> _[UNMARKED — ...]_`)
3. Document the check in root `CLAUDE.md` "Testing template
   changes" section as level 4 above the manual walkthrough.
4. Decide whether the script runs in CI or only locally before
   tagging a new dist release. (Open question until Phase 3
   starts — see PROJECT_PLAN.md scope note.)

**Constraints (what NOT to do).**
- Do NOT introduce a language runtime as a hard requirement —
  PowerShell is acceptable as it's the documented shell; pure
  bash is acceptable as Linux is supported; cross-platform either
  via parallel scripts or a tool that works on both.
- Do NOT extend the script to fix violations automatically.
  Detect, report, exit non-zero — let humans decide what to do.

**Exit criteria.**
- Running the check against current dist exits 0.
- Intentionally introducing each violation in a sandbox copy
  causes the check to exit non-zero with a clear message naming
  the violation.
- Documentation in root `CLAUDE.md` describes when to run it
  (before tagging a new dist release, at minimum).

**Revisions since this prompt ran:** none tracked.
