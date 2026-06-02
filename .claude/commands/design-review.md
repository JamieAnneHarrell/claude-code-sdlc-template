---
description: Run a sign-off-ready design review checkpoint at a high-risk transition. Stage 1 produces findings; stage 2 walks dispositions and either lands the doc revisions or opens another addendum round.
---

# /design-review

`/design-review` is the **fourth** template command, and the first
recurring one (alongside `/exit-test-plan`). Where `/onboard`,
`/bootstrap`, and `/deployment-plan` each run once during
configuration, `/design-review` runs at every high-risk transition
over the life of the project.

It answers *are we still building the right thing the right way?*
and produces a sign-off-ready checkpoint document modeled on
`c:\Users\jamie\Documents\code\ds-niche-stream\docs\design\design-review-checkpoint-002.md`
(which exercised the iterative addendum pattern this command now
specifies).

This command is the deliberate exception to CLAUDE.md's
"don't add a fourth command" guideline. The justification: design
drift between planning artifacts and implementation is expensive to
catch late, and a structured checkpoint pattern (severity-tiered
findings, inline audit decisions, follow-up doc revisions) has
already proven its value on `ds-niche-stream`. See
`CLAUDE.md` § Load-bearing invariants for the recurring-
vs-configuration distinction.

This command runs in the current working directory and writes files
relative to it.

**Required reading before Step 0:** read
`rules/coding-session-rules.md` and
`rules/design-philosophy-rules.md` in full. Rule 4 (no unsolicited
design decisions) is load-bearing — Stage 1 is findings-only;
doc revisions only land at Stage 2's "land" path, and only after
Jamie's inline sign-off has explicitly approved each change.

---

## Two stages, auto-detected; iterative across rounds

`/design-review` runs in two stages. The full lifecycle is
**iterative**: a checkpoint document can grow with multiple
addendum rounds before it lands. The pattern mirrors
`/exit-test-plan`'s addendum lifecycle.

- **Stage 1 — Author findings.** Two branches:
  - **Initial branch**: runs when no checkpoint exists or the newest
    is `LANDED`. Writes a new
    `docs/design/design-review-checkpoint-NNN.md` with the original
    Findings (Round 1), an empty `## Disposition log` placeholder,
    and the trailing Sign-off Summary / Follow-up sections. Status
    `AWAITING-DECISIONS`. Edits no other doc (rule 4).
  - **Addendum branch**: runs when the newest checkpoint is
    `AWAITING-DECISIONS`, every AUDIT NOTE block in the latest
    round is marked, and the Disposition log already has rows for
    that round (signal that Stage 2 walked it and Jamie picked
    "open another round"). Appends a new `## Addendum N — date`
    section with re-opened findings (suffix `-AN`, e.g. `B1-A1`)
    and any new findings (continuing the global ID space). Status
    stays `AWAITING-DECISIONS`. Edits no other doc.
- **Stage 2 — Walk dispositions; land or open another round.** Runs
  when the newest checkpoint is `AWAITING-DECISIONS`, every AUDIT
  NOTE block in the latest round is marked, and the Disposition
  log does NOT yet have rows for that round. Triages the marked
  findings as a batch — surfacing only genuinely ambiguous
  markings — appends rows to the Disposition log, then asks Jamie
  explicitly: **land the document or open Addendum N+1?**
  - **Land**: applies every accumulated final disposition (Accepted
    / Accepted with caveats / DECISION) as doc revisions to
    REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN / CLAUDE_CODE_PROMPTS,
    fills the Sign-off Summary table and Follow-up actions section,
    flips the frontmatter to `LANDED <date>`, and invokes
    `/wind-down` to wrap up the session.
  - **Open another round**: leaves the frontmatter at
    `AWAITING-DECISIONS`, surfaces a TODO describing what the
    addendum needs (research, follow-up clarification, off-the-
    shelf investigation, etc.), and surfaces NO commit handoff.
    Wind-down stages mid-iteration changes at session close.

There is **no limit** on the number of addendums. The doc is not
`LANDED` until every round's findings have a final disposition and
Jamie picks "land" at a Stage 2 walk.

Stage detection avoids requiring Jamie to manually flip a status —
the markup, the Disposition log, and Stage 2's land-or-addendum
decision together signal readiness.

### Arguments

Progressive disclosure: bare invocation handles the happy path.

- `/design-review` — auto-detect stage and branch per above.
- `/design-review --new` — force Stage 1 *initial* (new checkpoint)
  even if a prior `AWAITING-DECISIONS` checkpoint exists. Rare;
  surfaces a warning that the prior checkpoint will be left
  unfinished.
- `/design-review --trigger "<reason>"` — supply the frontmatter
  trigger string explicitly. If omitted, Claude infers from project
  state (e.g. "Post-onboarding sanity check" if `BOOTSTRAP-STATUS`
  is still `UNCONFIGURED`; "Between Prompt N-1 and Prompt N" if
  CLAUDE_CODE_PROMPTS.md has a checkpoint marker before the next
  unlanded prompt). Used by Stage 1 initial only — the addendum
  branch inherits the trigger from the existing file.

---

## Step 0: Verify prerequisites and detect stage

Read `CLAUDE.md`:

- `<!-- ONBOARD-STATUS: ... -->` must be `COMPLETE <date>`. If it's
  `UNCONFIGURED`, refuse: "Design review requires `/onboard` to have
  run first — it needs `docs/REQUIREMENTS.md` and
  `docs/ARCHITECTURE.md` to review against. Run `/onboard`, then come
  back."
- `<!-- BOOTSTRAP-STATUS: ... -->` and
  `<!-- DEPLOYMENT-PLAN-STATUS: ... -->` are not gated. Design review
  is meaningful from the moment requirements + architecture exist.

Then list `docs/design/design-review-checkpoint-*.md` (sorted
lexicographically — the zero-padded NNN convention guarantees this
sorts as numeric order through 999). Newest file determines stage:

- **No matches** → Stage 1 *initial*, with N = 001.
- **Newest is `LANDED <date>`** → Stage 1 *initial*, with
  N = previous + 1 (zero-padded to 3 digits).
- **Newest is `AWAITING-DECISIONS`**:
  - If `--new` was passed → Stage 1 *initial* with N = previous + 1,
    surface a warning that the prior checkpoint is being left
    unfinished.
  - Otherwise → identify the **latest round** in the document. The
    latest round is the most recent `## Addendum N — YYYY-MM-DD`
    section if any exists; otherwise it is the original Findings
    section (Round 1). Then:
    - If any AUDIT NOTE block in the latest round is still the
      placeholder → refuse with a list of unmarked finding IDs from
      that round (e.g. "B2-A1 and R4-A1 in Addendum 1 are still
      unmarked. Fill them in inline and re-run."). Do not check
      AUDIT NOTE blocks in earlier rounds — those are historical
      record once their dispositions landed in the Disposition log.
    - If every AUDIT NOTE block in the latest round is marked, look
      at the `## Disposition log` table:
      - If the log has no rows whose `Round` column matches the
        latest round (round identifier is `Original` for the Round 1
        Findings section, or `Addendum N` for the latest
        `## Addendum N` section) → **Stage 2** (walk this round's
        dispositions).
      - If the log already has rows for the latest round → **Stage 1
        addendum branch** (Jamie is invoking after Stage 2 picked
        "open another round").
- **Newest has any other status** → unexpected state; surface and
  ask Jamie.

The placeholder line that distinguishes "unmarked" is exactly:

```
> _[UNMARKED — replace this line with your decision per the legend above]_
```

Any AUDIT NOTE block whose body is still that placeholder counts as
unmarked. Anything else (including a manually edited line that says
"Accepted", "Defer Approved", etc.) counts as marked.

---

# Stage 1 — Author findings

Stage 1 has two branches:

- **Initial branch** (Steps S1.1 through S1.9) — used when the
  checkpoint is being created (no prior file, or newest is
  `LANDED`, or `--new` was passed). Writes the whole document
  including the empty Disposition log placeholder and the empty
  Sign-off Summary placeholder.
- **Addendum branch** (Step S1.A) — used when an
  `AWAITING-DECISIONS` checkpoint exists, every AUDIT NOTE in the
  latest round is marked, and the Disposition log already has rows
  for that round. Appends a new `## Addendum N — date` section to
  the existing document. Does not rewrite anything above the new
  addendum.

The branches share the input-reading discipline (S1.1) and the
finding-composition rules (S1.4). The addendum branch skips S1.2
reconnaissance, S1.5 doc-template authoring (only appends), the
S1.6 REVIEWS.md update, and the S1.8 wind-down (mid-iteration — no
session wrap-up; wind-down stages pending changes only when Jamie
wraps the session).

## Step S1.1: Read project context

Load these in order. Skip files that don't exist (project may be
pre-`/bootstrap` or pre-`/deployment-plan`).

1. `docs/REQUIREMENTS.md`
2. `docs/ARCHITECTURE.md`
3. `docs/PROJECT_PLAN.md`
4. `docs/CLAUDE_CODE_PROMPTS.md` — pay attention to which prompts
   have landed (the "revisions since this prompt ran" footer is
   the marker `/wind-down` updates) and which checkpoint markers
   are present.
5. `README.md` — particularly the Developer setup section if it
   exists.
6. `docs/DEPLOYMENT.md` — if it exists.
7. Every file in `docs/design/` other than the folder's own
   `README.md` and prior checkpoint files. These are the
   authoritative design intake.
8. Every prior `design-review-checkpoint-*.md`. Prior decisions
   shape what's still open; do not re-litigate findings already
   marked Accepted or Defer Approved unless something has changed
   since.
9. **If the addendum branch is firing**: read the current
   checkpoint file end-to-end. The Disposition log tells you which
   findings are already finalized; the latest Addendum (or Round 1
   if none yet) tells you which markings asked for further
   investigation. The addendum you're about to author should
   re-open those threads with concrete research/clarification, plus
   any new findings the intervening session surfaced.

Do not start asking questions yet.

## Step S1.2: Reconnaissance (initial branch only)

State back to Jamie a 4–6 bullet summary covering:

- The trigger for this review (post-onboarding sanity check, between
  Prompt N-1 and Prompt N, ad-hoc, etc.). If `--trigger` was passed,
  use it verbatim; otherwise infer.
- The doc cluster being reviewed (which files; which phases of
  PROJECT_PLAN.md are in scope; which design-intake docs were
  consulted).
- Any prior checkpoints whose decisions are load-bearing for this
  review.
- The gate this review unblocks (e.g. "before `/bootstrap` runs",
  "before pasting Prompt 1", "before merging Phase 2").
- One or two areas where Claude already sees likely findings (do not
  pre-commit to a severity tier yet).

Don't proceed until Jamie confirms the picture.

## Step S1.3: Targeted questions

Ask via `AskUserQuestion`, batched into 1–3 dialogs of 3–4
questions each. Tailor to the trigger — a post-onboarding review
asks different questions than a between-prompts review. Skip any
category not relevant.

Cover when relevant:

- **Concerns Jamie wants surfaced.** Ask directly: "Is there
  anything you've been worried about in the design that you want
  this review to scrutinize specifically?" — this catches issues
  Claude wouldn't see from the docs alone.
- **Versions and dependency justifications.** When the trigger is
  post-onboarding or post-bootstrap, audit the version pins in
  `rules/environment-rules.md` and the dependency allowlist in
  `rules/project-rules.md` against
  `rules/environment-rules.md` § Version freshness. Ask Jamie
  before recommending bumps.
- **Architecture coherence.** Ask whether any architectural
  decision has shifted since the design intake (and so
  REQUIREMENTS / ARCHITECTURE may now be stale).
- **Phase scope.** When between phases, ask whether Phase N's
  exit criteria still match what was intended, and whether
  Phase N+1's prompt body is still consistent with the architecture
  as it stands now.

For the addendum branch, the question dialog is focused: which
prior-round AUDIT NOTE asks for further investigation, what the
intervening session surfaced as new concerns, and whether the
addendum scope should pull in any items deferred from earlier
rounds.

## Step S1.4: Compose findings

Three severity tiers:

- **Blockers (B1, B2, …)** — must fix before the gate this review
  unblocks. If a finding is a blocker, the gate doesn't open until
  it's resolved.
- **Recommendations (R1, R2, …)** — resolve before or during the
  gate. Not blockers; soft gates.
- **Notes (N1, N2, …)** — acceptable now, revisit later. These
  shouldn't gate progress; they're future-Claude's reading list.

Each finding has:

- **Title** (one line).
- **Sources** — concrete cross-doc links: `REQUIREMENTS.md FR-2`,
  `ARCHITECTURE.md §3`, `CLAUDE_CODE_PROMPTS.md Prompt 1 step 4`,
  design-intake-doc § N. Be specific enough that Jamie can verify
  in seconds.
- **Problem** — narrative, not bullets.
- **Recommendation** — actionable change Jamie can accept, accept
  with caveats, or defer. Phrase as "do X" not "consider doing X."
- **Inline AUDIT NOTE block** — see the template below.

Plus two non-finding sections at the end of Round 1: "What the
design got right (preserve)" and "Next steps." See template.

### Composition rules — load-bearing

These rules apply equally to Round 1 findings and to addendum
findings. They exist because Jamie needs to sign each decision off
independently with detailed inline comments — bundling defeats
that.

- **One finding = one decision.** Every finding gets exactly one
  `AUDIT NOTE — JAH:` block, and that block records exactly one
  decision. If a finding's "Recommendation" section spans N
  independently-decidable items (e.g. "add FR-41, add FR-42, add
  FR-43, where each could be Accepted / Rejected / Deferred on
  its own"), split into N findings. Numbering: use suffixed IDs
  (`B1a`, `B1b`, `B1c`) when the items share a narrative root and
  Jamie should read them together; use distinct top-level IDs
  (`B1`, `B2`, `B3`) when the items don't share a root. Never
  bundle multiple independent recommendations under one AUDIT
  NOTE.
- **First-level decisions are primary.** When a finding presents
  an options-with-recommendation shape (Option A / Option B /
  Option C), analyze the first-level options deeply but do NOT
  fully spec downstream sub-options that depend on which first-
  level option is picked. Surface downstream implications in 1-2
  sentences each — *what does picking A eliminate, lock in, or
  open up?* — so Jamie sees the consequence without wading
  through three sub-trees of analysis. If a downstream decision
  becomes load-bearing after the first-level pick, surface it as
  a separate finding in the next addendum.
- **Cite requirements concretely.** FR-N, NFR-N, ARCHITECTURE.md
  §N, PROJECT_PLAN.md Phase N, etc. The Sources line is what
  makes Jamie's audit fast.
- **Phrase Recommendations as "do X" not "consider X."** A
  finding's Recommendation is the exact change Stage 2 will apply
  if Jamie accepts. Wishy-washy wording forces Stage 2 to guess.

For the addendum branch specifically:

- **Re-opened findings use suffix `-AN`** where N is the addendum
  number. Example: Round 1 had B2 marked `REJECTED with directive
  to investigate off-the-shelf RBAC`; Addendum 1 re-opens it as
  `B2-A1` with concrete research and a re-scoped recommendation.
  Suffix the original ID exactly — a Round 1 R3 becomes `R3-A1`
  in Addendum 1, then `R3-A2` if it re-opens again in Addendum 2.
- **New findings continue the global ID space.** If Round 1 ended
  at B3 / R6 / N1 and Addendum 1 introduces two new blockers,
  they are `B4` and `B5`, not `B1-A1` / `B2-A1` (which would
  collide with re-opened IDs). Round 1's last-used number per
  tier is the starting point.
- **Round 2+ supersedes earlier rounds for re-opened items.**
  When Stage 2 walks the latest round's dispositions, the
  disposition on `B1-A2` supersedes whatever `B1-A1` was marked,
  which supersedes `B1`. Older rounds' AUDIT NOTE blocks remain
  in the doc as historical record but are not re-applied.

## Step S1.5: Write the checkpoint document (initial branch)

Filename: `docs/design/design-review-checkpoint-NNN.md` where NNN is
the zero-padded 3-digit number from Step 0. Pad width is fixed at 3
so files sort lexicographically as numeric order through 999.

Frontmatter status: `AWAITING-DECISIONS`.

Use this template verbatim, filling in placeholders:

````markdown
---
checkpoint: NNN
date: YYYY-MM-DD
reviewer: Claude (<model name>) with Jamie
status: AWAITING-DECISIONS
trigger: <one-line>
---

# Design Review — Checkpoint NNN

## Context

<2–4 paragraphs: what's being reviewed, why now, which docs were
read, what gate this unblocks. Include explicit references to prior
checkpoints whose decisions shape this one, if any.>

## How to mark up this review

For each finding below, replace the placeholder line under
`AUDIT NOTE — JAH:` with one of:

- `Accepted`
- `Accepted with caveats: <your caveats>`
- `Defer Approved`
- `DECISION: <explicit choice>`
- `REJECTED: <your reframing prose>` — none of the listed
  recommendations is satisfactory. This opens another addendum
  round; your rejection prose is the authoritative input the next
  addendum reframes the finding from.

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

### Blockers — fix before <gate>

#### B1. <title>

**Sources.** <links>
**Problem.** <narrative>
**Recommendation.** <actionable change>

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### B2. <title>
...

### Recommendations — resolve before or during <gate>

#### R1. <title>

**Sources.** <links>
**Problem.** <narrative>
**Recommendation.** <actionable change>

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### R2. <title>
...

### Notes — acceptable now, revisit later

#### N1. <title>

**Sources.** <links>
**Problem.** <narrative>
**Recommendation.** <actionable change>

> AUDIT NOTE — JAH:
> _[UNMARKED — replace this line with your decision per the legend above]_

#### N2. <title>
...

## What the design got right (preserve)

- <bullet — affirmative, names a decision worth preserving>
- <bullet>

## Next steps

1. Jamie marks each finding inline (AUDIT NOTE blocks above).
2. Re-run `/design-review`. Stage 2 walks each disposition,
   appends rows to the Disposition log, and asks whether to land
   the doc or open another addendum round.
3. After landing, the next session pastes <next prompt or gate>.

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
````

If a tier has zero findings in Round 1, write the section header
and a single italic line: `*No findings in this tier.*` — do not
omit the section. The shape stays consistent across reviews.

## Step S1.6: Update the REVIEWS.md index (initial branch only)

`docs/design/REVIEWS.md` is a one-line-per-checkpoint index. If it
doesn't exist, create it with this header:

```markdown
# Design Review Index

One line per checkpoint. Status values: AWAITING-DECISIONS,
LANDED.

```

Append (do not rewrite) one line for the new checkpoint:

```
- [checkpoint-NNN](design-review-checkpoint-NNN.md) — YYYY-MM-DD — <trigger> — AWAITING-DECISIONS
```

Stage 1 addendum does NOT touch REVIEWS.md — the index line stays
`AWAITING-DECISIONS` for the entire iterative loop. Stage 2 only
updates REVIEWS.md when the doc lands.

## Step S1.7: Update TODO.txt (initial branch)

`TODO.txt` is gitignored and personal. Rewrite it (do not append) so
that the first entry under "What's next" is the design-review
follow-up:

```
- Mark up docs/design/design-review-checkpoint-NNN.md (decide each
  finding inline per the legend in the doc), then re-run
  /design-review. Stage 2 will walk each disposition and ask
  whether to land the doc or open another addendum round.
```

Carryover entries stay below in priority order. Do not invent
carryovers; preserve whatever was already there.

## Step S1.8: Surface git commands (rule 7) — initial branch only

**Re-read rule 7's "Commit handoff format" section in
`rules/coding-session-rules.md` end-to-end before drafting the
message.** The brevity bar needs to be active in working memory
before you write a single word.

Stage only the new files:

```
git status
```

```
git add docs/design/design-review-checkpoint-NNN.md docs/design/REVIEWS.md
```

```
git status
```

```
git commit -m "Design review checkpoint NNN — awaiting decisions

Trigger: <one-line>.
Findings: <B count> blockers, <R count> recommendations, <N count> notes."
```

Do not stage `TODO.txt` (it's gitignored). Do not run any commands.

The Stage 1 addendum branch does NOT surface a commit handoff —
addendum authoring is mid-iteration of an existing artifact.
Wind-down stages any pending changes at session close.

## Step S1.9: Final report (initial branch)

Tell Jamie:

- Path to the new checkpoint file.
- Counts per tier (`<B count> blockers, <R count> recommendations,
  <N count> notes`).
- The marking-up legend (echo the five options).
- That re-running `/design-review` after markup will enter Stage 2,
  which walks each disposition and asks whether to land or open
  another addendum.

End with: "Design review checkpoint NNN written. Mark up the
findings inline (one decision per AUDIT NOTE block), then re-run
`/design-review` to walk dispositions."

---

## Step S1.A: Author Addendum N (addendum branch)

Runs instead of S1.2 / S1.5 / S1.6 / S1.7 / S1.8 / S1.9 when Step 0
routed to the addendum branch. The checkpoint file already exists;
this step appends a new `## Addendum N — YYYY-MM-DD` section to it
without rewriting anything above. Mid-iteration — no commit
handoff.

1. **Determine the next addendum number N.** Count existing
   `## Addendum M — ...` sections in the file; N is the next
   integer (1 if none exist yet).

2. **Identify what this addendum re-opens and what it adds.** Walk
   the Disposition log and the latest round together:
   - Every REJECTED marking, plus every disposition whose Reason
     cites "further review", "investigate X", "DECISION: <choice
     that asks for more analysis>", or any other marking that
     didn't close the thread, should be re-opened in the addendum
     with concrete research / clarification, under a new ID
     suffixed `-AN`. For a REJECTED finding, the rejection prose
     IS the authoritative reframe input — re-author the finding
     from Jamie's stated objection; do not generate fresh
     alternatives from scratch as if the finding were new.
   - Any new findings the intervening session surfaced (off-the-
     shelf research, a related concern that came up while
     investigating, a follow-up question Jamie raised) get added
     under new top-level IDs continuing the global ID space.
   - Items already finalized in earlier rounds (Accepted, Defer
     Approved with no caveat asking for more) stay closed and do
     NOT re-appear in the addendum.

   State this list back to Jamie as "Addendum N will re-open:
   <ID list>. Addendum N will add: <ID list>." Wait for
   confirmation, additions, or removals.

3. **Compose findings using the same composition rules as Step
   S1.4.** Re-opened findings use the original ID with `-AN`
   suffix (`B1-A1`, `R3-A2`); new findings continue the global ID
   space. One AUDIT NOTE block per finding; first-level decisions
   primary; downstream implications named in 1-2 sentences.

4. **Append the addendum section** at the end of the file using
   this template:

   ````markdown

   ---

   ## Addendum N — YYYY-MM-DD

   ### Why this addendum exists

   <one short paragraph: which prior-round markings asked for
   further investigation, what the intervening session surfaced,
   what the addendum's scope is.>

   ### Round 1 dispositions that stand (not re-opened here)

   <bulleted list naming the findings whose earlier-round
   disposition is final and not being revisited. This is for
   Jamie's audit — confirms nothing slipped silently.>

   ### Off-the-shelf / investigation summary

   <if the addendum is driven by research between rounds (e.g.
   evaluating a package, comparing two implementation patterns),
   summarize the research here in 1-3 paragraphs per option, with
   capability + integration sketch + cost. Skip if the addendum
   is purely clarifying re-opens with no external research.>

   ### How to mark up Addendum N

   Each finding below has its own `AUDIT NOTE — JAH:` block.
   Replace each placeholder line per the legend at the top of the
   document. Addendum N dispositions supersede earlier rounds for
   re-opened findings.

   ### Findings

   #### Blockers (re-opened + new)

   ##### B1-A1. <title> [re-opened from B1]

   **Sources.** <links, including reference to the original B1>
   **Problem.** <narrative — what changed since B1's marking;
   what the research surfaced>
   **Recommendation.** <actionable change>

   > AUDIT NOTE — JAH:
   > _[UNMARKED — replace this line with your decision per the legend above]_

   ##### B4. <title> [new]
   ...

   #### Recommendations (re-opened + new)
   ...

   #### Notes (re-opened + new)
   ...
   ````

5. **Keep frontmatter at `AWAITING-DECISIONS`.** The status
   doesn't change — the addendum needs to be marked and walked
   before the doc can land.

6. **Update `TODO.txt`** so the first entry under "What's next"
   becomes:

   ```
   - Mark up docs/design/design-review-checkpoint-NNN.md
     Addendum N (decide each finding inline per the legend at the
     top of the doc). Then re-run /design-review. Stage 2 walks
     Addendum N's dispositions and asks whether to land the doc
     or open Addendum N+1.
   ```

   Carryover entries stay below in priority order.

7. **Final report.** Tell Jamie: path to the checkpoint file, the
   addendum number, the count of re-opened vs new findings per
   tier, and that re-running `/design-review` after the addendum
   is marked will enter Stage 2 for that round.

   End with: "Addendum N written into
   `docs/design/design-review-checkpoint-NNN.md`. Mark up its
   findings (one decision per AUDIT NOTE block), then re-run
   `/design-review` to walk Addendum N's dispositions."

---

# Stage 2 — Walk dispositions; land or open another round

Steps S2.1 through S2.8. Stage 2 runs when the newest checkpoint
is `AWAITING-DECISIONS`, every AUDIT NOTE in the latest round is
marked, and the Disposition log does NOT yet have rows for that
round.

## Step S2.1: Read the marked latest round

Re-read the latest `design-review-checkpoint-NNN.md` end-to-end.
Identify the **latest round** — the most recent
`## Addendum N — YYYY-MM-DD` section if any exists, otherwise the
original Findings section (Round 1).

Parse each AUDIT NOTE block in the latest round. Classify each as
one of:

- **Accepted** — apply the recommendation as written when the doc
  lands.
- **Accepted with caveats** — apply the recommendation, shaped by
  the caveats. The caveats are the load-bearing part; read them
  carefully.
- **Defer Approved** — do not apply; the finding becomes a future
  consideration recorded only in the audit trail.
- **DECISION: <choice>** — apply the explicit choice when the doc
  lands. May differ from the original Recommendation.
- **REJECTED: <reframing prose>** — none of the listed
  recommendations was satisfactory. Record `REJECTED` verbatim as
  the disposition; the rejection prose is the authoritative input
  the next addendum reframes the finding from. REJECTED applies no
  doc revision and is an automatic open-another-round trigger
  (see S2.5).

If any AUDIT NOTE line is malformed (not one of the five shapes),
surface it and ask Jamie to clarify. Do not guess.

A REJECTED marking always triggers Addendum N+1 (S2.5). A marking
in any of the other four shapes can also carry a further-
investigation directive (examples: "Accepted, but verify X
first", "Defer — revisit after Y resolves") — note any such
directive for the land-or-addendum decision in S2.5. In every
case the marking IS a disposition for the Disposition log this
round; REJECTED or the directive is what opens the next addendum.

Re-state to Jamie: round name (Original / Addendum N), per-tier
counts, and a brief summary of which markings closed cleanly vs
which asked for further investigation. Confirm before walking.

## Step S2.2: Triage the latest round's markings

Jamie's inline markings are settled decisions — her act of writing
each one IS the confirmation. Do NOT paraphrase-and-confirm each
marking back; that round-trip is busywork this step exists to
avoid. Scan the whole round, then surface only what is genuinely
unresolved.

1. **Scan every marking in the latest round** (B/R/N order) and
   classify each per S2.1: Accepted / Accepted with caveats /
   Defer Approved / DECISION / REJECTED. For "Accepted with
   caveats" and "DECISION", capture the caveats / chosen path so
   the landing application later has unambiguous instructions.
2. **Sort into unambiguous vs genuinely ambiguous.** A marking is
   *genuinely ambiguous* only when one of these holds:
   - **Malformed** — not one of the five legend shapes.
   - **Internally contradictory caveats** — the caveat text
     conflicts with itself or with the disposition keyword, or a
     caveat expands the finding's scope in a way the
     recommendation didn't anticipate.
   - **Scope conflict between findings** — two markings, applied
     as written, would collide (e.g. one edits a section another
     deletes).
   Everything else is unambiguous — including a plain REJECTED
   (record it; it drives the S2.5 open-another-round
   recommendation).
3. **Surface only the genuinely ambiguous markings**, one focused
   question each, and resolve them with Jamie before recording. Do
   not re-walk the unambiguous ones.

The classified dispositions feed S2.3, which writes all rows in a
single batch.

## Step S2.3: Append rows to the Disposition log

After the triage in S2.2, append one row per finding to the
`## Disposition log` table in a single batch. Format:

| Round | Finding | Disposition | Reason |
|-------|---------|-------------|--------|
| <Original or Addendum N> | <ID> | <Accepted / Accepted with Caveats / Deferred / Decision / REJECTED> | <one-sentence reason or "investigation directive: <text>"> |

Rows append only — never modify earlier rows. The latest-round
disposition on a re-opened finding (e.g. `B1-A2` Decision)
supersedes earlier rows on related IDs (`B1`, `B1-A1`) for the
purposes of Step S2.6's landing application; the earlier rows
remain in the log as historical record.

State the full set of new rows back to Jamie as one table, then
write them with `Edit` in a single batch. Genuinely ambiguous
markings were already resolved in S2.2, so no per-row confirmation
is needed here.

## Step S2.4: Mid-round read of the doc state

After the rows land, Stage 2 takes stock. State to Jamie:

- **Final dispositions across all rounds.** Walk the Disposition
  log and produce a per-finding shorthand:
  - For each finding root ID that has at least one row, use the
    *latest-round* disposition. (Earlier rounds for the same root
    ID are superseded.)
  - List finding IDs that closed cleanly (Accepted / Defer
    Approved / Accepted with Caveats with no further-investigation
    directive / DECISION with no further-investigation directive).
  - **Count and list the REJECTED finding IDs in the latest
    round.** Each one forces an open-another-round recommendation
    in S2.5 — foreground them so the land-or-addendum call is
    obvious.
  - List finding IDs whose latest disposition includes a
    further-investigation directive.

- **What an Addendum N+1 would re-open.** If any latest-round
  marking is REJECTED or includes a further-investigation
  directive, name the IDs. If none, say so explicitly ("All
  findings closed cleanly this round. Recommend landing.").

- **Outstanding earlier-round findings not yet in the log.** This
  shouldn't happen under normal flow (Step 0 routes to Stage 2
  only when every AUDIT NOTE in the latest round is marked), but
  if a prior round had unmarked findings that got silently
  skipped, surface them — they need attention before landing.

This is Stage 2's tee-up for the land-or-addendum decision.

## Step S2.5: Decide — land the document, or open Addendum N+1?

Ask Jamie one question explicitly:

> "Should the document land now, or open Addendum N+1?"

Default recommendation:

- **Recommend "open Addendum N+1"** if any latest-round marking is
  `REJECTED` (always an automatic trigger — the finding has no
  satisfactory disposition yet) **or** includes a further-
  investigation directive (text like "further review required",
  "investigate X", or any marking that explicitly asks for more
  research / clarification before the disposition is final).
- **Recommend "land"** otherwise: every latest-round disposition
  closed cleanly, with no REJECTED markings and no further-
  investigation directives outstanding.

State the recommendation and the reasoning. Wait for Jamie's
decision; she can override either way.

### If "open Addendum N+1"

Leave the frontmatter at `AWAITING-DECISIONS`. Apply the TODO.txt
rewrite below (the "open another round" shape). Skip directly to
S2.8. **Do not surface a commit handoff** — this is mid-iteration;
wind-down stages any pending changes at session close.

The next session is typically a research / clarification session
driven by the TODO entry. When that work concludes, Jamie re-
invokes `/design-review` and Step 0 routes to Stage 1's addendum
branch (S1.A).

TODO.txt rewrite — first entry under "What's next":

```
- Investigate <one-line summary of what the next addendum needs:
  e.g. "off-the-shelf RBAC packages (Filament Shield, etc.) per
  B2 directive; resolve B1's deferred decision on
  site_admin/site_editor write boundaries">. When ready, re-run
  /design-review to author Addendum N+1 of
  docs/design/design-review-checkpoint-NNN.md. Mark up its
  findings, then re-run again to walk dispositions and decide
  whether to land or open Addendum N+2.
```

Carryover entries stay below in priority order.

### If "land the document"

Proceed to Step S2.6. Doc revisions land in this path only.

## Step S2.6: Land the document (only on the "land" path)

Skip this step entirely on the "open Addendum N+1" path.

### S2.6.1 Plan doc revisions from final dispositions

Walk every finding root ID that appears in the Disposition log.
For each, use the **latest-round disposition** (later rounds
supersede earlier for the same root ID). For each finding whose
latest disposition is Accepted / Accepted with Caveats / DECISION:

- Which file? (`docs/REQUIREMENTS.md`, `docs/ARCHITECTURE.md`,
  `docs/PROJECT_PLAN.md`, `docs/CLAUDE_CODE_PROMPTS.md` — and only
  these; never `docs/design/` intake docs, never `rules/`, never
  `CLAUDE.md`, never `README.md`. If a finding implies a change
  in one of those files, surface as a TODO instead of editing.)
- Which section / line / requirement / phase?
- What concrete change? (Add NFR-X. Edit FR-2 to read "...".
  Insert a step into Prompt N. Add a phase exit criterion.) Apply
  caveats from the latest disposition.
- Does the finding imply a future checkpoint? If yes, plan a
  checkpoint marker insert (see S2.6.3).

Skip Defer-Approved findings — they appear only in the audit
trail (Disposition log + Sign-off Summary), not as doc edits.

State the full edit plan back to Jamie. One bullet per finding,
showing file + section + summary of change. Wait for confirmation
before any write.

### S2.6.2 Apply doc revisions

On confirmation, apply the edits one by one. For each:

- Use the `Edit` tool on the specific section. Do not rewrite
  whole files.
- Preserve surrounding structure, numbering, and cross-references.
- If a numbered requirement (FR-N, NFR-N) is being added, append
  at the end of the relevant list — do not renumber existing
  items.
- If `CLAUDE_CODE_PROMPTS.md` Prompt N's body is being revised,
  also add a line to that prompt's "revisions since this prompt
  ran" footer noting the revision and the checkpoint number that
  drove it.

After each edit, briefly confirm to Jamie what landed.

### S2.6.3 Insert future-checkpoint markers (if any)

If any finding implied a future checkpoint, add a marker to the
relevant prompt header in `CLAUDE_CODE_PROMPTS.md`. Two forms,
chosen per the finding's weight:

- **Header-block note** (default — lighter checkpoint): add a
  line at the top of the next prompt's body, before its first
  numbered scope item:

  ```
  **Before running this prompt:** this is a good place to run
  `/design-review` (recommended by checkpoint NNN, finding <ID>).
  ```

- **First-class entry** (for major transitions Jamie flagged as
  expensive-to-retrofit): insert a standalone block between two
  prompts:

  ```
  ## Design Review Checkpoint — pre-Prompt-M

  Run `/design-review` (recommended by checkpoint NNN, finding <ID>).
  Trigger: <one-line>.
  ```

Default to header-block notes. First-class entries are for
findings where skipping the future review would be expensive.

### S2.6.4 Fill the checkpoint's bottom sections

Edit the checkpoint file itself (the only file `/design-review`
fully owns):

1. **Sign-off Summary table.** Replace the empty placeholder row
   with one row per finding root ID (deduplicated across rounds).
   For each root ID, the disposition column shows the
   latest-round shorthand: `Accepted`, `Accepted with Caveats`,
   `Deferred`, or `Decision: <one-line>`. Cite the round in the
   ID column when the latest disposition is from an addendum
   (e.g. `B1 / B1-A1` to show the supersession chain).

2. **Follow-up actions landed.** Append a bulleted list of every
   doc edit and every TODO that landed during this landing pass.
   One bullet per action:

   ```
   - B1 / B1-A2: edited `docs/REQUIREMENTS.md` — added NFR-7 (tenant scope at query layer).
   - R3: edited `docs/CLAUDE_CODE_PROMPTS.md` Prompt 1 step 4 — added migration ordering note.
   - N1: deferred — recorded in this review's audit trail.
   ```

3. **Frontmatter status** — flip from `AWAITING-DECISIONS` to
   `LANDED <YYYY-MM-DD>` using today's date.

### S2.6.5 Update REVIEWS.md

Find the line for this checkpoint in `docs/design/REVIEWS.md` and
update its status from `AWAITING-DECISIONS` to
`LANDED <YYYY-MM-DD>`. If the doc closed across multiple rounds,
append the round count in parentheses:
`LANDED YYYY-MM-DD (Round 1 + N addendums)`.

### S2.6.6 TODO.txt rewrite (landing shape)

Rewrite `TODO.txt` to remove the "mark up the checkpoint" /
"investigate before next addendum" entry. The new first entry
under "What's next" depends on what gate this review unblocked.
Common shape:

```
- At the beginning of the next session, paste Prompt N from
  docs/CLAUDE_CODE_PROMPTS.md.
```

If the landing pass added TODOs that require a separate session
(e.g. a finding called for a doc edit that's outside Stage 2's
edit scope — `open-questions.md`, `design-decisions.md`, a new
project doc), those TODOs go above the prompt entry. Carryovers
preserved below.

State the rewritten `TODO.txt` back to Jamie before saving.

## Step S2.7: Wind down (only on the "land" path)

This step runs only when S2.5 picked "land the document." Skip it
on the "open Addendum N+1" path — that path is mid-iteration and
does not wind down.

Landing is a true artifact close and the session's work is
complete, so invoke `/wind-down` to wrap up. `/wind-down` owns the
commit handoff (its Step 4, including the bundled-vs-split choice)
and the doc-coherence sweep for any docs this command doesn't own.
Stage 2 landing typically modified the checkpoint file plus one or
more of REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN /
CLAUDE_CODE_PROMPTS, plus REVIEWS.md — `/wind-down` surfaces the
commit handoff covering them. Step S2.6.6 already set `TODO.txt`'s
first entry; `/wind-down`'s safety net confirms it.

## Step S2.8: Final report

Tell Jamie:

- The S2.5 decision (land or open Addendum N+1) and the reasoning.
- Files modified this Stage 2 run (one bullet per file with the
  change summary). On the polish-round path, this is just the
  checkpoint file (Disposition log rows added).
- The Sign-off Summary in shorthand (echo the table) — landing
  path only.
- What `TODO.txt`'s first entry is now.
- What gate the review unblocks — landing path only (e.g.
  "Prompt 1 is now ready to paste once the commit lands").

End with one of:

- **Landed**: "Design review checkpoint NNN landed across <round
  count> round(s). <Gate-unblock sentence>."
- **Addendum round opened**: "Latest-round dispositions recorded;
  doc stays `AWAITING-DECISIONS` for the next round. Do the
  research/clarification work next session; re-run
  `/design-review` when ready to author Addendum N+1."

---

## Failure modes

- **`/onboard` hasn't run.** Refuse and point at `/onboard`.
- **`AWAITING-DECISIONS` checkpoint with unmarked findings in the
  latest round.** Refuse, list which findings are unmarked and
  which round they're in, remind Jamie of the legend. Don't
  auto-create a new checkpoint, don't author another addendum
  until the current latest round is fully marked.
- **`AWAITING-DECISIONS` checkpoint, latest round all marked, but
  a caveat is unclear.** Surface the specific caveat in S2.2 and
  ask before recording. Don't guess.
- **AUDIT NOTE line malformed (not one of the five shapes).**
  Surface and ask. Don't try to interpret novel wording.
- **Stage 1 addendum branch fires but no marking actually asks for
  further investigation.** S1.A Step 2 asks Jamie which findings
  the addendum re-opens and what new findings to add. If the
  answer is "none — every disposition closed cleanly," refuse the
  addendum: "No findings need re-opening and no new findings to
  add. Re-run `/design-review` and pick 'land' at the
  land-or-addendum question instead." Don't author an empty
  addendum.
- **`--new` invoked while a prior `AWAITING-DECISIONS` checkpoint
  exists.** Proceed but warn: "Checkpoint NNN-1 is being left
  unfinished. Its findings will not be applied. Continue?" Wait
  for confirmation.
- **Frontmatter `status:` value is unexpected** (e.g. file
  hand-edited to `IN PROGRESS` or removed entirely). Refuse and
  ask Jamie how to recover.
- **A finding's disposition implies an edit to a file outside the
  four doc-revision targets** (e.g. `docs/design/` intake doc, a
  rules file, `CLAUDE.md`, `README.md`). Do not edit. Add a TODO
  for Jamie to make the edit in a separate session, and record it
  in the Follow-up actions section. Stage 2's edit scope is fixed
  at REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN /
  CLAUDE_CODE_PROMPTS — anything else is out of scope.
- **Newest checkpoint is `LANDED` but stage detection is unsure**
  (e.g. multiple files, or unexpected status values). Surface the
  state and ask Jamie before proceeding.
- **Disposition log has rows for the latest round but Jamie says
  she didn't run Stage 2.** Possible if the file was hand-edited
  or if Step 0's detection misread the round boundary. Surface
  the apparent state and ask Jamie how to recover.

---

## What this command does NOT do

- Does not own a status comment in `CLAUDE.md`. Per-file frontmatter
  status (`AWAITING-DECISIONS` / `LANDED`) is the source of truth.
  See `CLAUDE.md` § Load-bearing invariants.
- Does not edit `docs/design/` intake docs, `rules/*.md`,
  `CLAUDE.md`, or `README.md`. Stage 2's edit scope is exactly
  REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN /
  CLAUDE_CODE_PROMPTS plus the checkpoint file itself plus
  `docs/design/REVIEWS.md`. Anything else surfaces as a TODO.
- Does not run any git commands (rule 7).
- Does not run any tests (rule 8).
- Does not push to remotes.
- Does not auto-mark findings on Jamie's behalf. Stage 2 refuses
  if any latest-round finding is unmarked.
- Does not edit prior-round AUDIT NOTE blocks. Earlier rounds
  remain as historical record once the Disposition log records
  their dispositions.
- Does not edit prior `LANDED` checkpoint files. They are
  historical record.
- Does not re-litigate findings already finalized in earlier
  rounds of the same checkpoint. Latest-round dispositions
  supersede earlier for the same root ID; earlier rounds are not
  re-applied.
- Does not auto-land a document. Stage 2's S2.5 asks Jamie
  explicitly whether to land or open another addendum — there is
  no implicit transition to `LANDED`.
- Does not surface a commit handoff inline. Artifact-boundary
  paths — Stage 1 *initial* (new artifact created) and Stage 2
  *landing* (artifact closes) — invoke `/wind-down`, which owns the
  commit handoff. Mid-iteration paths (Stage 1 addendum branch,
  Stage 2 opening another round) do not wind down at all;
  wind-down stages pending changes when Jamie wraps the session.
- Does not bundle multiple independent recommendations under one
  AUDIT NOTE block. Composition rule (S1.4): one finding = one
  decision.
- Does not fully spec out downstream sub-options when a first-
  level decision would eliminate them. Composition rule (S1.4):
  first-level decisions primary; downstream implications named
  in 1-2 sentences only.
