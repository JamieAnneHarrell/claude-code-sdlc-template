---
checkpoint: 001
date: 2026-05-24
reviewer: Claude (Opus 4.7) with Jamie
status: LANDED 2026-05-24
trigger: Post-onboarding sanity check (between /onboard and /bootstrap)
---

# Design Review — Checkpoint 001

## Context

This is the post-onboarding sanity check for the
`claude-code-sdlc-template` source-of-truth project. `/onboard`
ran on 2026-05-24 and produced REQUIREMENTS / ARCHITECTURE /
PROJECT_PLAN / CLAUDE_CODE_PROMPTS, filled the `<!-- ONBOARD-FILL:
project-scope -->` block in `rules/project-rules.md`, rewrote
`rules/multi-agent-rules.md` to the explore-plus-plan rich shape,
and put both LICENSE files in place. The next gate this review
unblocks is `/bootstrap`.

The review covers all four planning docs, root `CLAUDE.md`,
`README.md`, `docs/design-decisions.md`, `docs/open-questions.md`,
the design intake at `docs/design/cc-template-product-spec.md`,
the rules files at root, and the `cc-template/CLAUDE.md`
placeholder. No prior `/design-review` checkpoints exist; this is
checkpoint 001.

Context-shaping framing from Jamie:

- This is the cc-template reviewing **itself** — the template
  under review is the template producing this review.
- The template is **already in use by downstream consumers**
  (notably `ds-niche-stream` and `ds-auto-dailies`), so changes
  that touch the dist have real blast radius.
- Findings should be sparing in the Blocker tier — only true
  show-stoppers for a brief phase cleanup. Critiques /
  optimizations / suggestions are framed as later-phase work or
  open-question follow-ups.

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

### Blockers — fix before /bootstrap

*No findings in this tier.*

### Recommendations — resolve before or during /bootstrap

#### R1. PROJECT_PLAN.md Phase 1 vs CLAUDE.md banner — operating-order drift

**Sources.** `CLAUDE.md` banner lines 7–11 ("Next step is
`/design-review` ... then `/bootstrap` ... then the first phase
prompt"); `docs/PROJECT_PLAN.md` Phase 1 lines 43–81 (Phase 1
deliverables include "/bootstrap run on the source", exit
criteria requires `BOOTSTRAP-STATUS: COMPLETE`, and the
"Design review checkpoint: before Phase 1.1 begins" sits
*after* the bootstrap deliverable); `TODO.txt` (matches the
CLAUDE.md banner ordering: design-review → bootstrap → Prompt
1.1).

**Problem.** Three documents disagree about when the post-
onboarding `/design-review` fires. The CLAUDE.md banner and
TODO.txt say it runs *between* `/onboard` and `/bootstrap` (which
is what's happening right now). PROJECT_PLAN.md Phase 1 puts
`/bootstrap` inside Phase 1's deliverables and sets the
checkpoint "before Phase 1.1 begins" — which reads as "after
`/bootstrap` runs and after Phase 1 closes." Operationally we're
running this review pre-bootstrap, so the PROJECT_PLAN.md
narrative is stale. The drift is small but it's the kind of thing
a downstream consumer reading these docs will trip over.

**Recommendation.** Edit `docs/PROJECT_PLAN.md` Phase 1 to
explicitly state the checkpoint sequencing: `/onboard` →
`/design-review` (post-onboarding sanity check, gates `/bootstrap`)
→ `/bootstrap` → Phase 1 closes → Phase 1.1 begins. Either
(a) keep `/bootstrap` inside Phase 1 deliverables and split the
exit criteria into pre-bootstrap (checkpoint runs) and
post-bootstrap (status flips to `COMPLETE`), or (b) move
`/bootstrap` out of Phase 1 into its own micro-phase (Phase 1.0
or similar). Option (a) is the smaller edit. Pick one and align
the narrative.

> AUDIT NOTE — JAH:
> _DECISION: (a)_

#### R2. `cc-template/CLAUDE.md` placeholder Reading-order list omits `docs/design/`

**Sources.** `cc-template/CLAUDE.md` lines 47–60 (Reading-order
section, five items: this file, TODO.txt, PROJECT_PLAN.md,
CLAUDE_CODE_PROMPTS.md, docs/test-plans/); root `CLAUDE.md`
lines 32–51 (Reading-order section, six items — adds
`docs/design/` at item 5 with the conditional phrasing "any
`/design-review` checkpoints in flight. A checkpoint with
`AWAITING-DECISIONS` frontmatter means Stage 1 ran but Stage 2
hasn't walked dispositions yet"); FR-6 (REQUIREMENTS.md).

**Problem.** The dist's `cc-template/CLAUDE.md` placeholder has
the test-plans reading-order entry but not the design/ entry.
When a downstream consumer runs `/onboard`, their generated
`CLAUDE.md` inherits this list. The result: their next-session
reading order will direct them to `docs/test-plans/` for
in-flight test plans but not to `docs/design/` for in-flight
design-review checkpoints. The reading-order convention is "read
this if it's relevant," gated by the AWAITING-DECISIONS
frontmatter signal — exactly parallel to the test-plans entry.
Per Jamie's clarification: TODO.txt is the definitive
next-action, so this is a cleanup (not a /bootstrap blocker), but
the asymmetry between the two recurring-artifact entries is the
kind of small drift that compounds.

**Recommendation.** Edit `cc-template/CLAUDE.md` to add a
`docs/design/` reading-order entry that mirrors the root CLAUDE.md
phrasing (item 5: "design intake plus any `/design-review`
checkpoints in flight. A checkpoint with `AWAITING-DECISIONS`
frontmatter means Stage 1 ran but Stage 2 hasn't walked
dispositions yet"). One-line edit. Same change applies if the
proposed Phase 1.2 cleanup (R4a–R4c) absorbs this work.

> AUDIT NOTE — JAH:
> _Accepted._

#### R3. Stale `open-questions.md` entries — already resolved on-disk

**Sources.** `docs/open-questions.md` "Multi-agent-rules.md output
shape under-specified in /onboard" entry (lines 74–112, claims
the rich Use/Don't-use shape and embedded briefing rule are
missing); `rules/multi-agent-rules.md` (the rich shape IS present
— lines 19–69 have Use Explore for / Don't use / Use Plan for /
Don't use Plan for / What's NOT enabled — and the briefing rule
is embedded at lines 72–96 rather than referenced from global
`~/.claude/CLAUDE.md`).

**Problem.** The open-questions.md entry's prescription ("address
via a /design-review checkpoint after /onboard runs … beef up
onboard.md's spec, OR ship the structured shape as a scaffold in
the placeholder") is already satisfied on-disk — the rich shape
landed during the source/dist restructure or the recent rules
work. The entry remains in open-questions.md as if still open.
Stale open-questions entries erode the signal of that file (when
you read it, you can't trust that entries reflect current reality
without spot-checking each one).

**Recommendation.** During Stage 2 landing, surface a TODO.txt
entry to do an `open-questions.md` cleanup pass next session:
remove or migrate-to-design-decisions the multi-agent-rules entry
(and the embedded-vs-referenced briefing-rule sub-question, which
is also resolved on-disk in favor of embed). Spot-check the other
remaining OQ entries (status callout, quick orientation,
placeholder qualifiers, source-only release helper,
regression-test automation) against on-disk state — they appear
correctly open, but verify. Open-questions.md is outside Stage 2's
edit scope, so this lands as a TODO, not a doc edit.

> AUDIT NOTE — JAH:
> _Accepted._

#### R4a. Add Phase 1.2 cleanup — rules and CLAUDE.md context-size optimization

**Sources.** Rules + CLAUDE.md line counts (aggregate): root
`rules/coding-session-rules.md` (~222 lines), `design-philosophy-
rules.md` (~98), `project-rules.md` (~128), `multi-agent-rules.md`
(~97), plus `environment-rules.md` and `testing-rules.md`
(present, sizes unaudited); root `CLAUDE.md` (~297 lines);
`cc-template/CLAUDE.md` placeholder (~88 lines). NFR-6 (no
runtime; markdown-only). Jamie's concern: "It's possible that all
rules get read so we may need a cleanup phase soon if the rules
are too big or redundant."

**Problem.** Universal rules + CLAUDE.md content collectively run
into the high hundreds / low thousands of lines. Every project
seeded from the dist inherits this footprint, and every session
that re-reads "all rules every session" (as
`CLAUDE.md` Collaboration-rules section directs for
`coding-session-rules.md` and `design-philosophy-rules.md`) pays
that token cost. Some content is plausibly redundant — rule 7's
commit handoff section alone is ~60 lines of PowerShell quoting
guidance; `design-philosophy-rules.md` overlaps with global
`~/.claude/CLAUDE.md` content (which downstream consumers may not
have). No measurement of how much could be cut while preserving
load-bearing content has been done.

**Recommendation.** Add a Phase 1.2 (Cleanup) to PROJECT_PLAN.md
and an accompanying Prompt 1.2 to CLAUDE_CODE_PROMPTS.md. Scope:
(1) measure current rules + CLAUDE.md line counts and identify
load-bearing vs trimmable content; (2) target a meaningful
reduction (e.g. 25–35%) without removing any invariant; (3) for
rules that overlap with global `~/.claude/CLAUDE.md`, decide
whether to keep self-contained (downstream consumers don't have
Jamie's globals) or trim with explicit "see global" notes;
(4) ship to both root and `cc-template/`. Land before Phase 2.1
(refresh-from-repository) so the cleaner content ships first.
Sequencing: Phase 1.1 (BLOCKED disposition) first, then Phase 1.2
(this cleanup), then Phase 2.x.

> AUDIT NOTE — JAH:
> _Accepted._

#### R4b. Add to Phase 1.2 — tighten rule 7 commit-message guidance

**Sources.** `rules/coding-session-rules.md` rule 7 commit
handoff format section (lines 78–155, ~80 lines, mostly about
PowerShell quoting and HEREDOC negation; the brevity guidance
itself is one sub-bullet at line 122–127: "Subject under 72
chars... Body is at most a few sentences plus an optional short
bullet list"); Jamie's concern: "I continue to see very long
commit messages, so would like to investigate reducing the size
of commit messages — they should be short and refer back to the
project documents, rather than fully self-standing."

**Problem.** Rule 7's quoting/HEREDOC mechanics dominate the
section; the brevity-and-doc-references norm is buried in a
sub-bullet. In practice (per Jamie's observation across sessions)
commit messages keep landing long and self-standing rather than
referencing FR-N / ARCHITECTURE.md / phase docs. The current
guidance is correct but doesn't have enough emphasis to change
default behavior.

**Recommendation.** Within Phase 1.2's cleanup pass (depends on
R4a being Accepted), rework rule 7's commit-message guidance to
foreground brevity: (a) make "subject + zero or one body
sentence" the default; (b) explicitly require body bullets to
cite doc IDs (FR-N, NFR-N, ARCHITECTURE §N, Prompt N) over
restating context; (c) the multi-paragraph bullet list is the
rare case, not the default. Decide independently of R4c so each
sub-finding stands on its own.

> AUDIT NOTE — JAH:
> _Accepted._

#### R4c. Add to Phase 1.2 — `cc-template/CLAUDE.md` placeholder qualifier cleanup

**Sources.** `docs/open-questions.md` "Placeholder qualifiers
persist into configured CLAUDE.md" entry (lines 172–208,
recommends option A — reword the placeholder to read correctly
in both states); `cc-template/CLAUDE.md` line 47 ("Reading order
at session start (configured projects)"), lines 81–83
("Project-specific environment is filled in by `/bootstrap`",
"filled in by `/onboard` based on the chosen mode").

**Problem.** The dist's `cc-template/CLAUDE.md` uses pre-onboard
qualifiers — "(configured projects)" parenthetical, "filled in
by /bootstrap" / "filled in by /onboard" phrasings — that become
stale and noisy in every configured downstream CLAUDE.md.
`/onboard` and `/bootstrap` specs say "keep collaboration rules
and references unchanged," so these phrases persist forever.
The open-questions entry recommends KISS option A: reword once
so it reads correctly pre- and post-onboard.

**Recommendation.** Within Phase 1.2's cleanup pass (depends on
R4a being Accepted), apply option A from the OQ entry: drop the
"(configured projects)" parenthetical; rephrase the "filled in
by" qualifiers to be stable across pre- and post-onboard reading.
Apply to both root and `cc-template/` for parity. Couples
naturally with R4a since both touch the same CLAUDE.md file.

> AUDIT NOTE — JAH:
> _Accepted._

### Notes — acceptable now, revisit later

#### N1. In-flight artifact status callout (defer formally)

**Sources.** `docs/open-questions.md` "In-flight artifact status
callout at top of CLAUDE.md" entry (lines 114–139); ds-niche-
stream CLAUDE.md (referenced in the OQ entry, not read here).

**Problem.** ds-niche-stream evolved a 🟡 callout at the top of
its CLAUDE.md summarizing in-flight artifact state ("Phase 1.3
exit AWAITING-DISPOSITIONS — paused on ... Next action is
/design-review ..."). Neither this project nor ds-niche-stream is
blocked by its absence (per Jamie). The OQ entry proposes a
`/wind-down` enhancement to detect mid-flight checkpoints and
write a status callout block. This is potentially-useful template
evolution that doesn't need to happen now.

**Recommendation.** Formally defer with a target: keep the OQ
entry as-is for now (no doc edit) and revisit at a future
`/design-review` checkpoint — e.g. between Phase 2.1
(/refresh-from-repository, since that command also touches
CLAUDE.md) and Phase 2.2 (/write-user-documentation). If accepted
now, the disposition adds nothing to PROJECT_PLAN.md or other
landing-scope docs.

> AUDIT NOTE — JAH:
> _Accepted._

#### N2. Project quick orientation section (defer formally)

**Sources.** `docs/open-questions.md` "Project quick orientation
section in onboarded CLAUDE.md" entry (lines 141–170); ds-niche-
stream CLAUDE.md (referenced in the OQ entry).

**Problem.** ds-niche-stream evolved a rich Project-quick-
orientation section in CLAUDE.md (admin/public stack split, DB
engine, source-tree location, format/lint commands). Neither
`/onboard` nor `/bootstrap` currently produces this. KISS
counter-argument: don't add a section that every project then has
to maintain. Jamie's framing: "interesting but should defer."

**Recommendation.** Formally defer. Revisit at the same future
checkpoint as N1, when more downstream projects have accumulated
and the maintenance-vs-utility tradeoff is clearer. No
landing-scope doc edit.

> AUDIT NOTE — JAH:
> _Accepted._

#### N3. PROJECT_PLAN.md missing a Phase 2 parent header (cosmetic)

**Sources.** `docs/PROJECT_PLAN.md` — phases go Phase 0, Phase 1,
Phase 1.1, Phase 2.1, Phase 2.2, Phase 3. No Phase 2 parent
header. The design intake at
`docs/design/cc-template-product-spec.md` (lines 133–144) does
have "**Phase 2 — Planned project enhancements.** Known items to
add and improve:" as a parent.

**Problem.** Cosmetic only. The plan's phase tree reads as a flat
list rather than a hierarchy. Downstream is unaffected.

**Recommendation.** Optional: add a `## Phase 2 — Planned project
enhancements (roadmap)` parent header between Phase 1.1 and Phase
2.1 in PROJECT_PLAN.md, with a one-line description. Or leave as
is — the flat list is also readable.

> AUDIT NOTE — JAH:
> _DECISION: Add now._

## What the design got right (preserve)

- **Iterative addendum lifecycle on both recurring commands.**
  `/design-review` and `/exit-test-plan` share the same Stage 1
  initial / Stage 1 addendum / Stage 2 walk / land-or-open-next
  shape. The pattern proved itself on `ds-niche-stream` before
  being lifted into the template, and the specs are unusually
  thorough about edge cases (round detection from the
  Disposition log, placeholder-string parsing, commit-handoff
  discipline only on artifact boundaries).
- **Load-bearing invariants section in `CLAUDE.md`.** The
  ~170-line section explicitly names every convention that
  stage detection and the recurring lifecycles depend on, with
  "audit the whole chain first" warnings. This is the kind of
  thing that prevents subtle regressions when invariants drift.
- **File-ownership-doesn't-overlap discipline.** Each command
  states what others own; `/design-review` owns checkpoints
  exclusively, `/exit-test-plan` owns test plans exclusively.
  Codified as NFR-8 and called out in each command's "What this
  command does NOT do" section.
- **Source/dist split with explicit duplication discipline.**
  Universal content is identical between root and `cc-template/`,
  rationale captured in `docs/design-decisions.md`. The
  alternatives (gitignored dist, sibling directory, separate
  repo, `.gitattributes export-ignore`) are all called out as
  rejected with reasons.
- **Three-status-comments-for-configuration / per-file-
  frontmatter-for-recurring separation.** The recurring commands
  deliberately don't own status comments in CLAUDE.md; per-file
  frontmatter status is the source of truth. Avoids conflating
  the configuration ritual with the recurring lifecycle.
- **Dual license, explicitly reasoned.** MIT for the dist
  (commercial reuse allowed), CC BY-NC-ND for source-only
  project-management docs (not for redistribution). The licensing
  rationale captures the "this is a template, not a product"
  intent precisely.

## Next steps

1. Jamie marks each finding inline (AUDIT NOTE blocks above).
2. Re-run `/design-review`. Stage 2 walks each disposition,
   appends rows to the Disposition log, and asks whether to land
   the doc or open another addendum round.
3. After landing, the next session runs `/bootstrap`, then pastes
   the first prompt from `docs/CLAUDE_CODE_PROMPTS.md` (Prompt
   1.1, or — if R4a is Accepted — a new Prompt 1.2 that lands
   first).

## Disposition log

*Stage 2 fills this section. Empty until Stage 2's first walk.
Rows accumulate append-only across rounds — later-round
dispositions on the same finding ID supersede earlier ones for
the purposes of Stage 2's landing application.*

| Round | Finding | Disposition | Reason |
|-------|---------|-------------|--------|
| Original | R1 | Decision | Pick option (a): split Phase 1 exit criteria pre/post bootstrap; restate checkpoint sequencing as /onboard → /design-review → /bootstrap → Phase 1 closes. |
| Original | R2 | Accepted | Add `docs/design/` Reading-order entry to `cc-template/CLAUDE.md` mirroring root CLAUDE.md item 5; folded into Prompt 1.2 scope (out of Stage 2 edit scope). |
| Original | R3 | Accepted | Surface TODO for next-session `open-questions.md` cleanup pass: remove resolved multi-agent-rules + briefing-rule entries; spot-check remainder. |
| Original | R4a | Accepted | Add Phase 1.2 (Cleanup) to PROJECT_PLAN.md and Prompt 1.2 to CLAUDE_CODE_PROMPTS.md; scope = rules + CLAUDE.md context-size reduction. |
| Original | R4b | Accepted | Folded into Phase 1.2 scope: rework rule 7 commit-message guidance to foreground brevity and doc-id citations. |
| Original | R4c | Accepted | Folded into Phase 1.2 scope: apply OQ option A to cc-template/CLAUDE.md placeholder qualifiers (drop "(configured projects)" and "filled in by" phrasings). |
| Original | N1 | Deferred | Revisit at a future `/design-review` checkpoint (pre-Phase-2.1 or pre-Phase-2.2); OQ entry remains as-is for now. |
| Original | N2 | Deferred | Revisit at same future checkpoint as N1; OQ entry remains as-is for now. |
| Original | N3 | Decision | Add `## Phase 2 — Planned project enhancements (roadmap)` parent header between Phase 1.x and Phase 2.1. |

## Sign-off Summary

| ID | Final disposition |
|----|-------------------|
| R1 | Decision: option (a) — split Phase 1 exit criteria pre/post bootstrap; restate checkpoint sequencing |
| R2 | Accepted (folded into Prompt 1.2 scope) |
| R3 | Accepted (surfaced as TODO; open-questions.md is out of Stage 2 edit scope) |
| R4a | Accepted (Phase 1.2 + Prompt 1.2 added) |
| R4b | Accepted (folded into Prompt 1.2 scope) |
| R4c | Accepted (folded into Prompt 1.2 scope) |
| N1 | Deferred (revisit pre-Phase-2.1 or pre-Phase-2.2; OQ entry remains) |
| N2 | Deferred (revisit at same future checkpoint as N1; OQ entry remains) |
| N3 | Decision: add Phase 2 parent header now |

## Follow-up actions landed

- R1: edited `docs/PROJECT_PLAN.md` Phase 1 — added Checkpoint sequencing note, split Exit criteria into pre-bootstrap and post-bootstrap, added the post-onboarding `/design-review` to the deliverables list.
- R4a: edited `docs/PROJECT_PLAN.md` — inserted `## Phase 1.2 — Rules and CLAUDE.md cleanup pass` between Phase 1.1 and the new Phase 2 header.
- R4a: edited `docs/CLAUDE_CODE_PROMPTS.md` — inserted Prompt 1.2 (Rules and CLAUDE.md cleanup pass), citing R4a/R4b/R4c/R2 in its Read-first and Scope sections.
- R4b, R4c, R2: folded into Prompt 1.2's Scope (rule 7 rework, placeholder qualifier cleanup, `docs/design/` Reading-order entry in `cc-template/CLAUDE.md`).
- R2: scope addition only — `cc-template/CLAUDE.md` edit deferred to Phase 1.2 (out of Stage 2 edit scope).
- N3: edited `docs/PROJECT_PLAN.md` — inserted `## Phase 2 — Planned project enhancements (roadmap)` parent header between Phase 1.x and Phase 2.1.
- R1, R4a sequencing: edited `docs/CLAUDE_CODE_PROMPTS.md` Prompt 1.1 — "Before running this prompt" reworded to reflect checkpoint-001 sequencing decisions; revisions-footer entry added.
- R3: TODO surfaced for next-session `open-questions.md` cleanup pass (`docs/open-questions.md` is outside Stage 2's edit scope).
- N1, N2: recorded as deferred in this checkpoint's Disposition log + Sign-off Summary; OQ entries unchanged.
