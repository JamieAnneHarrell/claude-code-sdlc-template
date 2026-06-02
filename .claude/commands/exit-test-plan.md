---
description: Author and close out a manual phase-exit test plan. Stage 1 writes the plan from the phase's exit criteria; stage 2 walks the run log and trailing notes and lands dispositions across the project's docs.
---

# /exit-test-plan

`/exit-test-plan` is the **fifth** template command, and the second
recurring one (alongside `/design-review`). Where `/onboard`,
`/bootstrap`, and `/deployment-plan` each run once during
configuration, `/exit-test-plan` runs at the close of every phase
in `docs/PROJECT_PLAN.md` whose exit criteria warrant a manual
walkthrough.

It answers *did the phase actually meet its exit criteria as a
human can verify them?* and produces two artifacts in one file:
the test plan itself (Stage 1) and the review-and-dispositions
section the plan grows when Jamie has run it (Stage 2). The
reference shape is
`c:\Users\jamie\Documents\code\ds-niche-stream\docs\test-plans\phase-1-exit.md`.

This command is the second deliberate exception to
`CLAUDE.md`'s "don't add a fourth command" guideline.
The justification mirrors `/design-review`'s: a structured close-
out pattern (sections §1–§5, the five disposition values, the
three-shelf close where decisions mirror to design-decisions /
open-questions / TODO.txt) prevents drift between what a phase
shipped and what subsequent phases assume, and has already proven
its value on `ds-niche-stream`. See `CLAUDE.md`
§ Load-bearing invariants for the recurring-vs-configuration
distinction.

This command runs in the current working directory and writes files
relative to it.

**Required reading before Step 0:** read
`rules/coding-session-rules.md` and
`rules/design-philosophy-rules.md` in full. Rule 4 (no unsolicited
design decisions) and rule 8 (Jamie runs the tests) are
load-bearing — Stage 1 must not invent new project affordances
just to make a test step easier, and the plan is something Jamie
runs, not Claude.

---

## Two stages, auto-detected

`/exit-test-plan` runs in two stages. Which stage runs is detected
from the state of `docs/test-plans/phase-*-exit.md` for the
target phase:

- **Stage 1 — Author the plan.** Runs when no test plan exists for
  the target phase, or the newest is `LANDED`. Reads
  `docs/PROJECT_PLAN.md` (phase exit criteria),
  `docs/CLAUDE_CODE_PROMPTS.md` (prompt body + deviation footer),
  `docs/REQUIREMENTS.md` (FR/NFR citations for Fail signals), and
  the implementation files the exit criteria touch. Writes
  `docs/test-plans/phase-NNN-exit.md` with frontmatter status
  `AWAITING-DISPOSITIONS`. Edits no other doc (rule 4).
- **Stage 1 — Author tests (addendum branch).** Runs when the
  newest test plan is `AWAITING-DISPOSITIONS`, every run log
  (§4 plus every §6.N) is filled, and §5 has dispositions for
  each. Appends a new §6.N polish addendum with first-class test
  cases in the same shape as §3, plus its own run log and
  comments section. Frontmatter stays `AWAITING-DISPOSITIONS`.
- **Stage 2 — Land dispositions.** Runs when the newest test plan
  is `AWAITING-DISPOSITIONS` and the **newest run log** in the
  document (§4 if no addendum yet, otherwise the latest §6.N)
  has been filled (no `[PENDING]` placeholders remain) but
  doesn't yet have its dispositions in §5. Walks each TC note,
  each `[BLOCKED]` row, and each out-of-scope observation one at
  a time with Jamie, proposes a disposition, performs root-cause
  analysis where warranted, appends rows to §5.1 and sub-sections
  to §5.2, mirrors decisions to `docs/design-decisions.md` /
  `docs/open-questions.md` / `TODO.txt`, then asks Jamie whether
  to flip the frontmatter to `LANDED <date>` or open another
  polish round.

Stage detection avoids requiring Jamie to manually flip a status —
the run log content and §5 coverage together signal readiness.

### Arguments

Progressive disclosure: bare invocation handles the happy path.

- `/exit-test-plan` — infer the target phase from
  `docs/PROJECT_PLAN.md`. The target is the first phase whose work
  is code-complete but whose exit plan is absent or still
  `AWAITING-DISPOSITIONS`. Stage auto-detects per above.
- `/exit-test-plan N` — force phase N (e.g.
  `/exit-test-plan 3`). Use when phase boundaries are ambiguous or
  Jamie wants to author/close a non-current phase's plan.
- `/exit-test-plan --new N` — force Stage 1 (new plan) even if a
  prior `AWAITING-DISPOSITIONS` plan exists for that phase. Rare;
  surfaces a warning that the prior plan will be left unfinished.

---

## Step 0: Verify prerequisites and detect stage

Read `CLAUDE.md`:

- `<!-- ONBOARD-STATUS: ... -->` must be `COMPLETE <date>`. If it's
  `UNCONFIGURED`, refuse: "Exit test plans require `/onboard` to
  have run first — they read `docs/PROJECT_PLAN.md` and
  `docs/REQUIREMENTS.md`. Run `/onboard`, then come back."
- `<!-- BOOTSTRAP-STATUS: ... -->` should be `COMPLETE <date>` for
  most phases (a manual UI walkthrough usually depends on a
  working dev environment). If it's `UNCONFIGURED` or
  `INSTRUCTIONS-WRITTEN`, surface this to Jamie and ask whether
  she wants to proceed anyway — there are pre-bootstrap phases
  (e.g. a docs-only Phase 0 variant) where a test plan is still
  useful.

Determine the target phase:

- If a phase number was passed as an argument, use it.
- Otherwise, read `docs/PROJECT_PLAN.md` and identify the first
  phase whose work is code-complete but whose exit plan is absent
  or still `AWAITING-DISPOSITIONS`. If unsure, surface the
  candidate phase to Jamie and confirm before continuing.

Then list `docs/test-plans/phase-*-exit.md` (sorted
lexicographically — the zero-padded NNN convention guarantees this
sorts as numeric order through 999). Find the newest match for the
target phase. Determine stage:

- **No matches for the target phase** → Stage 1, with N = the
  phase number, zero-padded to 3 digits.
- **Newest is `LANDED <date>`** → Stage 1 (re-running a closed
  phase). If `--new` was not passed and the user didn't supply an
  explicit phase number, surface a warning: "Phase N already has a
  landed exit plan. A new plan will live alongside the prior one.
  Confirm?" If they confirm, proceed with N = the phase number
  (filename collision is impossible — only one plan per phase, and
  the prior one is `LANDED`; if Jamie truly needs a second plan
  per phase, surface and ask how to name it).
- **Newest is `AWAITING-DISPOSITIONS`**:
  - If `--new` was passed → Stage 1 (initial), leave the prior
    plan unfinished. Surface a warning that its dispositions will
    not be applied.
  - Otherwise → identify the **newest run log** in the document
    (§4 if no addendum exists yet, otherwise the latest §6.N
    addendum's run log). Then:
    - If the newest run log contains any `[PENDING]` row → refuse:
      "Phase N's test plan has unfilled run-log rows in
      <§4 or §6.N>: <list TC IDs>. This is a rule-8 walkthrough —
      Jamie runs the steps, marks the rows, then re-runs
      `/exit-test-plan` to land dispositions." Do not auto-mark
      rows on her behalf.
    - If the newest run log is filled but §5 doesn't yet have
      dispositions for it (no §5.1 row with `Round: <name>`
      matching the newest run log's round, where round is
      "Original" for §4 or "Addendum N" for §6.N) → **Stage 2**
      (write dispositions for this round).
    - If the newest run log is filled **and** §5 has its
      dispositions recorded → **Stage 1 addendum branch**
      (Jamie's invoking after a polish coding session to author
      §6.(N+1)).
- **Newest has any other frontmatter status** → unexpected state;
  surface and ask Jamie how to proceed.

The placeholder line that distinguishes "unfilled" rows is exactly
the literal string `[PENDING]` (uppercase, in square brackets) at
the end of the row, in any run log in the document (§4 or any
§6.N). Any row whose status field has been changed (to `[PASS]`,
`[FAIL]`, `[SKIP]`, `[BLOCKED]`, or any other value) counts as
filled. The detection logic above treats §4 and every §6.N as
first-class run logs of identical shape.

`[BLOCKED]` means the test could not run at all — a setup
prerequisite failed, the dev server wouldn't start, a dependency
wasn't installed, or an unexpected error blocked progress before
the Steps could be exercised. It is distinct from `[FAIL]`, which
means the test ran and produced wrong output. The tester records
the blocker itself in the run log's notes block, keyed by TC ID
(e.g. `TC-3 blocked: dev server won't start — port 5173 already
in use`). Stage 2 walks `[BLOCKED]` rows alongside per-TC notes
(see Step S2.2) using the same five disposition values.

---

# Stage 1 — Author tests

Stage 1 has two branches:

- **Initial branch** (Steps S1.1 through S1.8) — used when the
  target phase has no test plan yet, when the newest is `LANDED`,
  or when `--new` was passed. Writes the whole document
  (§1 through §4 plus an empty §5 placeholder).
- **Addendum branch** (Step S1.A) — used when an
  `AWAITING-DISPOSITIONS` plan exists, every run log is filled,
  every run log's dispositions are recorded in §5, and Jamie is
  invoking after a polish coding session. Appends a new §6.N
  section to the existing document with the same shape as
  §3 + §4: test cases, run log, comments section. Does not
  rewrite anything above the new addendum.

The branches share the input-reading discipline (S1.1) and the
TC-composition rules (S1.4, S1.6). The addendum branch skips S1.2
triage, S1.5 prep authoring (no new prep needed for an addendum),
and uses a focused variant of S1.3 questions that asks which
post-polish items the addendum verifies.

## Step S1.1: Read project context

Load these in order. Skip any that don't exist.

1. `docs/PROJECT_PLAN.md` — Phase N's exit criteria. This is the
   source of truth for what the plan must cover.
2. `docs/CLAUDE_CODE_PROMPTS.md` — Prompt N's body and its
   "revisions since this prompt ran" footer. Deviation notes
   matter — they pinpoint where the plan needs to be extra-
   explicit because what landed diverged from the original plan.
3. `docs/REQUIREMENTS.md` — keep open while drafting. Fail signals
   in §3 cite FR/NFR numbers; future-Claude grepping
   `docs/test-plans/` for an FR wants to land here.
4. `docs/ARCHITECTURE.md` — for the architectural anchors test
   cases verify (e.g. data flow, isolation boundaries).
5. The implementation files Phase N's exit criteria touch. Exit
   criteria tend to be abstract ("super-admin can create two
   sites and switch between them"); the implementation tells you
   how the UI gets there and where the failure modes live. Read
   them — not just to draft Steps, but to write good **Fail
   signals** that point at code sites, not symptoms.
6. `README.md` Developer setup section — for prep cross-references.
7. Prior `docs/test-plans/phase-*-exit.md` files. If any exist:
   - The most recent `LANDED` plan's §5.3 process note is the
     canonical pattern for §5 in this plan. Lift the pattern, not
     the content.
   - Deferred-for-retest items from prior plans (Deferred User
     Stories that the design-review session promoted into a later
     phase) should re-test now if the current phase's scope covers
     them. Note these for Step S1.3.
8. **If the addendum branch is firing**: read the current phase's
   plan (`docs/test-plans/phase-NNN-exit.md`) end-to-end. The
   §5.1 disposition table tells you which Fix-now items need
   manual re-verification; the latest §6.N (if any) tells you
   what was already verified. The addendum you're about to author
   should cover the unverified Fix-now items, any FAILs from the
   newest run log that have been fixed since, and every
   `[BLOCKED]` row from the newest run log (auto-re-run, see
   S1.A Step 2).

Do not start asking questions yet.

## Step S1.2: Reconnaissance and triage

State back to Jamie a 4–6 bullet summary covering:

- **Phase N's exit criteria** as you read them, in your own words.
  This is where Jamie catches misreads of the PROJECT_PLAN.md
  bullets.
- **Implementation summary** — which files implement the exit
  criteria, named explicitly. Jamie should recognize the list.
- **Any deviation footer** content in CLAUDE_CODE_PROMPTS.md
  Prompt N — these are the places the plan needs to be extra-
  explicit.
- **In-scope (UI walkthrough)** — the exit criteria that a human
  can verify by clicking. Bullet list.
- **Out-of-scope (covered by automated tests, not this plan)** —
  invariants that need code to detect (cryptographic round-trips,
  scope-isolation closures, anything inside a middleware that a
  user can't observe directly). Bullet list. This boundary is
  load-bearing — it tells future sessions what's covered without
  re-asking.

The in-scope / out-of-scope split is the most important judgment
in Stage 1. Get it wrong and either the plan misses real coverage
(in-scope items moved to "covered by tests") or pads itself with
items a human can't verify (out-of-scope items added to §3).

Don't proceed until Jamie confirms the split.

## Step S1.3: Targeted questions

Ask via `AskUserQuestion`, batched into 1–2 dialogs of 3–4
questions each. Tailor to the phase — a phase that ships UI
features asks different questions than a phase that ships
infrastructure with no UI surface. Skip any category not relevant.

Cover when relevant:

- **Environment-dependent test cases.** If the plan will exercise
  a feature whose verification depends on tooling whose behavior
  varies by machine (DNS resolution on Windows, hosts file edits,
  a specific browser, a hardware device), surface the dependency
  and ask whether to mark the TC optional.
- **Deferred items from a prior test plan.** If prior plans have
  Deferred User Stories that the current phase might have
  delivered, ask Jamie whether to re-test them now. The plan's
  §3 should call them out by their prior TC ID and prior plan
  filename ("Re-test TC-2b from `phase-001-exit.md`").
- **Test data setup.** Ask how Jamie wants prep recipes shaped:
  inline in §2 (rerunnable, copy-paste), or a one-line pointer to
  an existing seeder / fixture script. Default to inline if the
  project has no existing affordance.
- **Tester identity** for the §4 header row. Default to Jamie's
  name; ask only if a different tester is expected.

## Step S1.4: Compose test cases

Apply these compositional rules before writing the file:

- **One concept per TC.** "Create" and "switch" are separate
  concepts even if both verify the same code path. Splitting them
  obscures *which* step regressed when one fails.
- **Decompose multi-branch behaviors into sub-table TCs.** When
  one piece of code has N branches (e.g. a middleware with four
  resolution paths, a permission check with three role tiers),
  write one parent TC with a small table covering every branch
  and its expected outcome, plus the boundary cases. Columns:
  branch / inputs (auth, URL, payload) / expected. This reads
  cleanly and survives "I want to add one more case" without
  renumbering.
- **Three labeled blocks per TC.** Every TC has:
  - `**Steps.**` — numbered, imperative. Each step is something
    the tester clicks, types, or pastes. No prose narrative.
  - `**Expected.**` — bulleted. What should be true after the
    steps. Where the expectation comes from a spec, cite inline
    (e.g. "…no delete button or bulk-delete action (NFR-20)").
  - `**Fail signals.**` — named regression modes, each tied to a
    code site or a requirement. "Page renders with Site B context
    → tenant isolation regression" is useful; "doesn't work" is
    not. Fail signals are what makes mid-run troubleshooting
    fast — they point at the fix-site, not the symptom.
- **Environment-dependent cases marked optional.** If a TC
  depends on tooling whose behavior varies by machine (per
  S1.3), label it optional and, where possible, route the same
  code branch through a different TC that's machine-portable.
  Don't promise something the doc can't deliver on every machine.
- **Cite requirements in Fail signals.** FR-N, NFR-N, etc.
  Future-Claude grepping `docs/test-plans/` for an FR wants to
  find every place it's exercised.

## Step S1.5: Write the test plan document

Filename: `docs/test-plans/phase-NNN-exit.md` where NNN is the
zero-padded 3-digit phase number from Step 0. Pad width is fixed
at 3 so files sort lexicographically as numeric order through 999.

If `docs/test-plans/` does not exist, create it as part of writing
the file. No `.gitkeep` or other marker is needed — the directory
exists once the first plan lands.

Frontmatter status: `AWAITING-DISPOSITIONS`.

Use this template verbatim, filling in placeholders:

```markdown
---
phase: N
date: YYYY-MM-DD
status: AWAITING-DISPOSITIONS
---

# Phase N exit — manual <stack> test plan

A scripted walkthrough that exercises Phase N's exit criteria
(`docs/PROJECT_PLAN.md` Phase N) against the running <surface>.
Jamie runs each section top-down; Claude does not run these — per
coding-session rule 8.

Pass criteria for closing Phase N: every **Expected** in §3 holds.
Anything red gets reported back; the next session is interactive
troubleshooting.

---

## 1. What this covers

In scope (<surface> walkthrough):

- <bullet — UI-observable behavior tied to an exit criterion>
- <bullet>

Out of scope (covered by the automated test suite, not this plan):

- <bullet — code-internal invariant; name the test file or suite
  if known>
- <bullet>

---

## 2. Preparation

### 2.1 <prep step name>

<Setup instruction. If a command, fenced. If a tinker/REPL/seed
recipe, fenced inside language hint. Use `firstOrCreate` or
equivalent rerunnable pattern so a re-run mid-walkthrough doesn't
explode.>

**Expected:** <what should be true after this prep step>.

### 2.2 <prep step name>
...

---

## 3. Test cases

Each case is **Steps → Expected → Fail signals** (Fail signals
optional when the Expected outcome is unambiguous). Mark pass/fail
in §4 at the bottom of this file when you run through.

### TC-1 — <one-line title>

**Steps.**

1. <click / type / paste>
2. <click / type / paste>

**Expected.**

- <verifiable state>
- <verifiable state, with inline citation: (NFR-20), (FR-2), etc.>

**Fail signals.** <named regression mode> → <code site or
requirement>. <named regression mode> → <code site>.

### TC-2 — <title>
...

### TC-N — <multi-branch case with sub-table>

<short intro paragraph if needed>

| # | Branch | Auth as | URL / input | Expected |
|---|---|---|---|---|
| Na | <branch description> | <auth state> | <input> | <expected outcome> |
| Nb | <branch> | <auth> | <input> | <expected> |
| ... | | | | |

**Fail signals.** <branch ID> returns <wrong outcome> → <code
site or requirement>.

---

## 4. Run log

Copy this block into the next session's chat when reporting back.
One word in the result column is fine: `pass`, `fail`, `skip`,
`blocked`. Until then, every row reads `[PENDING]` — this is the
literal sentinel `/exit-test-plan` Stage 2 detection looks for.

Use `blocked` when the test could not run at all (setup
prerequisite failed, dev server wouldn't start, etc.). Use `fail`
when the test ran and produced wrong output. Don't conflate them.

```
TESTER: <name>  <date>
TC-1   <short name> ............................. [PENDING]
TC-2   <short name> ............................. [PENDING]
...
```

Notes / surprises:

```
<free-form. The tester writes inline observations here while
running — UX feedback, hypotheses about root cause, "while I was
in here I noticed…" remarks. Stage 2 reads this verbatim, in
whatever shape the tester wrote it.

For any [BLOCKED] row above, include a one-sentence description
of the blocker keyed by TC ID (e.g. `TC-3 blocked: dev server
won't start — port 5173 already in use`). Stage 2 needs the
blocker description to disposition the row.>
```

---

## 5. Review and dispositions

*Stage 2 of `/exit-test-plan` appends this section after Jamie
has run the plan and filled §4. Empty until then.*
```

A few rules for filling the template:

- **Replace `<stack>` and `<surface>` with the project's actual
  framing.** "UI" for a web app, "CLI" for a command-line tool,
  "REPL" for a library exercise, etc.
- **§1 bullets reference Phase N's exit criteria by their text
  in PROJECT_PLAN.md.** Don't paraphrase aggressively — Jamie
  should recognize each bullet as one of the phase's
  deliverables or as a derived check.
- **§2 prep uses only existing project affordances.** Stage 1
  must not invent new CLI commands, seeders, fixture helpers, or
  test-mode routes to make a TC easier (rule 4). If the phase
  shipped no affordance for some setup step, use the REPL / a
  manual recipe / a tinker block / equivalent, and note that the
  absence of an affordance is intentional.
- **§3 multi-branch sub-tables are TC-N with sub-rows Na, Nb, Nc…**
  Match `phase-001-exit.md` TC-8 from the ds-niche-stream
  reference (which covered 9 sub-rows for a middleware with four
  resolution paths). The §4 run log lists each sub-row on its
  own line so the tester marks them individually.
- **§4 short names match what's after the TC ID in §3** —
  consistent across the two surfaces so a `[FAIL]` on row 7 is
  obvious which TC it refers to.

If §3 has zero out-of-scope items, write the §1 "Out of scope"
header anyway and a single italic line: `*Every exit criterion in
this phase has a manual check below — no automated-test coverage
is being relied on.*` The shape stays consistent across plans.

## Step S1.6: Update TODO.txt

`TODO.txt` is gitignored and personal. Rewrite it (do not append)
so the first entry under "What's next" is the test-plan follow-up:

```
- At the beginning of the next session, run
  docs/test-plans/phase-NNN-exit.md top-down. Work §2 prep, then
  the §3 test cases. Mark §4's run log inline as you go and
  capture inline notes for surprises. If any TC fails, the same
  session is interactive troubleshooting (Fail signals point at
  the fix-site). After every TC is run, re-run /exit-test-plan
  to land dispositions.
```

Carryover entries stay below in priority order. Do not invent
carryovers; preserve whatever was already there.

## Step S1.7: Wind down

Stage 1 initial created a new test plan and the session's work is
complete — the plan now awaits Jamie's run, which is her
next-session task. This is a true artifact boundary, so invoke
`/wind-down` to wrap up the session. `/wind-down` owns the commit
handoff (its Step 4) and the doc-coherence sweep for any docs this
command doesn't own; the only file `/exit-test-plan` owns is the
test plan itself. Step S1.6 already set `TODO.txt`'s first entry —
confirm `/wind-down`'s safety net preserves it.

(The Stage 1 addendum branch runs S1.A instead and does NOT wind
down — appending a §6.N addendum is mid-iteration.)

## Step S1.8: Final report

Tell Jamie:

- Path to the new test plan file.
- TC count (and sub-row count if any TC uses a sub-table).
- A 1-sentence reminder that the plan is rule-8 work — Jamie
  runs it, Claude doesn't.
- That re-running `/exit-test-plan` after §4 is filled will enter
  Stage 2 and apply the implied dispositions.

End with: "Phase N exit test plan written. Run it top-down; when
§4 is filled, re-run `/exit-test-plan` to land dispositions."

---

## Step S1.A: Author addendum N (addendum branch)

Runs instead of S1.2–S1.8 when Step 0 routed to the addendum
branch. The plan file already exists; this step appends a new
§6.N section to it without rewriting anything above. Mid-
iteration work — no commit handoff. Wind-down handles staging
when Jamie wraps the session.

1. **Determine the next addendum number N.** Count existing §6.M
   sub-sections in the file; N is the next integer (1 if §6 does
   not exist yet).

2. **Identify what this addendum verifies.** Walk §5.1's
   disposition table and the newest run log together:
   - Every Fix-now row whose verification is not already a
     PASS in some later run log needs a TC in this addendum.
   - Every FAIL in the newest run log whose underlying defect was
     fixed during the polish coding session needs a TC verifying
     the fix. (Ask Jamie which FAILs were fixed; do not assume.)
   - **Every BLOCKED row in the newest run log gets a TC in
     this addendum automatically — never drop a BLOCKED test.**
     The TC is the same test from §3 (or the prior addendum
     where it last ran); revise the Steps inline if §5.1's
     unblocking work changed prereqs or affordances. No
     cross-reference cruft in the title — the prior run log
     and §5.1 row are already the record.
   - SKIP rows usually do not need re-verification unless the
     skip condition has changed (Jamie confirms case by case).

   State this list back to Jamie as "Addendum N will verify:
   <bullet list>." Wait for confirmation, additions, or
   removals.

3. **Compose TCs in the same three-block shape as §3** (Steps,
   Expected, Fail signals — see S1.6). Number them
   `TC-AN.1`, `TC-AN.2`, … where N is the addendum number.
   Title conventions:
   - **Fix-now verifications and FAIL re-tests**: title
     references the §5.1 row or original-§4 TC being verified
     (e.g. `TC-A1.1 — verify §5.1 TC-2a fix: timezone selector
     replaced free-text`). The linkage is what lets Stage 2 know
     which §5.1 row this addendum closes.
   - **BLOCKED re-runs**: title is the original test's title as-is
     (e.g. `TC-A1.2 — Site creation and switching`). No
     cross-reference prefix — the prior run log and §5.1 row are
     already the record of what was blocked and why.

4. **Append §6.N to the file** using this template:

   ```markdown

   ---

   ## 6.N Addendum N — YYYY-MM-DD

   <one-sentence context: what this addendum verifies.>

   ### Test cases

   #### TC-AN.1 — <title>

   **Steps.** ...
   **Expected.** ...
   **Fail signals.** ...

   #### TC-AN.2 — <title>
   ...

   ### Run log

   ```
   TESTER: <name>  <date>
   TC-AN.1   <short name> ............................. [PENDING]
   TC-AN.2   <short name> ............................. [PENDING]
   ...
   ```

   ### Comments / surprises

   <free-form. Same role as §4's notes block — the tester writes
   inline observations here while running. Stage 2 reads this
   verbatim alongside the addendum run log.

   For any [BLOCKED] row above, include a one-sentence
   description of the blocker keyed by TC ID. Stage 2 needs the
   blocker description to disposition the row.>
   ```

5. **Keep frontmatter at `AWAITING-DISPOSITIONS`.** The status
   doesn't change — the addendum needs to be run before the doc
   can be landed.

6. **Update `TODO.txt`** so the first entry under "What's next"
   becomes:

   ```
   - At the beginning of the next session, run §6.N of
     docs/test-plans/phase-NNN-exit.md top-down. Mark §6.N's run
     log inline and capture comments / surprises. After every TC
     is run, re-run /exit-test-plan to land addendum N (or open
     addendum N+1 if more polish surfaces).
   ```

   Carryover entries stay below in priority order.

7. **Final report.** Tell Jamie: path to the plan, the addendum
   number, addendum TC count, which §5.1 rows each TC verifies,
   the rule-8 reminder, and that re-running `/exit-test-plan`
   after §6.N is filled will enter Stage 2 for the addendum.

   End with: "Addendum N written into
   `docs/test-plans/phase-NNN-exit.md` §6.N. Run it top-down;
   when §6.N is filled, re-run `/exit-test-plan` to land it."

---

# Stage 2 — Land dispositions

Steps S2.1 through S2.8. This stage runs when the newest test
plan for the target phase is `AWAITING-DISPOSITIONS`, its newest
run log (§4 or the latest §6.N) has no `[PENDING]` rows, and §5
does not yet have dispositions covering that newest run log.

## Step S2.1: Read the marked run log and trailing notes

Re-read the latest `docs/test-plans/phase-NNN-exit.md` end-to-end.
Identify the **newest run log** — §4 if no addendum exists yet,
otherwise the latest §6.N. That's the run log Stage 2 is
dispositioning this invocation. Older run logs have already been
dispositioned in earlier §5 entries; don't re-litigate them.

In the newest run log, in particular:

- The run-log table — each row's result (`pass` / `fail` /
  `skip` / `blocked`).
- The "Notes / surprises" block (for §4) or "Comments / surprises"
  block (for §6.N) — a single free-form surface. Observations
  come in whatever shape the tester wrote them. Don't expect
  structure. Don't reformat.

Re-state to Jamie what you found, in three short bullets:

- The pass/fail/blocked summary for this run (e.g. "20/24 pass,
  2 fail, 1 blocked, 1 skip" for §4; or "3/3 pass" for an
  addendum).
- A count of inline notes (per-TC notes + out-of-scope
  observations at the bottom of the notes block) and a count of
  `[BLOCKED]` rows.
- Any obvious clusters — multiple TCs reporting the same symptom,
  multiple notes pointing at the same code site, multiple
  blockers traceable to the same prereq.

Confirm with Jamie that this is the right read before walking
findings. If she points out something you missed in the notes,
incorporate it before continuing.

## Step S2.2: Triage the per-TC notes and BLOCKED rows

Read the whole notes block first and draft a disposition for every
item, then surface only what genuinely needs discussion. Do NOT
propose-and-confirm each note one at a time — that round-trip is
the friction this step exists to avoid.

Unlike `/design-review` (where Jamie pre-writes each decision),
Claude *proposes* the dispositions here. So the batch table is the
confirmation surface: present all proposed dispositions at once and
let Jamie confirm or adjust them in a single pass. Don't record
before she signs off on the table.

### Per-TC notes

1. **Draft a disposition for every per-TC note** in the newest run
   log (TC order), using the five values:
   - **Fix-now** — small, in-scope of the phase's exit contract.
     Bundles into the phase polish batch as the next session's
     first item.
   - **Defer to Deferred User Stories** — feature work that isn't
     part of the phase's contract, recorded for later pickup.
   - **Defer to Known Limitations** — accepted-as-is constraint
     of the current stack / approach.
   - **Spec gap** — the observation surfaces a question
     REQUIREMENTS / ARCHITECTURE doesn't answer. Queues for the
     next `/design-review` session as an agenda item.
   - **Note-only** — no action; documenting the observation is
     enough.
2. **Present all drafted dispositions as one table** — TC /
   abbreviated observation / proposed disposition / reason — for
   Jamie to confirm or adjust in a single pass.
3. **Pull out individually only the notes that genuinely need it:**
   a note whose surface hypothesis warrants real investigation, or
   an observation too unclear to disposition from the run log
   alone. Do that work, discuss, then fold the result back into
   the batch.

Where a note's surface hypothesis warrants real investigation —
the tester wrote down what they observed but the cause might be
elsewhere — perform root-cause analysis: read the relevant source
files, trace branches, separate symptom from real defect. The
canonical example is the ds-niche-stream Phase 1 O-5 finding,
where a "slug resolved as subdomain" symptom was actually a
fail-open in a different branch of the same middleware. Preserve
the full investigation (Investigation / Root cause / Implication
/ Decision) in §5.2 so future-Claude can re-read the reasoning,
not just the conclusion.

**Special case for addendum run logs.** A FAIL on an addendum TC
that verifies a known Fix-now item from §5.1 does **not** need a
new disposition — the underlying Fix-now is still queued, the
polish just didn't take this round, and the next polish session
will address it. Move on without proposing a new §5.1 row. Same
pattern for a `[BLOCKED]` row on an addendum re-run of a
previously-blocked test — the original §5.1 BLOCKED row's
unblocking path (Fix-now / Spec gap / Defer) is still active;
don't write a duplicate §5.1 row. (A **new** `[BLOCKED]` —
a test that was never blocked before becoming blocked in the
addendum — DOES get walked as above and gets a new §5.1 row.)
Notes that describe **new** observations (something the tester
saw during the addendum run that wasn't in any prior §5.1 row)
DO get walked as above and get a new §5.1 row with disposition.

### BLOCKED rows

A `[BLOCKED]` row is definitive: the test couldn't run, so work is
needed to unblock it and a new addendum will re-run it regardless
of anything decided here. Don't discuss BLOCKED rows one at a
time — record them in the batch and move on.

1. For each `[BLOCKED]` row in the newest run log, read the
   tester's blocker description and restate it in one sentence —
   what couldn't run, and what stopped it. If no blocker
   description was recorded, refuse to disposition that row until
   Jamie supplies one.
2. Draft the path to unblock as one of the same five values:
   - **Fix-now** when the blocker is a small in-scope cleanup
     coding pass (the typical case).
   - **Spec gap** when the blocker surfaces a question
     REQUIREMENTS / ARCHITECTURE doesn't answer — queues for the
     next `/design-review` checkpoint, which may decide to
     author a new PROJECT_PLAN.md phase to do the work.
   - **Defer to Deferred User Stories** when the blocker depends
     on a feature the current phase didn't ship; the test will
     be SKIPped in the next addendum.
   - **Defer to Known Limitations** when the test simply isn't
     runnable on the current stack and the blocker is accepted
     as a constraint; the test will be SKIPped in the next
     addendum.
   - **Note-only** is rare for blockers — typically a blocker
     warrants at least one of the four above.
3. **Add the BLOCKED rows to the same batch table** from the
   per-TC walk, each with its path to unblock (the §5.1 row's
   Reason column gets prefixed `Blocker:` at composition in S2.4).
   They're recorded, not debated — the blocked state itself
   mandates the next addendum. Surface a BLOCKED row individually
   only when its path to unblock genuinely needs root-cause
   analysis to determine; do that analysis as above and preserve
   it in §5.2.

Every `[BLOCKED]` row in the newest run log results in a TC re-run
in the next addendum (S1.A Step 2 handles that automatically) —
the disposition governs what unblocking work happens between
addendums, not whether the test re-runs.

## Step S2.3: Walk each out-of-scope observation

At the bottom of the newest run log's notes block (§4 for the
original run, the "Comments / surprises" block at the bottom of
§6.N for an addendum run), the tester often records observations
from playing with the system outside the scripted TCs ("while I
was in here I noticed…"). Each is typically a larger thread than
a per-TC note.

For each:

1. State it back, propose a disposition (same five values).
2. Where the surface hypothesis warrants real investigation, do
   root-cause analysis as in S2.2. Out-of-scope observations are
   where most of the heavy investigations land — they tend to be
   the discoveries that change architectural understanding.
3. Wait for Jamie to settle the disposition before moving on.

Assign each out-of-scope observation an ID. IDs are global to the
document, not per-round — count existing O-N entries in §5.2 and
continue from the next integer. So if the original run produced
O-1 through O-5 and addendum 1 surfaces two new observations,
they become O-6 and O-7.

## Step S2.4: Compose §5

§5 is a single growing section across all rounds. On the first
Stage 2 run (original-run dispositions) you compose §5 from
scratch with the three sub-sections below. On any later Stage 2
run (addendum dispositions) you **append** rows to §5.1 and
sub-sections to §5.2 — do not rewrite earlier content.

**§5.1 TC findings dispositions** — a single table that grows
across rounds. Columns:
`Round | Note | Observation (abbreviated) | Disposition | Reason`.
- `Round` is `Original` for findings from §4, or `Addendum N`
  for findings from §6.N.
- One row per per-TC note from S2.2 (only those that warranted
  a new disposition — see the addendum special case in S2.2),
  plus one row per `[BLOCKED]` row from S2.2's BLOCKED walk.
- Abbreviate observations to one short sentence; the full text
  stays in the run log's notes block. For BLOCKED-driven rows,
  prefix the abbreviated observation with `Blocker:` and put the
  path to unblock (Fix-now / Spec gap / Defer to …) in the
  Reason column.
- Reason column is a short justification for the disposition.

**§5.2 Out-of-scope observations** — one sub-section per O-ID,
grows across rounds:

```markdown
#### O-N — <one-line title>

*From.* <Original §4 run | §6.M addendum M>, YYYY-MM-DD.

*Observation.* <one short paragraph restating the observation in
Claude's words, with Jamie's clarifications.>

*Disposition.* <Fix-now / Defer to Deferred User Stories / Defer
to Known Limitations / Spec gap / Note-only> with one-sentence
reason.
```

When the disposition involved real investigation, also include
(in this order, between *Observation* and *Disposition*):

```markdown
*Investigation.* <what was read; which files; which branches were
traced>

*Root cause.* <the actual cause separated from the surface
symptom>

*Implication.* <what this means for the system — security,
correctness, maintenance>

*Decision.* <what to do about it, in spec terms (not yet in code
terms — code lands in a later session)>
```

Where a decision overlaps with a previously-deferred decision (in
`open-questions.md` or `design-decisions.md`), include a
*Conflict with prior decision.* paragraph that names the prior
decision, explains how the new clarification reconciles, and
notes whether the prior entry drops out.

**§5.3 Process note** — written on the *first* test plan for the
project; pins the §5 pattern. Written once on the original-run
Stage 2; subsequent rounds do not modify it. Later plans link
back to it rather than restating.

For the project's first plan, write:

```markdown
### 5.3 Process note for future test-plan reviews

This §5 is the working pattern going forward: any manual test
plan that produces non-trivial findings gets a "Review and
dispositions" section appended after the §4 run log.
Architectural decisions discovered during testing mirror to
`docs/design-decisions.md` as concise canonical entries with a
pointer back here. Spec-gap items queue in `TODO.txt` as
`/design-review` agenda items, then promote to
`docs/REQUIREMENTS.md` once that review decides.

Polish-batch fixes for §5.1 Fix-now items get verified by either
the automated test suite (most cases) or a §6.N polish addendum
(for UI-only fixes the test suite can't reach). Addendums are
authored by re-invoking `/exit-test-plan` after a polish coding
session lands the fixes — Stage 1's addendum branch appends a
new §6.N with first-class TCs in the same shape as §3, plus its
own run log and comments block. Stage 2 runs again to read each
addendum's results and either land the document or open another
polish round. The document isn't `LANDED` until every run log
(§4 plus every §6.N) is PASS / SKIP — no `[BLOCKED]` or `[FAIL]`
rows in the newest run log — and §5 has no outstanding Fix-now
items left unverified.
```

For later plans, write:

```markdown
### 5.3 Process note

See `docs/test-plans/phase-001-exit.md` § 5.3 for the §5 and §6
authoring patterns.
```

(Replace `phase-001-exit.md` with whichever earlier plan first
defined the pattern.)

State the composed §5 back to Jamie before writing it to the
file. Don't write until she confirms. On addendum-round Stage 2,
state only the appended rows and sub-sections.

## Step S2.5: Apply mirrors

Apply doc edits one by one. For each: state the edit, wait for
Jamie's confirmation, then apply. Edits land in **three** places.

### `docs/design-decisions.md`

For each S2.3 finding whose Decision changes architectural
understanding (not just polish, not just deferred work), append
a concise canonical entry. Format follows the project's existing
design-decisions.md pattern (typically **Decision** / **Why** /
**Why not (alternatives)** — read the file's header for the
exact shape if unsure):

```markdown
### <one-line decision title> (YYYY-MM-DD, Phase N exit review)

**Decision.** <one short paragraph>

**Why.** <one short paragraph or 2–3 bullets>

**Why not (alternatives).** <one short paragraph or 2–3 bullets>

See `docs/test-plans/phase-NNN-exit.md` §5.2 O-N for the full
investigation.
```

The link back to §5.2 is load-bearing — design-decisions.md is
the canonical decision log, but the *reasoning* lives in the
test plan. Don't duplicate the analysis; link to it.

### `docs/open-questions.md`

For each Defer (Deferred User Stories) and Defer (Known
Limitations) disposition, append to the existing section header
in `docs/open-questions.md`. One bullet per item:

```markdown
- <one-sentence description of the deferred item>. From
  `docs/test-plans/phase-NNN-exit.md` <TC ID or O-N>.
```

If the review **resolved** a previously deferred item (a Defer
entry in open-questions.md that this phase's work made concrete),
remove it. Surface the removal to Jamie before applying.

### `TODO.txt`

Rewrite `TODO.txt` to reflect the new state. The first entry
depends on what comes next (and Step S2.6 settles that), so
defer the rewrite until after S2.6 picks land-or-addendum. The
shapes are:

- **If S2.6 picks "open the next polish round"** (the typical
  case when §5.1 has any new Fix-now items, or the newest run
  log had any FAIL or `[BLOCKED]`):

  ```
  - At the beginning of the next session, work the Phase N polish
    batch from docs/test-plans/phase-NNN-exit.md §5.1 (latest
    Fix-now rows + any FAILs from the newest run log + any
    BLOCKED unblocking work). Plan a single bundled PR if scope
    allows; split if the fixes touch unrelated concerns. When
    polish lands, re-run /exit-test-plan to author the next §6.N
    addendum verifying the fixes and re-running any blocked tests.
  ```

- **If S2.6 picks "land the document"** (no outstanding Fix-now
  items, no FAILs, no `[BLOCKED]` rows in the newest run log,
  every run log is PASS / SKIP):

  ```
  - At the beginning of the next session, <gate-unblock — e.g.
    paste Prompt N+1 from docs/CLAUDE_CODE_PROMPTS.md, or run
    /design-review for Phase N+1>.
  ```

- **Spec-gap items become design-review agenda items.** Add a
  second entry directly under the first:

  ```
  - Run /design-review with trigger "Phase N exit review spec
    gaps". Findings to surface: <one-line per spec-gap item,
    citing TC ID or O-N>.
  ```

  If no Spec-gap items, omit this entry. Spec-gap items queue
  regardless of whether the doc is landing or opening another
  polish round — they're orthogonal to the polish lifecycle.

- **Previously-deferred decisions that this review resolves drop
  out.** If `TODO.txt` had a "Deferred decisions" section entry
  that §5.2's findings settled, remove it. Surface the removal.

- **Carryover entries preserved below** in priority order.

State the rewritten `TODO.txt` back to Jamie before saving.

### Do NOT edit

Stage 2 does not touch:

- `docs/REQUIREMENTS.md`, `docs/ARCHITECTURE.md`,
  `docs/PROJECT_PLAN.md`, `docs/CLAUDE_CODE_PROMPTS.md`. These
  are `/design-review` Stage 2's territory. Spec-gap items
  promote to these docs only after a `/design-review` session
  decides. Surface as a TODO instead (handled by the Spec-gap
  TODO entry above).
- `rules/*.md`, `CLAUDE.md`, `README.md`. Anything implied here
  surfaces as a TODO; the test plan does not edit them
  directly.

If a finding seems to imply an edit to one of these files,
surface it as an additional `TODO.txt` line under the polish
batch.

## Step S2.6: Decide — land the document, or open another polish round?

After §5 is composed and mirrors are queued, ask Jamie one
question explicitly:

> "Should the document land now, or stay open for another polish
> round?"

Default recommendation:

- **Recommend "open another round"** if any of the following are
  true:
  - §5.1 contains any Fix-now row whose verification hasn't been
    PASS in some run log.
  - The newest run log has any FAIL.
  - **The newest run log has any `[BLOCKED]` row.** BLOCKED is a
    final disposition for that run log, but the test plan does
    not close while any test remains blocked — the next addendum
    re-runs it (see S1.A Step 2).
  - Any newly-added §5.1 row from this Stage 2 run is Fix-now.
- **Recommend "land"** otherwise: every run log is PASS / SKIP,
  every Fix-now in §5.1 has a passed verification (either via
  the automated test suite or a passed addendum TC), no
  `[BLOCKED]` rows in the newest run log, no new Fix-now items
  surfaced this round.

State the recommendation and the reasoning. Wait for Jamie's
decision; she can override.

### If "land the document"

Edit the test plan file's frontmatter:

```markdown
---
phase: N
date: YYYY-MM-DD          # original creation date — do not change
status: LANDED YYYY-MM-DD  # today's date
---
```

Preserve the `date:` field. Update only `status:`.

Now apply S2.5's TODO.txt rewrite (the "land" shape) and proceed
to S2.7 (wind down — landing is a meaningful artifact close, so
the session wrap-up is appropriate here).

### If "open another polish round"

Leave frontmatter as `AWAITING-DISPOSITIONS`. Apply S2.5's
TODO.txt rewrite (the "polish round" shape). **Do not surface a
commit handoff** — this is mid-iteration; wind-down stages any
pending changes at session close. Skip directly to S2.8.

The next session will be a polish coding session driven by the
TODO entry. When polish lands, Jamie re-invokes
`/exit-test-plan` and Step 0 routes to Stage 1's addendum branch
(S1.A).

## Step S2.7: Wind down (only when landing)

This step runs only when S2.6 picked "land the document." Skip it
on the polish-round path — that path is mid-iteration and does not
wind down.

Landing is a true artifact close and the session's work is
complete, so invoke `/wind-down` to wrap up. `/wind-down` owns the
commit handoff (its Step 4, including the bundled-vs-split choice)
and the doc-coherence sweep for any docs this command doesn't own.
Stage 2 landing typically modified the test plan file plus
`docs/design-decisions.md` and `docs/open-questions.md` —
`/wind-down` surfaces the commit handoff covering them. S2.5's
`TODO.txt` rewrite already set the first entry; `/wind-down`'s
safety net confirms it.

## Step S2.8: Final report

Tell Jamie:

- Files modified this Stage 2 run (one bullet per file with the
  change summary).
- New disposition counts added this round (e.g. "Added 1 Fix-now,
  1 Defer to Known Limitations to §5.1; 1 new O-N in §5.2").
- The S2.6 decision (land or another round) and the reasoning.
- What `TODO.txt`'s first entry is now.
- Whether a `/design-review` session is queued (Spec-gap items
  surface this).
- Whether any out-of-scope observations triggered architectural
  decisions in `design-decisions.md`.

End with one of:

- **Landed**: "Phase N exit review landed. <Gate-unblock sentence
  — e.g. 'Phase N is closed; Phase N+1 prompt is ready to paste,'
  or 'Phase N+1 prompt is ready, but run /design-review first if
  spec gaps surfaced.'>"
- **Polish round opened**: "Phase N exit dispositions recorded;
  doc stays `AWAITING-DISPOSITIONS` for the next polish round.
  Work the polish batch next session; re-run `/exit-test-plan`
  when polish lands to author the next §6.N addendum."

---

## Failure modes

- **`/onboard` hasn't run.** Refuse and point at `/onboard`.
- **No `docs/PROJECT_PLAN.md`.** Refuse: "`/exit-test-plan` reads
  Phase N's exit criteria from `docs/PROJECT_PLAN.md`. That file
  doesn't exist. Run `/onboard`, then come back."
- **Target phase number not found in `docs/PROJECT_PLAN.md`.**
  Surface the phase list and ask Jamie which to use.
- **`AWAITING-DISPOSITIONS` plan with `[PENDING]` rows in any run
  log (§4 or any §6.N).** Refuse, list which rows are pending and
  which run log they're in, remind Jamie this is a rule-8
  walkthrough she runs. Don't auto-create a new plan, don't mark
  rows on her behalf, don't author another addendum until the
  current one's run log is filled.
- **`AWAITING-DISPOSITIONS` plan, every run log filled, but a
  note is ambiguous.** Surface the specific note in S2.2 or S2.3
  and ask before planning the disposition. Don't guess at intent.
- **`[BLOCKED]` row in the newest run log with no blocker
  description in the notes block.** Refuse to disposition that
  row: "TC-N is marked `[BLOCKED]` but no blocker description is
  recorded in the notes block. Add a one-sentence description
  keyed by TC ID, then re-run `/exit-test-plan`." Stage 2 needs
  the blocker to choose between Fix-now / Spec gap / Defer paths.
- **Addendum branch fires but Jamie hasn't actually landed a
  polish session yet, and no BLOCKED rows need re-running.**
  S1.A Step 2 asks Jamie which Fix-now items the addendum
  verifies. If the answer is "none — polish isn't done yet" AND
  the newest run log has no `[BLOCKED]` rows to re-run, refuse
  the addendum: "No Fix-now items are ready for verification and
  no BLOCKED tests are queued. Run the polish coding session
  first, then re-invoke `/exit-test-plan`." Don't author an empty
  addendum. (A BLOCKED-only addendum — purely re-running
  previously-blocked tests after unblocking work landed — is
  valid and proceeds normally.)
- **A finding's disposition implies an edit to `REQUIREMENTS.md`
  /  `ARCHITECTURE.md` / `PROJECT_PLAN.md` /
  `CLAUDE_CODE_PROMPTS.md` / `rules/*.md` / `CLAUDE.md` /
  `README.md`.** Do not edit. Add a TODO under the polish batch
  for Jamie to make the edit in a separate session (typically a
  `/design-review` if it's a spec gap; a follow-up coding session
  if it's a rules / CLAUDE / README change). Stage 2's edit scope
  is fixed at the test plan file + `design-decisions.md` +
  `open-questions.md` + `TODO.txt` — anything else is out of
  scope.
- **`--new` invoked while a prior `AWAITING-DISPOSITIONS` plan
  exists for the same phase.** Proceed but warn: "Phase N's prior
  plan is being left unfinished. Its dispositions will not be
  applied. Continue?" Wait for confirmation.
- **Frontmatter `status:` value is unexpected** (file
  hand-edited to a non-standard value or removed entirely).
  Refuse and ask Jamie how to recover.
- **Multiple `AWAITING-DISPOSITIONS` plans for the same phase**
  (shouldn't happen under normal flow, but possible if `--new`
  was used). Surface the state and ask Jamie which to land.

---

## What this command does NOT do

- Does not own a status comment in `CLAUDE.md`. Per-file
  frontmatter status (`AWAITING-DISPOSITIONS` / `LANDED`) is the
  source of truth, mirroring `/design-review`. See
  `CLAUDE.md` § Load-bearing invariants.
- Does not run the test plan. The plan is rule-8 work — Jamie
  runs it, Claude doesn't. Even if Jamie asks for help running a
  specific TC, the answer is troubleshooting (read the Fail
  signals, investigate the symptom), not "run TC-3 for you."
- Does not edit `docs/REQUIREMENTS.md`, `docs/ARCHITECTURE.md`,
  `docs/PROJECT_PLAN.md`, `docs/CLAUDE_CODE_PROMPTS.md`,
  `rules/*.md`, `CLAUDE.md`, or `README.md`. Stage 2's edit scope
  is exactly the test plan file + `docs/design-decisions.md` +
  `docs/open-questions.md` + `TODO.txt`. Spec-gap items promote
  to other docs through `/design-review` instead.
- Does not run any git commands (rule 7).
- Does not run any tests (rule 8). The "tests" here are a manual
  walkthrough that Jamie executes; Claude proposes the plan,
  Jamie runs it.
- Does not push to remotes.
- Does not invent new project affordances to make a test step
  easier (rule 4). No new CLI commands, no new seeders, no
  test-mode routes. Use what the phase shipped; if it shipped
  nothing for some setup step, use the REPL / a manual recipe.
- Does not edit prior `LANDED` test plan files. They are
  historical record.
- Does not re-litigate findings already dispositioned in prior
  rounds of the same plan. §5 grows append-only across rounds;
  earlier rows are not modified.
- Does not auto-land a document. Stage 2's last step asks Jamie
  explicitly whether to land or open another polish round —
  there is no implicit transition to `LANDED`.
- Does not surface a commit handoff inline. Artifact-boundary
  stages — Stage 1 initial (new artifact created) and Stage 2
  landing (artifact closing) — invoke `/wind-down`, which owns the
  commit handoff. Mid-iteration stages (Stage 1 addendum branch,
  Stage 2 opening another round) do not wind down at all;
  wind-down stages pending changes when Jamie wraps the session.
- Does not generate a third "mid-run troubleshooting" stage.
  Mid-run failures route to normal coding sessions, with the
  TC's Fail signals as starting points. The plan is a live,
  editable surface during execution — Jamie updates the run log
  inline as she goes.
