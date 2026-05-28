---
checkpoint: 003
date: 2026-05-27
reviewer: Claude (Opus 4.7) with Jamie
status: AWAITING-DECISIONS
trigger: Pre-Phase-2.1 rules-clean checkpoint — clean cc-template before /refresh-from-repository wraps it
---

# Design Review — Checkpoint 003

## Context

This checkpoint cleans the `cc-template/` rules surface, `cc-template/CLAUDE.md`, and the onboarding seed **before** Phase 2.1 wraps refresh-managed regions in `<!-- CC-TEMPLATE-BLOCK -->` markers. Once wrapping lands, drift in the wrapped content propagates to every downstream consumer who runs `/refresh-from-repository`, and reshaping wrapped content forces inline-edit conflict UX on every consumer who's already accepted the shape. The mission of this review is: every drift-prone decision the template currently invites should either be fixed in `cc-template/` now, or explicitly deferred with a concrete re-open trigger.

Scope is **`cc-template/` and this project's planning docs**. Project-root `rules/*.md` and root `CLAUDE.md` are out of scope — they catch up via `/refresh-from-repository` once Phase 2.1 ships. Where this review cites drift in the root files, it does so as **diagnostic evidence** of where `cc-template/`'s constraints were too loose; the remedy is a tighter `cc-template/`, not a root patch.

The review consolidates five deferred user stories from [`docs/open-questions.md`](../open-questions.md) into actionable dispositions:

- *Trim CLAUDE.md* (open-questions.md lines 315–373) — three independently-decidable threads split into B1, R6, and R7.
- *Rules-read reliably happen* (lines 138–244) — Directions 1+2 are nearly free and land here (B1, B2); Directions 3+4 defer with concrete re-open triggers (N3).
- *Seed cc-template/TODO.txt as the active onboarding checklist* (lines 375–434) — lands here (R9).
- *Artifact-boundary command landings → /wind-down* (lines 49–136) — defers explicitly with reasoning (N4).
- *Universal-rules customization partitioning* (TODO.txt JAH NOTE) — folds into the Phase 2.1 prompt scope (R10).

It also closes two checkpoint-001 deferrals that named "a future `/design-review` pre-Phase-2.1 or pre-Phase-2.2" as their re-open trigger — N1 (in-flight artifact status callout) and N2 (project quick orientation). Both are revisited here (R8 and N2 in this checkpoint's numbering, respectively).

Prior checkpoints whose decisions shape this review's edges:

- **Checkpoint 002 R6** bounds marker-overhead at ~5–7% of post-Phase-1.2 baseline; every compression finding here buys back budget for the upcoming wrap. **R3** partitions `cc-template/CLAUDE.md`: Collaboration-rules + Reading-order are template-owned (wrapped); banner / project-specific / Load-bearing invariants are consumer-owned (free regions). Findings on the template-owned regions are wrap-locked (B1, B2); findings on the consumer-owned banner are not strictly locked but every newly-onboarded project inherits whatever shape ships in the seed.
- **Phase 1.2** landed at -11.7% reduction against a -25% target because "load-bearing content density was the limit." This checkpoint reopens that ceiling with **structural** moves rather than line-by-line trimming: migrating per-doc guidance to the command file that owns it, merging the three restatements of rule 7's pre-commit sequence, and stripping false-prior rule summaries.

Jamie's framing constraints for this review:

- **Be sparing in the Blocker tier.** Only true Phase-2.1 show-stoppers belong there — decisions that wrap will lock in and that reshaping later would force conflict UX on every consumer.
- **Future-Claude perspective in a downstream context.** Findings must read the rules as a fresh Claude in a brand-new `cc-template/`-seeded project would — no project memory, no learned conventions, the rules teach behavior on first contact.
- **Name the simpler alternative explicitly (rule 4).** Every option set includes "leave it, no change" as a real choice. Don't recommend a change without naming what doing nothing costs.
- **Don't pre-decide.** Each finding presents options with downstream constraints and the tradeoffs; Jamie picks via `AUDIT NOTE — JAH:` inline.

The gate this review unblocks is **Phase 2.1**: authoring [`cc-template/.claude/commands/refresh-from-repository.md`](../../cc-template/.claude/commands/refresh-from-repository.md), wrapping `cc-template/rules/*.md` and `cc-template/CLAUDE.md` in template markers, and shipping the refresh logic that becomes load-bearing per NFR-4.

## How to mark up this review

For each finding below, replace the placeholder line under
`AUDIT NOTE — JAH:` with one of:

- `Accepted`
- `Accepted with caveats: <your caveats>`
- `Defer Approved`
- `DECISION: <explicit choice>`

Each finding has its own AUDIT NOTE block — mark each
independently. Inline detail, caveats, and rationale are welcome
alongside the decision keyword (the keyword is what Stage 2 keys
off; the surrounding text is the audit trail).

Then re-run `/design-review`. Stage 2 will walk each marked
disposition with you, append rows to the Disposition log, and ask
whether to **land the document** or **open another addendum
round** (e.g. when a marking asks for further investigation
before the disposition is final).

**Do not fill the Sign-off Summary table at the bottom** — Stage 2
fills that when the doc lands.

## Findings

### Blockers — fix before Phase 2.1's CC-TEMPLATE-BLOCK wrap

Two blockers. Each is a decision that Phase 2.1 will wrap in template markers in [`cc-template/CLAUDE.md`](../../cc-template/CLAUDE.md)'s Collaboration-rules section. Whatever shape lands here becomes the wrapped shape; reshaping post-wrap forces inline-edit conflict UX on every consumer who's accepted the shape.

#### B1. Rule-topical summaries in `cc-template/CLAUDE.md` Collaboration-rules section

**Sources.** [`cc-template/CLAUDE.md` lines 68–93](../../cc-template/CLAUDE.md#L68-L93) (Collaboration-rules section). [`docs/open-questions.md` "Trim CLAUDE.md" lines 340–346](../open-questions.md#L340-L346) (drift category 3 — "Rule summaries in the Collaboration-rules section"). [`docs/open-questions.md` "Rules-read reliability" Direction 2 lines 177–186](../open-questions.md#L177-L186) ("Strip rule summaries from CLAUDE.md, move drift-phrases into the rule files themselves"). Checkpoint 002 R3 partition: Collaboration-rules section is template-owned and will be wrapped (Sign-off Summary, `R3 / R3-A1`). Phase 2.1 PROJECT_PLAN.md scope item 4 (lines 211–219).

**Problem.** The Collaboration-rules section currently summarizes the 9 rules by topic in a parenthetical (lines 74–78) — "(root-cause, trust diagnosis, rejections permanent, no unsolicited design, decouple data from display, reference rule numbers, Jamie runs commits, Jamie runs tests, session wind-down rewrites TODO.txt)" — plus a "Includes the rule-7 commit handoff format and the rule-4 simpler-alternative self-check" gloss, and a one-line topical summary for each "Read these when relevant" rule file (lines 82–89). The pattern is the same: the rule file is named, then its content is glossed in CLAUDE.md.

`docs/open-questions.md`'s rules-read-reliability story (lines 152–158) names this directly as a failure mode: "CLAUDE.md summarizes the rules ... so Claude treats the summary as sufficient priors." The summary is enough to pattern-match against the request without re-reading the actual rule, even though CLAUDE.md *also* says "MUST DO before responding... Read these two files end-to-end" two lines later. Diagnostic evidence: this project's own root CLAUDE.md exhibits the same drift — Claude in this session, on this project, only re-read the rule files because Jamie's first message explicitly named them.

Future-Claude in a fresh downstream project gets the same trap baked-in. Direction 2 of the rules-read story says: don't tell Claude what the rules *are* — tell Claude to read them.

Wrap-lock: Phase 2.1 R3 places the Collaboration-rules section inside a CC-TEMPLATE-BLOCK. Whatever shape lands here becomes the wrapped shape. Reshaping the summaries post-wrap means modifying wrapped content, which forces refresh's inline-edit conflict UX on every downstream consumer.

**Recommendation.** Pick one of the three options at markup via `DECISION:` or `Accepted` (Option A is recommended).

- **Option A (Recommended) — Strip topical summaries; pointer-only shape.** Replace the rule-list parenthetical and the per-file topical glosses with a minimal pointer:

  ```
  - `rules/coding-session-rules.md` — 9 standing rules.
    Read end-to-end before responding.
  - `rules/design-philosophy-rules.md` — KISS, progressive
    disclosure. Read end-to-end before responding.
  ```

  And for the "Read these when relevant" list, replace topical glosses with file-name + one-clause purpose only ("project scope discipline", "test discipline", "cross-platform conventions", "subagent use"). No mention of rule contents, no quick-recall tokens at this surface — those live in the rule files where reading them earns the pattern. *Downstream:* the Direction 2 fix lands. Per-file size: cc-template/CLAUDE.md drops ~12–15 lines. Phase 2.1 wraps the minimal shape; future drift in CLAUDE.md from re-summarizing rules is structurally suppressed.

- **Option B — Keep file pointers verbose; strip only the parenthetical 9-rule list.** Remove the "(root-cause, trust diagnosis, ...)" parenthetical and the "Includes the rule-7 commit handoff format..." sentence, but leave the "Read these when relevant" per-file topical summaries intact. *Downstream:* partial fix. Direction 2's intent is "stop letting Claude pattern-match against summaries"; leaving per-file summaries in place lets that pattern-match re-emerge against the rarer-read rules. Saves ~5 lines. The compromise has cost without proportionate benefit.

- **Option C — Leave the summaries; rely on the foregrounded MUST-DO directive (B2) alone.** Trust that B2's repositioning will fix the read-skip behavior; keep the summaries as accessible reference. *Downstream:* simpler-alternative per rule 4 — does nothing here, defers all the work to B2's placement. Cost: the failure mode the rules-read story names (false-prior pattern-match) survives this checkpoint and gets locked in by the wrap. If B2 alone doesn't fix the behavior, reopening B1 post-wrap is expensive.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### B2. Foreground the rules-read directive in `cc-template/CLAUDE.md`

**Sources.** [`cc-template/CLAUDE.md` lines 68–72](../../cc-template/CLAUDE.md#L68-L72) (current placement: Collaboration-rules section, third major section after the banner and Reading-order). [`docs/open-questions.md` "Rules-read reliability" Direction 1 lines 168–175](../open-questions.md#L168-L175) ("Move the rules-read instruction to the top of CLAUDE.md"). Checkpoint 002 R3 partition (banner is consumer-owned, free region; Collaboration-rules + Reading-order are template-owned, wrapped).

**Problem.** The MUST-DO rules-read directive currently sits as the lead of the third major section of `cc-template/CLAUDE.md`. By the time Claude reads it, the banner has framed the configuration ritual, the Reading-order has framed session-start scanning, and attention is already on the user's request. The open-questions.md story (line 155) names this directly: "The instruction is buried — it's the third section of CLAUDE.md, after a status banner and a Reading-order list. By the time Claude reaches it, attention is already on the user's request."

Direction 1 of the rules-read story proposes: "STOP. Read `rules/coding-session-rules.md` and `rules/design-philosophy-rules.md` end-to-end before responding." First line, no preamble.

Wrap-lock partial: if the directive moves to the banner-or-above region (consumer-owned per R3), it's *not* wrapped and stays editable; if it moves within Collaboration-rules (template-owned per R3), it's wrapped and locked. The placement option set is downstream of the wrap-lock question.

**Recommendation.** Pick one of the three options at markup. Option A is recommended.

- **Option A (Recommended) — Lift to a top-of-file directive above the banner.** Insert a pre-banner block as line 3 (after the `# CLAUDE.md` heading and before the three status comments):

  ```markdown
  > **Before responding to any user message in this session,
  > read `rules/coding-session-rules.md` and
  > `rules/design-philosophy-rules.md` end-to-end.** The
  > contents are not loaded into context by default. This is
  > the #1 cause of rule drift; the configuration banner and
  > reading list below assume you've done this.
  ```

  Then strip the equivalent MUST-DO paragraph from the (former) Collaboration-rules section so it doesn't duplicate. *Downstream:* lives in the consumer-owned banner region per R3 (not wrapped). Future template revisions can adjust the directive without forcing inline-edit UX. Maximally foregrounded — Claude reading this file sees the directive before anything else. Pairs with B1: with the rule summaries stripped, the directive is the only Path to the rules priors.

- **Option B — Keep within Collaboration-rules but make it the section's opening.** Promote the MUST-DO from a sub-bullet of the section to the section's lead sentence; put the rule-file pointers below. *Downstream:* still within the wrapped Collaboration-rules section (template-owned per R3). Wrap-locks the placement; the directive lives where it lives now but slightly louder. Phase 2.1 wraps it as part of the section. Reshaping later forces conflict UX.

- **Option C — Add a UserPromptSubmit hook that injects the directive on every first message.** Direction 3 (defer per N3) suggested hook-based enforcement; this is a partial application — not enforcement of the read, but injection of the reminder. *Downstream:* requires `.claude/settings.json` machinery in the seed; cross-platform hook portability not validated; complicates the day-one experience for the consumer. Likely worse than Option A and properly belongs in the deferred Directions 3+4 work (N3). Surfaced for completeness; not recommended.

- **Option D (simpler-alternative per rule 4) — Leave the directive where it is.** Accept that placement isn't the bottleneck and rely on Claude's attention training. *Downstream:* the failure mode the story names survives the checkpoint. Cost: same as B1's Option C — the read-skip behavior persists.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

### Recommendations — resolve before Phase 2.1

Ten recommendations. Each is either a compression move (R1–R5), a doc-coherence cleanup that should ship in the seed before Phase 2.1 wraps (R6–R9), or a scope decision that affects Phase 2.1's prompt scope (R10).

#### R1. Rule 7 structural restructure — Pre-flight + Constraints + Mechanics consolidation

**Sources.** [`cc-template/rules/coding-session-rules.md` lines 67–148](../../cc-template/rules/coding-session-rules.md#L67-L148) (rule 7 in its entirety, ~82 lines). [`cc-template/.claude/commands/wind-down.md` Step 4 lines 232–253](../../cc-template/.claude/commands/wind-down.md#L232-L253), `design-review.md` Step S1.8 (this command, this run, in [the file you're editing's source spec](../../cc-template/.claude/commands/design-review.md)) and Step S2.7, `exit-test-plan.md` Step S1.7 and Step S2.7 (parallel rule-7 re-read instructions across all handoff-surfacing commands). Phase 1.2 R4b which last reworked rule 7 (landed -11.7% aggregate, brevity foregrounded). [`docs/PROJECT_PLAN.md` Phase 1.2 deliverables lines 130–169](../PROJECT_PLAN.md#L130-L169).

**Problem.** Rule 7 in `cc-template/rules/coding-session-rules.md` is ~82 lines and ~5× larger than the next-biggest rule. Phase 1.2 R4b restructured it once to foreground brevity ("subject + zero or one body sentence"). What R4b did not do is consolidate the *three sections* (Pre-flight at lines 88–92, Constraints at 94–122, Mechanics reference at 124–148) that each restate the same 6-step pre-commit sequence in different shapes.

- **Pre-flight** instructs Claude to "re-read this section end-to-end before generating any commit handoff," and names the surfacing commands. This is procedural ("when to re-read"), not the sequence itself.
- **Constraints** lists the 6-step pre-commit dry-run sequence numbered 1–6, plus 4 numbered meta-constraints.
- **Mechanics reference** restates the same constraints from a PowerShell-quoting angle and ends with a "Correct shape" example that embeds the 6-step sequence as commit-message anatomy.

Future-Claude reads rule 7 at the start of every session that involves a handoff. The cognitive load is "re-read three nearly-identical specifications of the same sequence and reconcile their framings." The structural fix is to make each section answer a *different* question with no overlap.

Diagnostic evidence: the root rule 7 (this project's local consumer) inherits the same structure. Drift here propagates to every onboarded consumer.

**Recommendation.** Pick one of the three options at markup. The recommendation is Option A.

- **Option A (Recommended) — Move Pre-flight to handoff-surfacing commands; merge Constraints + Mechanics into one annotated example.** Rule 7 becomes:
  1. The core imperative (Claude doesn't commit; Claude surfaces commands) at lines 67–74.
  2. Commit-message brevity (subject + zero/one body sentence; bullets cite doc IDs) at lines 76–86. Unchanged.
  3. **Constraints (merged)** — single section, 4 numbered constraints with one annotated example showing the 6-step dry-run sequence as the canonical shape. The example includes the PowerShell `-m "..."` quoting form inline as the worked case.
  4. **Mechanics reference removed.** Negative rules ("never multiple `-m` flags", "never here-docs") fold into the merged Constraints as inline notes alongside the worked example.
  5. **Pre-flight removed from rule 7.** Each command file that surfaces a handoff (`wind-down.md` Step 4, `design-review.md` Steps S1.8 + S2.7, `exit-test-plan.md` Steps S1.7 + S2.7) already names the re-read explicitly. Rule 7's Pre-flight paragraph is the meta-reminder *about* those reminders.

  *Downstream:* rule 7 drops from ~82 lines to ~45–55 lines. The handoff-surfacing command files are already self-contained re-read pointers; removing rule 7's Pre-flight doesn't remove the behavior, it removes the duplication. Per-session cost drops because rule 7 is in the must-read pair on every session. *Cost:* Pre-flight is a single-source-of-truth gain — having it in rule 7 means new handoff-surfacing commands inherit the discipline by reference; per-command Pre-flight makes new commands easy to forget the re-read. Mitigation: this checkpoint's own `/design-review` spec already names the re-read explicitly; future handoff-surfacing commands authored by Claude can pattern-match against existing commands. The discipline survives.

- **Option B — Keep three sections; trim only internal repetition.** Within each section, remove sentences that restate intent already covered earlier in the same section. Pre-flight loses its "Habit-mode handoffs drift long" sentence; Mechanics loses the example footer that duplicates Constraints' constraint 4. *Downstream:* rule 7 drops from ~82 lines to ~70 lines. Conservative; preserves the existing reader path. Trade: the three-restate problem survives; the next reviewer will see the same opportunity.

- **Option C (simpler-alternative per rule 4) — Leave rule 7 as it is.** Phase 1.2 R4b just touched it; rate-limit further changes until a new failure mode emerges. *Downstream:* rule 7's churn cost is real and Phase 1.2 only one cycle ago. Accept the bloat as paid-for safety prose. Cost: every consumer inherits the bloated rule 7 for the foreseeable future; structural drift becomes a long-term cost.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### R2. Rule 9 per-doc guidance migrates to `/wind-down`'s spec

**Sources.** [`cc-template/rules/coding-session-rules.md` rule 9 lines 198–210](../../cc-template/rules/coding-session-rules.md#L198-L210) (the "Per-doc edits to consider" enumeration). [`cc-template/.claude/commands/wind-down.md` Step 3 lines 160–229](../../cc-template/.claude/commands/wind-down.md#L160-L229) (the same per-doc enumeration, more detailed). NFR-8 (file-ownership doesn't overlap). The diagnostic frame: rule shape ≠ command procedure.

**Problem.** Rule 9 lines 198–210 enumerate per-doc edits at wind-down: `CLAUDE_CODE_PROMPTS.md` (mark prompts that landed), `design-decisions.md` (record significant decisions), `open-questions.md` (move unresolved questions), `ARCHITECTURE.md` / `REQUIREMENTS.md` (surface diffs), `[command].md` files (update on CLI parameter changes). This list also exists in `cc-template/.claude/commands/wind-down.md` Step 3, in greater detail with sub-sections per doc.

The duplication is the question. Rule 9 is *behavioral discipline* — "at wind-down, the doc cluster must be coherent before closing." `/wind-down`'s Step 3 is *procedure* — "here's the specific list to walk and what to look for in each." These are different artifacts; the per-doc enumeration in rule 9 reads as procedure embedded in a rule.

Future-Claude effect: a downstream Claude reading rule 9 in a session that doesn't involve `/wind-down` (e.g. mid-implementation) still reads through the per-doc list. If they're not winding down, the procedure is overhead. The rule's load-bearing claim ("at wind-down, rewrite TODO.txt and keep docs coherent") doesn't need the procedural list to land.

**Recommendation.** Pick one of the three options at markup. Option A is recommended.

- **Option A (Recommended) — Migrate the per-doc enumeration to `cc-template/.claude/commands/wind-down.md` Step 3; rule 9 keeps only the discipline.** Rule 9 in cc-template/rules/coding-session-rules.md ends after line 196 (the "Wind-down also keeps the rest of the repo's docs coherent — surface only the docs that need updates this session. State the intent in one short sentence, then edit directly via the Edit tool ... Multi-doc structural changes warrant a brief plan first" paragraph) with one new sentence:

  > See `/wind-down`'s Step 3 for the per-doc checklist.

  The bullet list at lines 198–210 moves into `wind-down.md` Step 3, which already covers the same items in more depth. *Downstream:* rule 9 drops ~13 lines. The per-doc procedure lives in one file (the command that runs it). Adding a new tracked doc (e.g. a future `docs/STATUS.md`) requires one edit, not two. *Cost:* a fresh Claude reading rule 9 without context loses the inventory unless they cross-reference `wind-down.md`. Mitigation: rule 9 is invoked during wind-down sessions; the cross-reference is fast.

- **Option B — Keep both lists; cross-reference for consistency.** Add a "this list mirrors `wind-down.md` Step 3; update both" note in rule 9. *Downstream:* duplication continues but is documented as such. New tracked docs require two edits. Cost: explicit duplication is a maintenance burden; the cross-reference itself is drift bait (one side updates, the note doesn't).

- **Option C (simpler-alternative per rule 4) — Leave both lists as-is.** Accept the duplication as an unavoidable cost. *Downstream:* no behavior change; new docs require two edits; the next compression checkpoint will see the same opportunity.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### R3. Rule 8 manual-test paragraph — collapse to a one-line pointer

**Sources.** [`cc-template/rules/coding-session-rules.md` rule 8 lines 156–162](../../cc-template/rules/coding-session-rules.md#L156-L162) (the "manual phase-exit walkthroughs" paragraph describing `/exit-test-plan` flow). `cc-template/.claude/commands/exit-test-plan.md` (the command file that owns the spec, including BLOCKED disposition added in Phase 1.1).

**Problem.** Rule 8 covers two things: (a) "Claude proposes test commands, Jamie runs them" — the core discipline; (b) "manual phase-exit walkthroughs use `/exit-test-plan`" — a 7-line paragraph (lines 156–162) describing Stage 1 / Stage 2 flow, the artifact-vs-runner distinction, and "Mid-run failures are normal coding work driven by Fail signals in each TC." The (b) content is a teaser for the command file; the spec lives in `cc-template/.claude/commands/exit-test-plan.md` which already covers it in detail.

The downstream-Claude cold-read question: does a Claude landing on rule 8 *without* `/exit-test-plan` context need the teaser, or is "see `/exit-test-plan` for manual walkthroughs" enough?

Two angles. The teaser names the protocol skeleton (Stage 1 authors plan, Jamie runs, Stage 2 disposes) without depth — useful as orientation. The teaser also restates what `/exit-test-plan.md`'s own spec says with less authority — drift bait if the command spec evolves (e.g. BLOCKED was added in Phase 1.1; rule 8's paragraph wasn't updated).

**Recommendation.** Pick one at markup. Option A recommended.

- **Option A (Recommended) — Collapse to a one-line pointer.** Replace lines 156–162 with:

  > Rule 8 also covers manual phase-exit walkthroughs — see `/exit-test-plan` for the artifact-driven flow. Claude never runs the steps; Jamie does.

  *Downstream:* rule 8 drops ~6 lines. The command file is canonical. Future revisions to the walkthrough protocol require one edit (the command file), not two. *Cost:* a Claude landing on rule 8 without prior `/exit-test-plan` exposure no longer sees the protocol skeleton in-line — they must read the command. Mitigation: rule 8 says "see `/exit-test-plan`" right after; the cost is one click for the procedural reader.

- **Option B — Keep the paragraph; add a single-sentence cross-reference and trim internal repetition.** Cut the "Mid-run failures are normal coding work driven by Fail signals" sentence (the command spec covers this); leave the Stage 1/Stage 2/artifact skeleton. *Downstream:* drops ~2 lines; protocol skeleton preserved. Same drift risk; same value lock.

- **Option C (simpler-alternative per rule 4) — Leave as-is.** Accept the teaser as paid-for orientation. *Downstream:* the drift between rule 8's paragraph and `exit-test-plan.md`'s spec is a maintenance discipline issue (must update both); next /exit-test-plan revision needs to remember to sync rule 8.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### R4. Environment-rules version-freshness — consolidate the "Where this fires" surfaces

**Sources.** [`cc-template/rules/environment-rules.md` Version freshness lines 128–175](../../cc-template/rules/environment-rules.md#L128-L175) — the full rule including "Surfaces in scope" (lines 159–165) and "Where this fires in the configuration ritual" (lines 167–175). [`cc-template/.claude/commands/onboard.md` Step 3 lines 81–88](../../cc-template/.claude/commands/onboard.md#L81-L88) (Version recency call-out). [`cc-template/.claude/commands/bootstrap.md` Step 3 lines 142–149](../../cc-template/.claude/commands/bootstrap.md#L142-L149) (Version recency at question time) **and** Step 5 lines 229–240 (re-check at write time, both README and environment-rules append). Each of the three commands restates the version-freshness check at its own step(s).

**Problem.** The rule statement (lines 137–157 — "Before pinning any software version into a project artifact: 1. Surface the assumption, 2. Offer to look up current stable/LTS, 3. Show what was found and let Jamie pick") is a 21-line load-bearing core. What follows is two enumerations: "Surfaces in scope" lists the categories of pinned versions (language runtimes, framework majors, etc.), and "Where this fires in the configuration ritual" names `/onboard` Step 3, `/bootstrap` Step 3 + Step 5, `/deployment-plan` Step 3 + Step 5 — all three commands by name.

The duplication is two-directional:
- environment-rules.md's "Where this fires" lists the commands.
- Each command file's relevant Step re-states the version-freshness rule (with varying levels of detail and references back to environment-rules.md).

The "Where this fires" inventory in environment-rules.md is the kind of thing that drifts: when a new command is added that pins a version (or an existing command grows a new pin surface), the inventory needs updating in the rules file — easy to forget. The rule itself is invariant; the inventory is project-state.

Future-Claude reading environment-rules.md gets the rule (canonical) plus an enumeration of where it applies (incidentally; lookups go via the command file anyway).

**Recommendation.** Pick one at markup. Option A recommended.

- **Option A (Recommended) — Strip the "Where this fires in the configuration ritual" subsection from environment-rules.md; keep the rule statement and "Surfaces in scope."** Each command file already names the check at its application points; rule lookup is by-command, not by-rules-file. *Downstream:* environment-rules.md drops ~10 lines. Adding a new pin-surface command requires updating only that command file. *Cost:* the rules-file reader doesn't get a global view of where the check fires. Mitigation: command files are discoverable; this isn't an obscure rule.

- **Option B — Strip "Surfaces in scope" and "Where this fires"; keep only "The rule" subsection.** Aggressive trim — both the category enumeration and the command list go. *Downstream:* environment-rules.md drops ~15 lines. The rule's scope ("when does this apply?") loses the categorical hint ("language runtimes; framework majors; database engines..."). Cost: a Claude reading the rule for the first time loses the contextual "what counts as a version pin" hint and has to derive it from the rule statement alone. Probably too aggressive.

- **Option C (simpler-alternative per rule 4) — Leave both enumerations as-is.** Accept the duplication as orientation overhead. *Downstream:* future command additions create drift risk (inventory in rules file doesn't track new commands). Maintenance burden continues.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### R5. "Simpler alternative in the same message" — canonical source

**Sources.** [`cc-template/rules/coding-session-rules.md` rule 4 lines 47–53](../../cc-template/rules/coding-session-rules.md#L47-L53) — the "Self-check before any addition" paragraph with the "in the same message" phrase and the visible-enforcement-of-KISS framing. [`cc-template/rules/design-philosophy-rules.md` KISS section lines 11–16](../../cc-template/rules/design-philosophy-rules.md#L11-L16) — "name what you'd add and why, and state the simpler alternative **in the same message**." Both files cross-reference each other at lines 51–53 (rule 4 cites design-philosophy) and line 22 (design-philosophy cites rule 4).

**Problem.** The "simpler alternative in the same message" imperative appears in both files in nearly identical form. Each file cites the other as the enforcement-or-framing partner: rule 4 cites design-philosophy as the KISS framework; design-philosophy cites rule 4 as the gate. The mutual citation is the discipline equivalent of "see also" — and it's load-bearing in the sense that the imperative needs to be in working memory at the moment of any proposal.

The question Stage 2 will resolve: which file owns the canonical text? The two are conceptually different — rule 4 is *behavioral guardrail* (what Claude does), design-philosophy is *judgment framework* (how to apply KISS). The "simpler alternative" mechanic is the bridge between them: it's a behavioral enforcement of a judgment principle.

**Recommendation.** Pick one at markup. Option A recommended.

- **Option A (Recommended) — Rule 4 owns the canonical statement; design-philosophy KISS links to it.** Rule 4's "Self-check before any addition" paragraph stays as the source. The KISS section in design-philosophy replaces lines 11–16 with:

  > Before adding complexity, name what you'd add and why, and apply rule 4's simpler-alternative self-check (`coding-session-rules.md` § Rule 4). KISS is the judgment frame; rule 4 is the behavioral gate.

  *Downstream:* coding-session-rules and design-philosophy-rules each get one canonical reference. Behavioral mechanism lives where behavioral guardrails live; the judgment frame points to the behavioral enforcement. Saves ~5 lines in design-philosophy. *Cost:* the KISS section becomes slightly less self-contained; a Claude reading design-philosophy in isolation must cross-reference. Mitigation: both files are in the MUST-DO read pair — both get read together by design.

- **Option B — KISS owns the canonical statement; rule 4 links to it.** Inverts Option A. Rule 4's "Self-check" paragraph becomes a one-line reference to KISS. *Downstream:* the imperative moves into the judgment-framework file. The argument: KISS is the *why* and rule 4 the *what*; better to anchor at the *why*. The counter: rule 4 is read first in the rule-list, so anchoring at rule 4 is the path of least surprise.

- **Option C (simpler-alternative per rule 4) — Leave both files as-is.** Both restatements continue; cross-references stand. *Downstream:* the duplication is a real cost; both files must stay in sync if the phrasing evolves. Acceptable if the imperative is so load-bearing that two anchor points actively help — but the cross-references make this an opinion call, not evidence-based.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### R6. Strip recurring-command cadence prose from `cc-template/CLAUDE.md` banner

**Sources.** [`cc-template/CLAUDE.md` banner lines 32–45](../../cc-template/CLAUDE.md#L32-L45) (the "two recurring commands run on Jamie's cadence" paragraph). [`docs/open-questions.md` "Trim CLAUDE.md" drift category 2 lines 331–338](../open-questions.md#L331-L338) ("Recurring-command cadence prose in `cc-template/CLAUDE.md` ... The cadence is the skill's own job to know"). `cc-template/.claude/commands/design-review.md` Step 0 and `cc-template/.claude/commands/exit-test-plan.md` Step 0 (each command's own cadence-detection logic).

**Problem.** The cc-template/CLAUDE.md banner currently includes 15 lines (lines 32–45) describing when `/design-review` and `/exit-test-plan` run. The text is partially evergreen (the names of the commands) and partially specific (what triggers each one — "post-onboarding sanity check; between phases that touch schema, multi-tenancy, auth, or anything expensive-to-retrofit"). The cadence description duplicates the command files' own Step 0 logic, where the actual stage-detection lives.

The story's argument: the cadence is the skill's job to know. CLAUDE.md naming when each command runs lets future-Claude pattern-match against the description without invoking the command. That works fine when the description is current, fails silently when it drifts.

Diagnostic evidence: root CLAUDE.md doesn't carry this paragraph in the same form (it's onboarded; the banner there is project-specific), but every onboarded project starts from this seed and inherits the paragraph until /onboard rewrites the banner. /onboard's Step 5 *does* rewrite the banner, so the inheritance is bounded — but during the gap between cc-template/ ship and /onboard run, the seed CLAUDE.md is the document a fresh Claude reads.

Banner is consumer-owned per R3 (not wrapped). So this isn't wrap-locked; reshaping later is cheap. But every newly-onboarded project until next refresh starts from the current shape.

**Recommendation.** Pick one at markup. Option A recommended.

- **Option A (Recommended) — Strip lines 32–45 entirely; replace with a one-line pointer.** Replace the paragraph with:

  > After configuration, `/design-review` and `/exit-test-plan` run on Jamie's cadence — invoke when needed; each command's Step 0 detects state and asks. No status comment in CLAUDE.md.

  *Downstream:* banner drops ~13 lines. The "no status comment" reminder (which is load-bearing per Load-bearing invariants) survives. The cadence prose itself goes. *Cost:* the seed-Claude no longer sees, at a glance, when each command runs. Mitigation: the command files are discoverable via `.claude/commands/` and each has a tight introductory description.

- **Option B — Trim the descriptive trigger text; keep the command names with one-line purposes.** Keep "`/design-review` produces sign-off-ready checkpoints" and "`/exit-test-plan` authors and closes out manual walkthroughs" — drop the example triggers ("between phases that touch schema, multi-tenancy, auth..."). *Downstream:* banner drops ~7 lines. Compromise; lighter strip; same drift vector if command behavior evolves.

- **Option C (simpler-alternative per rule 4) — Leave the paragraph as-is.** Accept the banner bloat as orientation. *Downstream:* the cadence prose drift continues; the next downstream consumer's CLAUDE.md inherits it until /onboard rewrites the banner.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### R7. Constrain self-edited "next-phase" banners in onboarded CLAUDE.md

**Sources.** [`docs/open-questions.md` "Trim CLAUDE.md" drift category 1 lines 321–330](../open-questions.md#L321-L330) ("Self-edited 'next phase' banners. Root CLAUDE.md currently opens with a status banner ... As of 2026-05-27 the banner still names 'Phase 1.1 — add BLOCKED' even though that landed three commits ago"). Diagnostic evidence — this project's own root CLAUDE.md banner. `cc-template/.claude/commands/onboard.md` Step 5 lines 412–435 (rewrites CLAUDE.md banner post-onboard). `cc-template/.claude/commands/bootstrap.md` Step 6 lines 304–342 (banner rewrite at Stage 1 and Stage 2). `cc-template/.claude/commands/wind-down.md` Step 3 (per-doc updates — CLAUDE.md is not in its scope today).

**Problem.** Root CLAUDE.md on this project carries a self-edited next-phase banner — the line "Ready for the next phase prompt in `docs/CLAUDE_CODE_PROMPTS.md` (Phase 1.1 — add BLOCKED disposition to `/exit-test-plan`)" — that has been stale through three commits. Nobody re-wrote it because no command's spec actually instructs anyone to. /onboard writes the banner once; /bootstrap writes it once more; nothing after that touches it. TODO.txt is the authoritative next-step source, but CLAUDE.md re-states a snapshot of next-step in prose.

Diagnostic frame: this is drift the template invited. cc-template/CLAUDE.md's seed has the ritual banner (which is correctly the entry-point for fresh projects); /onboard and /bootstrap re-write it for onboarded state. The downstream consumer then maintains it manually — and the maintenance has no rule, no command-driven trigger, and no anti-drift discipline. So it drifts. The fix is to make the template structurally not invite this — either by removing the next-step prose from the banner shape /onboard writes, or by making /wind-down responsible for refreshing it.

What `cc-template/` *should* ship that would have prevented this drift:

- **Option A — Strip next-step prose from the banner shape /onboard writes.** /onboard Step 5's post-onboard banner template (lines 419–428) currently reads "Next step is `/design-review` for a post-onboarding sanity check, then `/bootstrap` to plan the dev environment, then Prompt 0." Replace with a static framing that doesn't name a next prompt: "Configuration ritual complete. See `TODO.txt` for next steps." Same for /bootstrap Stage 2's banner (lines 428–438) and /onboard's stage-1 banner. *Downstream:* banner becomes a stable orientation surface; next-step prose lives only in TODO.txt where wind-down is responsible. The next-phase drift mode is structurally impossible. *Cost:* the banner stops being a quick "where are we right now" answer; consumers must read TODO.txt for that.

- **Option B — Add an anti-drift rule to rule 9 / wind-down spec.** Add to wind-down.md Step 3 a check: "if CLAUDE.md banner names a specific next prompt or phase, verify it matches TODO.txt's first entry; if not, surface a warning." *Downstream:* the banner can still hold next-step prose but a maintenance discipline kicks in. Cost: more rule weight; consumers may forget to run /wind-down.

- **Option C — Make /wind-down responsible for refreshing the banner.** Extend wind-down.md Step 3 to rewrite the next-step line in CLAUDE.md's banner from TODO.txt's first entry. *Downstream:* the banner stays accurate, automatically. Cost: wind-down now edits the highest-traffic file; CLAUDE.md becomes mutable in a new dimension; CC-TEMPLATE-BLOCK wrap interactions to think through.

- **Option D (Recommended) — Combine A + B.** /onboard and /bootstrap write a static banner without next-step prose; rule 9 / wind-down adds the consistency check as a safety net for projects whose banner does drift. Belt-and-suspenders. *Downstream:* the drift mode is suppressed at write-time AND caught at re-read time. /wind-down's check is cheap (read TODO.txt, scan CLAUDE.md banner, warn if mismatch). Cost: two surfaces of guardrail; rule 9 grows slightly. The growth offsets against the rule-9 trim in R2.

- **Option E (simpler-alternative per rule 4) — Leave it.** Accept that downstream consumers maintain their banner as they choose; the drift in this project's own root CLAUDE.md is acceptable noise. *Downstream:* this project's own banner stays drift-prone; future downstream consumers inherit the same pattern. The diagnostic that motivated the finding remains unaddressed at the template surface.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### R8. Retire the in-flight artifact status callout deferred story

**Sources.** [`docs/open-questions.md` "In-flight artifact status callout at top of CLAUDE.md" lines 246–272](../open-questions.md#L246-L272) (the deferred story). Jamie's own resolution note at lines 361–374 ("This story is in tension with existing 'In-flight artifact status callout at top of CLAUDE.md' — that story proposed automating a 🟡 status banner; this one argues banners drift and TODO.txt already serves the purpose. Resolve before either lands; probably retire the callout story in favor of this one"). Checkpoint 001 N1 disposition: "Deferred ... Revisit at a future `/design-review` checkpoint (pre-Phase-2.1 or pre-Phase-2.2)." This is that checkpoint.

**Problem.** The in-flight artifact status callout story proposed a `/wind-down` enhancement: detect mid-flight `AWAITING-DECISIONS` / `AWAITING-DISPOSITIONS` artifacts and write/refresh a 🟡 callout block at the top of CLAUDE.md. Jamie's own note in the trim-CLAUDE.md story resolved the tension: banners in CLAUDE.md drift; TODO.txt is authoritative.

The resolution is implicit but not formally recorded. Closing the loop here either retires the story permanently (Option A) or keeps it on a longer leash (Option B/C).

The pairing is direct: R7 (and B1 separately) argue against banners in CLAUDE.md; this story proposes adding a banner. Both can't be true. If R7 lands the static-banner-only shape, the callout story is incompatible.

**Recommendation.** Pick one at markup. Option A recommended.

- **Option A (Recommended) — Formally retire the callout story.** Mark the open-questions.md entry as closed (move to design-decisions.md as a resolved decision, or annotate in place as "closed — won't do; superseded by Trim CLAUDE.md"). The need it pointed at — "where are we mid-flight?" — is met by `/wind-down`'s existing safety nets (Step 2 design-review and exit-test-plan safety nets that name the artifact and the next action in TODO.txt). *Downstream:* the tension with R7 / B1 resolves cleanly; future maintainers don't re-propose the callout because the resolution is recorded.

- **Option B — Defer with a re-open trigger.** Keep the story open with a specific trigger: "reopen if downstream consumers report difficulty discovering mid-flight artifacts despite the /wind-down safety nets." *Downstream:* leaves the option alive without committing. Cost: open-questions.md keeps a story that R7 / B1 makes structurally hard to land — the next pre-checkpoint review re-evaluates the same tension.

- **Option C (simpler-alternative per rule 4) — Leave the story as-is.** Don't touch open-questions.md; let the next checkpoint re-decide. *Downstream:* the unresolved tension persists in the deferred-stories list; no commitment either way.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### R9. Seed `cc-template/TODO.txt` as a walkable onboarding checklist

**Sources.** [`cc-template/TODO.txt` line 5](../../cc-template/TODO.txt#L5) (current single-line content: "Run /onboard to configure this project from the design doc in docs/design/."). [`docs/open-questions.md` "Seed cc-template/TODO.txt as the active onboarding checklist" lines 375–434](../open-questions.md#L375-L434). [`cc-template/README.md`](../../cc-template/README.md) (the README the checklist's first step points to). [`cc-template/.claude/commands/onboard.md` Step 0 lines 36–46](../../cc-template/.claude/commands/onboard.md#L36-L46) (current /onboard prerequisite check).

**Problem.** The seed `cc-template/TODO.txt` has one entry: "Run /onboard." That makes /onboard look like the literal first action and silently skips the rules-review step the README is supposed to recommend. Two compounding failures:

1. **The rules-review step is silently expected.** The README correctly says (or should say) the consumer should review the rules before /onboard runs — to decide which rules they agree with, which to edit, which to move out of CC-TEMPLATE-BLOCK markers once those exist. TODO.txt doesn't reflect this; the first entry is "Run /onboard" cold.
2. **The TODO-driven habit doesn't get taught.** The downstream consumer learns TODO.txt as a Claude-maintained tracking file rather than a checklist humans walk through. Day-one habit setting matters because the template's whole design depends on TODO.txt being the working surface.

Pairing: this is the consumer-facing complement of B2 (foreground the rules-read directive). B2 ensures Claude reads the rules; R9 ensures the *consumer* reviews them.

Phase 2.1 interaction: the CC-TEMPLATE-BLOCK markers ship as part of Phase 2.1. The customization workflow ("review rules → decide which to lift out of markers → /onboard") only makes sense once markers exist. R9 can land the checklist with the marker step parenthesized as "once markers ship in Phase 2.1" — the framing is forward-compatible.

**Recommendation.** Pick one at markup. Option A recommended.

- **Option A (Recommended) — Seed with a numbered 1–6 checklist; soft framing on rules-review depth.** Replace the current single-line "Run /onboard" with:

  ```
  TODO

  What's next (consumer onboarding checklist — walk these in order)
  -----------------------------------------------------------------
  1. Read `README.md` to understand the template shape.
  2. Read each `rules/*.md` file. Edit anything you disagree
     with. (Once `/refresh-from-repository` ships in Phase 2.1,
     you'll be able to lift content out of CC-TEMPLATE-BLOCK
     markers so refresh leaves your edits alone.)
  3. Drop a design doc into `docs/design/`.
  4. Run `/onboard`.
  5. (Next session) Run `/bootstrap`.
  6. (When you're ready) Run `/deployment-plan`.

  Use this file as your handoff between sessions. Rewrite it
  (don't append) at each session close per rule 9.

  Carryover
  ---------
  - (nothing yet)

  Deferred decisions
  ------------------
  - (nothing yet)
  ```

  Update `cc-template/README.md` to describe the rule-divergence workflow at a high level (review → edit → optionally lift from markers) and point at TODO.txt as the working checklist. *Downstream:* day-one experience surfaces the rule-review step explicitly. Consumers learn TODO.txt as a walkable checklist from first contact. Phase 2.1 marker integration is teased ("once markers ship") but the checklist works pre-Phase-2.1. Soft framing on step 2: "edit anything you disagree with" lets consumers skip-and-defer without paternalism. *Cost:* the seeded TODO.txt becomes opinionated about workflow; consumers who want a clean slate must edit it. Mitigation: rule 9 already names TODO.txt as personal scratch — rewriting it is normal.

- **Option B — Same checklist but with /onboard verifying it was walked.** Add a check to /onboard Step 0: read TODO.txt; if the first entry is still "Run /onboard" verbatim (the seed), warn the consumer that the rules-review step was likely skipped and ask before proceeding. *Downstream:* paternalistic enforcement; risk of annoying consumers who explicitly chose to skip. Cost: enforcement layer that doesn't add capability — only friction.

- **Option C — Lighter checklist with /design-review optional.** A 4-step shorter list: read README, drop design, /onboard, /bootstrap. Rules-review elided; deployment-plan elided. *Downstream:* loses the rule-review surfacing; defeats the finding's purpose.

- **Option D (simpler-alternative per rule 4) — Leave the single-line "Run /onboard" seed.** Accept that consumers find the rules-review step on their own. *Downstream:* the silent-expectation drift continues; the TODO-driven habit isn't taught at day one.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### R10. Universal-rules customization partitioning — what Phase 2.1's prompt must address

**Sources.** [`TODO.txt` JAH NOTE line 15](../../TODO.txt#L15) ("How will we 'upgrade' an existing project that had the old rules without the new markers and system? Manual session?"). [`docs/PROJECT_PLAN.md` Phase 2.1 lines 180–268](../PROJECT_PLAN.md#L180-L268) (current scope). [`docs/CLAUDE_CODE_PROMPTS.md` Prompt 2.1 lines 350–474](../CLAUDE_CODE_PROMPTS.md#L350-L474) (current prompt). Checkpoint 002 R2d caveat (cross-file migration hash-upgrade-offer for known template-managed files).

**Problem.** The JAH NOTE in TODO.txt raises an unresolved question: when Phase 2.1 ships /refresh-from-repository with CC-TEMPLATE-BLOCK markers, existing downstream projects that were onboarded *before* the markers existed will have rules files without markers. Their first /refresh-from-repository run encounters files that don't fit the contract. What's the migration path?

Two interpretations of what Phase 2.1 must handle:
- **(a) First-run migration.** /refresh-from-repository detects pre-marker state on first invocation (rules files exist, no CC-TEMPLATE-BLOCK markers anywhere) and offers a one-time migration: insert markers at coarse-grained boundaries (per checkpoint 002 R6), seed the refresh state file with hashes of the current content as the baseline, then proceed normally.
- **(b) Manual session.** /refresh-from-repository refuses to run when no markers are present, points at a documented manual migration procedure (insert markers by hand, run an init command, then refresh works).

Either way, this is a Phase 2.1 prompt scope question. The brief flagged it: "fold into Phase 2.1 prompt requirement" was the proposed handling.

The diagnostic note in checkpoint 002 R2d is relevant: refresh already does hash-matching against the six template-managed rules files. The same logic could power first-run migration — if the consumer's rules files match upstream's hashes block-for-block, insert markers at the boundaries; if they diverge, prompt for inline-edit conflict UX per block.

**Recommendation.** Pick one at markup. Option A recommended.

- **Option A (Recommended) — Add to Phase 2.1's prompt scope: a numbered scope item naming first-run migration.** Insert into Prompt 2.1 in `docs/CLAUDE_CODE_PROMPTS.md` a new scope item between current items 11 and 12 (or wherever fits sequentially):

  > **Pre-marker migration.** When /refresh-from-repository is invoked in a downstream project that has rules files but no CC-TEMPLATE-BLOCK markers (the consumer was onboarded pre-Phase-2.1), the command performs a one-time migration: insert coarse-grained markers (per checkpoint 002 R6) at top-level rule-section boundaries; seed `.claude/claude-code-sdlc-template-refresh-state.md` with per-block hashes computed from the consumer's current content; surface block-by-block any boundaries where the consumer's content diverges from upstream's block boundaries, using the standard inline-edit conflict UX. After migration, normal refresh applies.

  *Downstream:* Phase 2.1's build session has a concrete contract for the migration path; the JAH NOTE has a clear home. No separate checkpoint or work-item needed. *Cost:* Phase 2.1's scope grows by one item; the build session is more work. Acceptable — the work was always going to be needed; this just makes it explicit.

- **Option B — Lift to a Phase 2.1 design-decision before the build session.** Add a new entry to `docs/design-decisions.md` capturing the first-run migration contract (Option A's text shape) as a closed decision; the build session implements against the decision. *Downstream:* design-decisions.md grows; Phase 2.1 scope stays as-is but cites the decision. Equivalent outcome; slightly heavier ceremony. The decision is small enough that the prompt-scope-item form fits.

- **Option C — Defer the migration question to a Phase 2.1.1 mini-phase.** Phase 2.1 ships /refresh-from-repository assuming markers exist; pre-marker migration becomes a separate phase post-2.1. *Downstream:* Phase 2.1 ships cleaner but consumers onboarded before Phase 2.1 can't use refresh until 2.1.1 lands. Friction.

- **Option D (simpler-alternative per rule 4) — Don't address it.** /refresh-from-repository refuses to run when markers are absent; document the manual migration in `cc-template/README.md` as a one-time procedure. *Downstream:* the JAH NOTE has a home (in the README), but the consumer experience is poorer.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

### Notes — acceptable now, revisit later

Four notes. N1 documents a deliberate "don't compress" decision so future cleanup doesn't try; N2 closes a checkpoint-001 deferral; N3 and N4 record deferrals with concrete observable triggers.

#### N1. Rule 4 self-check — document the "don't compress" decision

**Sources.** [`cc-template/rules/coding-session-rules.md` rule 4 lines 47–53](../../cc-template/rules/coding-session-rules.md#L47-L53). The "Self-check before any addition" paragraph. Cross-references in this checkpoint: R5 (canonicalization) treats this paragraph as the canonical statement.

**Problem.** Surface compression analysis on rule 4's self-check paragraph would flag it as a candidate: it's 7 lines, has emphatic phrasing ("**Self-check before any addition.**", "**in the same message**"), and could compress to ~3 lines by stripping the framing rhetoric ("Visible enforcement of KISS", "Silently picking the richer option is the failure mode").

The case against compressing it: the framing IS the load. "In the same message" is not redundant — it's the operational test (was the simpler alternative *visible to Jamie at decision time*, or considered internally and discarded?). "Visible enforcement of KISS" is what makes the self-check a guardrail rather than a private discipline. "Silently picking the richer option is the failure mode" names the exact failure mode the rule is preventing. Strip any of these and the rule loses operational teeth — the next reviewer will see it as ambiguous and propose adding the same framing back.

This finding's purpose is to record the "don't compress" decision so future cleanup checkpoints don't repeat the analysis. The note is documentation for future-Claude that this section was looked at, considered, and intentionally preserved.

**Recommendation.** Mark Accepted; Stage 2 records the decision in the Sign-off Summary as "Keep as-is; documented in checkpoint 003 N1 as intentional preservation." No doc edit other than the checkpoint itself.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### N2. Project quick orientation section — formal re-deferral

**Sources.** [`docs/open-questions.md` "Project quick orientation section in onboarded CLAUDE.md" lines 274–300](../open-questions.md#L274-L300). Checkpoint 001 N2 disposition: "Deferred ... Revisit at the same future checkpoint as N1, when more downstream projects have accumulated and the maintenance-vs-utility tradeoff is clearer." This is that checkpoint.

**Problem.** The story proposes that `/onboard` or `/bootstrap` write a "Project quick orientation" section (DB engine, source-tree location, public/admin stack split, full format/lint commands) modeled on ds-niche-stream's evolved CLAUDE.md shape. Today neither command produces it.

Checkpoint 001 N2's deferral rationale ("when more downstream projects have accumulated and the maintenance-vs-utility tradeoff is clearer") still applies. Currently the template has two known downstream consumers (ds-niche-stream and ds-auto-dailies per [`docs/design/design-review-checkpoint-001.md` lines 33–35](../design/design-review-checkpoint-001.md#L33-L35)). Sample size of two isn't enough evidence to decide whether the section is worth the maintenance load.

Counter-evidence might accumulate via: a third downstream consumer that also evolves the same section (signaling demand); or downstream consumer feedback that the banner alone is too thin; or Jamie's own template-maintenance experience finding the section consistently useful when she returns to a project.

**Recommendation.** Formally re-defer with a concrete observable re-open trigger. Mark Accepted to record:

- **Re-open trigger:** when a third downstream consumer (beyond ds-niche-stream and ds-auto-dailies) evolves a "Project quick orientation" section in its CLAUDE.md independently. The independent emergence is the demand signal.
- **Or:** when Jamie's own session-start experience on this project surfaces that the banner alone is consistently too thin.

The open-questions.md entry stays as-is; Stage 2 appends a "Re-open trigger added 2026-05-27 per checkpoint 003 N2" line to the entry's body.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### N3. Defer rules-read Directions 3+4 with concrete observable re-open triggers

**Sources.** [`docs/open-questions.md` "Rules-read reliability" Directions 3 (lines 188–198) and 4 + 4b (lines 200–224)](../open-questions.md#L188-L224). Industry reference: [Anthropic skill frontmatter conventions](https://docs.claude.com/en/docs/claude-code/skills) (skill metadata: name, description, when-to-use). Direction 3 = UserPromptSubmit hook for unconditional rules read; Direction 4 = co-locate rules with skills; Direction 4b = session entry-point skill that asks intent.

**Problem.** The brief identified Directions 1+2 as nearly-free (B2 and B1 in this checkpoint). Directions 3+4 add machinery — hook authoring, settings.json defaults that may not be portable across IDE / desktop / web Claude Code surfaces, skill restructuring for the session entry-point pattern. Per Jamie's framing in the brief: machinery on top of discipline; defer until evidence accumulates.

Per the answers to Step S1.3 (Question 3), the defer recording should name concrete observable triggers — not just "park indefinitely."

**Recommendation.** Mark Accepted. Stage 2 appends to the open-questions.md "Rules-read reliability" entry a "Direction 3+4 deferral — re-open triggers (added 2026-05-27 per checkpoint 003 N3)" subsection:

- **Direction 3 (hooks-based enforcement) re-opens when:** B1 (rules-summary strip) and B2 (rules-read directive foregrounding) ship and downstream consumers still report skipping the read. The simpler-alternative threshold: B1 + B2 must demonstrably fail before machinery is justified. Evidence form: a downstream consumer (or Jamie on this project) raises a session in `docs/open-questions.md` describing a skip incident post-B1/B2.
- **Direction 4 (rules co-located with skills) re-opens when:** a skill the template ships earns a rule that lives canonically in that skill — for example, if `/refresh-from-repository` (Phase 2.1) develops a rule about marker-respect that isn't usefully read at session start but is critical at refresh time. The first concrete rule-to-skill anchoring is the trigger.
- **Direction 4b (session entry-point skill) re-opens when:** Direction 4 lands at least one skill-anchored rule AND a session-start friction is measured (Jamie observes more than one "I started without /wind-down and forgot rule X" pattern in a single week).

The open-questions.md entry stays as-is; Stage 2 adds the trigger subsection without rewriting the existing analysis.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### N4. Defer the artifact-boundary → /wind-down user story

**Sources.** [`docs/open-questions.md` "Artifact-boundary command landings should route session-end through `/wind-down`" lines 49–136](../open-questions.md#L49-L136). Checkpoint 002's Follow-up T4 ("Plan-mode → /wind-down user-story session — carryover from prior TODO; queued before Phase 2.1 build session") — that work was deferred in the brief because the story isn't Phase-2.1-blocking.

**Problem.** The story proposes that artifact-boundary commands (`/design-review` Stage 2 landing; `/exit-test-plan` Stage 2 landing) route their session-end commit handoff through `/wind-down` instead of surfacing it inline. The current behavior surfaces commits at the artifact boundary; the proposal routes them through wind-down so the session also gets TODO.txt refresh and the doc-coherence sweep.

The work is real but not Phase-2.1-blocking. Phase 2.1's scope is /refresh-from-repository's authoring; nothing in that work is affected by where commits surface from /design-review or /exit-test-plan. The plan-brief flagged this as defer-cleanly.

**Recommendation.** Mark Accepted with a concrete re-open trigger: when Phase 2.1 lands and the next session is wind-down work, this story becomes the next-cleanup work-item. Stage 2 appends to the open-questions.md entry a "Re-open trigger added 2026-05-27 per checkpoint 003 N4" line: "Re-open after Phase 2.1 lands; this is the next-cleanup work-item once /refresh-from-repository is shipped."

The story stays as-is; the re-open trigger pins it to a calendar event (Phase 2.1 landing) rather than indefinite parking.

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

## What the design got right (preserve)

- **Source-of-truth + dist split with `cc-template/` as the wrapped surface.** Phase 2.1's marker work has a clear target: only `cc-template/` content gets wrapped. Root content catches up via refresh from the same upstream the consumers use, so the maintainer doesn't carry a separate update path. The architecture (per [`docs/ARCHITECTURE.md`](../ARCHITECTURE.md)) directly enables this checkpoint's "edit `cc-template/` only" scope discipline.
- **Two-stage `/design-review` with addendum iteration.** The pattern that landed checkpoint 002 (Round 1 + 1 addendum, mechanism-resolution surfacing in the addendum) is exactly the right shape for this checkpoint too. The Stage 1 / Stage 2 / land-or-addendum loop scales to complex multi-finding reviews without forcing a single oversize disposition pass.
- **Checkpoint 002's R6 wrap-budget discipline.** Pinning marker overhead at ~5–7% of post-Phase-1.2 baseline forced this checkpoint to be structural rather than line-level. Without R6's bound, the cleanup could have shipped without a real ceiling on Phase 2.1's added weight; with it, every compression has a budget contribution.
- **The `MUST DO before responding` directive — broken in placement, right in intent.** The MUST-DO is the right operational test (read end-to-end, fresh, every session). The fix isn't to remove or weaken it; the fix (B2) is to foreground it so the imperative reaches Claude before competing context arrives.
- **TODO.txt as authoritative next-step source.** Rule 9's "TODO.txt is what is next; the git log is what was done" framing is exactly the right separation. The drift this checkpoint addresses (R7 — banner duplicating TODO.txt; R8 — proposed callouts duplicating TODO.txt) is structural agreement on the rule, applied to surfaces that violated it.

## Next steps

1. Jamie marks each finding inline (AUDIT NOTE blocks above). B1, B2, R1–R10, N1–N4 — sixteen markings total.
2. Re-run `/design-review`. Stage 2 walks each disposition, appends rows to the Disposition log, and asks whether to land the doc or open Addendum 1 (e.g. if any marking asks for further investigation).
3. After landing, Phase 2.1 build session opens with Prompt 2.1 in [`docs/CLAUDE_CODE_PROMPTS.md`](../CLAUDE_CODE_PROMPTS.md), armed with cleaned rules + CLAUDE.md content and (per R10) an explicit pre-marker migration scope item.

## Disposition log

*Stage 2 fills this section. Empty until Stage 2's first walk.
Rows accumulate append-only across rounds — later-round
dispositions on the same finding ID supersede earlier ones for
the purposes of Stage 2's landing application.*

| Round | Finding | Disposition | Reason |
|-------|---------|-------------|--------|
| <empty until Stage 2> | | | |

## Sign-off Summary

*Filled when Stage 2 lands the doc. Jamie does not edit this
section — sign-offs go inline above.*

| ID | Final disposition |
|----|-------------------|
| <empty until landing> | |

## Follow-up actions landed

*Filled when Stage 2 lands the doc. Lists every doc edit and
every TODO that landed during the landing pass.*
