---
description: Run the session wind-down ritual — rewrite TODO.txt, update tracking docs, and surface the commit handoff. /wind-down owns the commit-handoff ritual.
---

# /wind-down

Run the session wind-down ritual at the end of a session, before
Jamie steps away. `/wind-down` is the canonical owner of the
session-close ritual: the `TODO.txt` rewrite, the doc-coherence
sweep, and the commit handoff. Coding-session rule 9 routes here;
the commit-handoff mechanics that used to live in rule 7 now live in
Step 4.

Jamie can invoke this explicitly (`/wind-down`), but Claude should
also surface the suggestion to run it whenever wind-down cues appear
in conversation: "calling it", "off to bed", "that's enough for
today", "done for tonight", "we're done", etc. **Don't run it
autonomously** — surface the suggestion and wait for Jamie's
go-ahead.

When another skill invokes `/wind-down` at an artifact boundary
(e.g. `/design-review` or `/exit-test-plan` at a landing),
`/wind-down` owns the commit handoff and the doc-coherence sweep for
any docs the calling skill doesn't own — run the full ritual below,
scoped to what the session actually changed.

---

## Step 1: Take stock of the session

Quickly recap:
- What was the goal of this session?
- What landed (code committed or staged)?
- What was started but didn't finish?
- What was discovered (design decisions made, questions raised, things
  that surprised us)?

State this back to Jamie as a 4–6 bullet recap so she can confirm and
correct.

## Step 2: Rewrite TODO.txt

`TODO.txt` is the handoff between sessions, **not** a running history.
At wind-down, **rewrite** it — do not append.

Required shape after rewrite (this is the canonical spec):

- **Completed items removed.** No strikethroughs, no "(done)"
  annotations. The git log is the record of what was done; TODO.txt
  is the record of what is next.
- **First entry is what we pick up next session**, written in the
  form: "At the beginning of the next session, ..."
- **Reference prompts by ID, never inline.** "Run Prompt 1.2 from
  `docs/CLAUDE_CODE_PROMPTS.md`" — not the prompt body, scope, or
  constraints.
- **Don't queue past the immediate next session.** Roadmap items
  belong in `docs/PROJECT_PLAN.md`; open questions belong in
  `docs/open-questions.md`. TODO.txt = exactly the immediate next
  step + decisions that genuinely block starting next session.
- **Preserve the format Jamie is using** in the file (headers,
  sections, separators). Only the contents change.

**Design-review safety net.** Before finalizing the rewrite, scan
`docs/design/` for any `design-review-checkpoint-*.md` whose
frontmatter status is `AWAITING-DECISIONS`. If one exists,
identify the **latest round** in the document (the most recent
`## Addendum N — date` section if any exists, otherwise the
original Findings section) and check both its AUDIT NOTE state
and whether the `## Disposition log` table has rows whose `Round`
column matches that round (`Original` for Round 1, `Addendum N`
for the Nth addendum).

The first TODO.txt entry MUST reflect which case applies. Four
shapes; pick the one that matches:

- **Latest round has unmarked AUDIT NOTE blocks AND the round is
  Round 1 (Findings).** Initial checkpoint authored but not
  marked. First entry: "At the beginning of the next session,
  mark up `docs/design/design-review-checkpoint-NNN.md` (decide
  each finding inline per the legend in the doc, one decision per
  AUDIT NOTE block). Then re-run `/design-review`. Stage 2 walks
  each disposition and asks whether to land the doc or open
  another addendum round."
- **Latest round has unmarked AUDIT NOTE blocks AND the round is
  an Addendum N.** Addendum authored but not marked. First entry:
  "At the beginning of the next session, mark up Addendum N of
  `docs/design/design-review-checkpoint-NNN.md` (decide each
  finding inline per the legend at the top of the doc). Then
  re-run `/design-review`. Stage 2 walks Addendum N's
  dispositions and asks whether to land the doc or open
  Addendum N+1."
- **Latest round all marked AND the Disposition log has no rows
  whose `Round` matches it.** Round walk hasn't happened yet.
  First entry: "At the beginning of the next session, re-run
  `/design-review` to walk the latest-round dispositions in
  `docs/design/design-review-checkpoint-NNN.md`."
- **Latest round all marked AND the Disposition log has rows for
  that round.** Stage 2 walked the round; Jamie picked "open
  another addendum"; the doc is between rounds, waiting for the
  research / clarification work before the next addendum is
  authored. First entry should be the
  "investigate <X> before Addendum N+1" entry that Stage 2 wrote
  into TODO.txt — likely already there. Verify it's still
  accurate. If TODO.txt's first entry is something else, propose
  restoring the investigate-before-addendum entry as the first.

If TODO.txt's first entry doesn't reflect the applicable case,
surface a warning to Jamie and propose the corrected first entry
— don't silently drop the review and prioritize something else.
Jamie can override (some sessions she may deliberately defer the
review), but the warning surfaces every wind-down so the open
review doesn't get forgotten.

**Exit-test-plan safety net.** Same pattern. Scan
`docs/test-plans/` for any `phase-*-exit.md` whose frontmatter
status is `AWAITING-DISPOSITIONS`. If one exists, identify the
**newest run log** in the document (§4 if no §6.N addendum
exists, otherwise the latest §6.N) and check both its `[PENDING]`
state and whether §5 has dispositions covering it.

The first TODO.txt entry MUST reflect which case applies. Four
shapes; pick the one that matches:

- **Newest run log has `[PENDING]` rows AND it's §4 (original
  run never run yet).** Plan has been authored but not run. First
  entry: "At the beginning of the next session, run
  `docs/test-plans/phase-NNN-exit.md` top-down — work §2 prep,
  then the §3 test cases. Mark §4's run log inline and capture
  inline notes for surprises. If any TC fails, the same session
  is interactive troubleshooting (Fail signals point at the
  fix-site). After every TC is run, re-run `/exit-test-plan` to
  land dispositions."
- **Newest run log has `[PENDING]` rows AND it's a §6.N addendum.**
  Addendum has been authored but not run. First entry: "At the
  beginning of the next session, run §6.N of
  `docs/test-plans/phase-NNN-exit.md` top-down. Mark §6.N's run
  log inline and capture comments. After every TC is run, re-run
  `/exit-test-plan` to land addendum N (or open addendum N+1 if
  more polish surfaces)."
- **Newest run log is filled but §5 has no row whose `Round`
  matches it.** Run has happened but dispositions haven't landed.
  First entry: "At the beginning of the next session, re-run
  `/exit-test-plan` to land dispositions for the newest run in
  `docs/test-plans/phase-NNN-exit.md`."
- **Newest run log is filled AND §5 has dispositions for it.**
  Dispositions landed; the doc is between rounds, waiting for the
  polish coding session before the next addendum. First entry
  should be the polish-batch entry that Stage 2 wrote into
  TODO.txt — likely already there. Verify it's still accurate.
  If TODO.txt's first entry is something else, propose restoring
  the polish-batch entry as the first.

If TODO.txt's first entry doesn't reflect the applicable case,
warn Jamie and propose the corrected first entry. Same
override-allowed posture as the design-review safety net.

If both a design-review checkpoint and a test plan are in flight
at once, surface both and ask Jamie which to prioritize as the
first entry (the other becomes the second). Don't guess the
ordering.

**How to apply the rewrite.** State the intent in one short
sentence ("I'll rewrite TODO.txt with the next-session entry
pointing at Prompt 1.2 plus the open-questions cleanup item"),
then call Edit/Write directly. Do **not** paste the full proposed
TODO.txt back into chat for pre-approval — the edit-approval mode
is the per-edit review surface and pre-pasting duplicates it.

## Step 3: Update tracking docs (only those that apply)

Wind-down keeps the rest of the repo's docs coherent — but not every
doc changes every session. Surface only the ones with real diffs to
apply. For each: state the intent in one short sentence, then edit
directly via the Edit tool — don't paste the full proposed text in
chat (the edit-approval mode is the per-edit review surface, and
pre-pasting duplicates it). Multi-doc structural changes warrant a
brief plan first.

### `docs/CLAUDE_CODE_PROMPTS.md`
- If a planned prompt landed this session, mark it landed in the
  prompt's "revisions since this prompt ran" footer — a date marker
  plus, if applicable, terse deviation bullets.
- **The footer is for plan deviations only — never a recap of what
  landed.** The git log is the recap; the prompt body is the plan;
  the footer captures where the plan had to change. Example
  deviation: "Scope item 5 skipped — conditional on §3 contract
  changing, which it didn't." Not a deviation: "Edited file X to
  add Y" — that's commit-log material.
- If the session was a clean execution of the prompt, the landing
  marker is enough — no bullets.
- **Do not edit prompts that landed in prior sessions.** Those are
  historical record.

### `docs/design-decisions.md`
- Record any significant design decision made this session.
- Format: **Decision** → **Why** → **Why not (alternatives)**.
- Only record decisions that future-Claude would benefit from
  knowing. Trivial implementation choices don't belong here.

### `docs/open-questions.md`
- Move unresolved questions from this session to this file.
- **Exception:** if a question must be resolved at the start of the
  next session to make progress, it belongs in TODO.txt instead of
  open-questions.md.
- Categorize: Deferred User Stories / Known Limitations / Abandoned
  Approaches / Open Questions.

### `docs/REQUIREMENTS.md` and `docs/ARCHITECTURE.md`
- If the session changed requirements or architecture, state the
  intent ("I'll add NFR-N to REQUIREMENTS covering X") and call
  Edit directly. Edit-approval gates the change — no pre-pasted
  diff in chat.
- These are slower-changing docs; most sessions don't touch them.

### `docs/PROJECT_PLAN.md`
- If a phase landed, mark its status.
- If scope changed (phase split, phase reordered, new phase inserted),
  state the intent and edit directly.

### `CLAUDE.md`
- The post-configuration banner is static orientation — it should
  not name a specific next prompt or phase (next-step lives in
  `TODO.txt`). If the banner *does* name one, verify it matches
  TODO.txt's first entry; if it has drifted, surface a warning and
  propose either dropping the next-step prose or correcting it.
  Don't silently rewrite the banner.

### `docs/[command].md` or `docs/commands/*.md`
- If a command's CLI parameters or functionality changed this
  session, update its command doc to match.

### `docs/design/design-review-checkpoint-*.md` and `docs/design/REVIEWS.md`
- If a checkpoint changed status this session
  (`AWAITING-DECISIONS` → `LANDED`, or a new `AWAITING-DECISIONS`
  checkpoint was written), note it in the session recap.
- **Do not edit checkpoint files.** They are owned by
  `/design-review` — stage 1 writes them, stage 2 fills the bottom
  sections and flips the frontmatter to `LANDED`. Wind-down only
  reports state.
- The `REVIEWS.md` index is also `/design-review`-owned; wind-down
  does not edit it.

### `docs/test-plans/phase-*-exit.md`
- If a test plan changed state this session — any of: a new
  `AWAITING-DISPOSITIONS` plan was written; any run log (§4 or
  any §6.N) went from `[PENDING]` rows to filled; a new §6.N
  addendum was appended; §5 grew with a new round's dispositions;
  or the doc flipped to `LANDED` — note it in the session recap.
- **Do not edit test plan files.** They are owned by
  `/exit-test-plan` — Stage 1 writes the initial plan or
  appends a §6.N addendum; the tester (Jamie) fills §4 or §6.N
  inline during the run; Stage 2 appends §5 dispositions for the
  newest run; the doc flips to `LANDED` only when Stage 2
  explicitly decides "land" after Jamie confirms there's no
  outstanding polish or fix-now work. Wind-down only reports
  state.

The goal: at session end, the repo's docs reflect reality. If they
don't, surface the gap before closing — even if the fix is a
follow-up next session.

## Step 4: Surface the commit handoff

`/wind-down` owns the commit-handoff ritual (coding-session rule 7
routes here). If there are uncommitted changes (staged or unstaged)
that should land before the session ends, surface the handoff per
the spec below. If there are none, skip this step.

**Trust Step 1's recap.** Don't re-run `git status` to rediscover
what changed, and don't re-verify state the rules already establish
(e.g. `TODO.txt` is gitignored). Project tooling — formatter,
pre-commit, test commands — lives in `rules/environment-rules.md`'s
"Build / run / test commands" subsection. Don't probe the
filesystem for `.pre-commit-config.yaml` or other configs. If that
subsection says N/A, skip the formatter and pre-commit steps below.

**Do not run the commit yourself.** Jamie reviews diff, message, and
scope, then pastes the commands.

### Commit messages stay short

Default shape: **subject + zero or one body sentence.** Subject ≤72
chars, imperative mood, one logical change per commit.

Body bullets (when used) cite doc IDs — `FR-N`, `NFR-N`,
`ARCHITECTURE §N`, `Prompt N`, finding IDs like `R4b` — instead of
restating context the diff or referenced doc already covers. If a
body bullet wants to be a paragraph, it's the wrong shape: make it a
doc reference, not a re-explanation. Multi-paragraph bodies are
rare, not normal.

### Constraints — every handoff satisfies all of these

1. **Pre-commit dry-run sequence**, in order:
   1. `<formatter> --check <source paths>` — detect formatting
      drift before staging. (Python: `ruff format --check`.)
   2. `<formatter> <source paths>` — only if step 1 reported drift.
   3. `git add <paths>` — stage the intended files (may include
      `.md`).
   4. `git status` — confirm no `MM` rows remain.
   5. `pre-commit run` — dry-run hooks against the staged snapshot.
      If any hook reports "files were modified by this hook",
      re-run `git add` and repeat.
   6. `git commit -m "<subject>"` — only after step 5 is green.

2. **One command per copyable line.** Each shell command on its own
   line in its own fenced code block (or, in a multi-command block,
   alone on its line with no trailing comment). Never join commands
   with `;`, `&&`, or `||` — chained commands force Jamie to edit
   before pasting.

3. **Filter formatter inputs to relevant file types.** Ruff doesn't
   process Markdown; eslint doesn't process Python. Filter
   `<source paths>` to the formatter's target file types. `git add`
   and `pre-commit run` still cover the full set including `.md`.

4. **The commit command is one copy/paste-ready block.** Jamie
   copies and runs the `git commit` line verbatim — she does not
   assemble subject and body from prose.

### Bundled vs split

Default to one bundled commit. If the changes cover unrelated
concerns and Jamie wants the git log to reflect that, offer a split
sequence — one commit per logical change, each with a focused
subject. Let Jamie pick.

### Mechanics reference — PowerShell quoting

- Single `-m "..."` flag holds the entire message (subject, blank
  line, body, bullets) inside one double-quoted string with literal
  newlines. Both git-bash and the VSCode PowerShell terminal accept
  this form when pasted as one block.
- **Never** use multiple `-m` flags to stack subject and body.
- **Never** present the message as separate `Subject:` / `Body:`
  prose sections for Jamie to reassemble.
- **Never** use here-docs (`<<EOF`) — not portable between git-bash
  and PowerShell.
- PowerShell single-quoted here-strings (`@'...'@`) work; messages
  inside cannot contain double quotes, and avoid single quotes to
  be safe.
- If the message has no body, single-line `git commit -m "Subject"`
  is fine.

Correct shape (one fenced block, one command):
```
git commit -m "Subject under 72 chars

One short body sentence referencing FR-2.
- R4a: trimmed env-rules version-freshness repetition
- R4b: rule 7 brevity foregrounded"
```

## Step 5: Final report

Tell Jamie:
- What was rewritten / updated (TODO.txt, design-decisions.md, etc.)
- Whether there's a pending commit handoff to run
- Any open questions that need an answer before the next session can
  start
- "Wind-down complete. See you next session."

---

## What this command does NOT do

- Does not run `git commit` (rule 7).
- Does not push to remotes.
- Does not edit files in past sessions' history (the prompt landing
  notes are forward-only; old prompts are immutable).
- Does not skip steps. If there's nothing to update in a tracking
  doc, say so explicitly — don't silently skip.
- Does not edit `docs/design/design-review-checkpoint-*.md` or
  `docs/design/REVIEWS.md`. Those are `/design-review`'s domain.
  Wind-down surfaces a warning if a review is `AWAITING-DECISIONS`
  and TODO.txt doesn't reflect it, but never edits the checkpoint
  itself.
- Does not edit `docs/test-plans/phase-*-exit.md`. Those are
  `/exit-test-plan`'s domain. Wind-down surfaces a warning if a
  plan is `AWAITING-DISPOSITIONS` and TODO.txt doesn't reflect
  the right next-step (run the plan vs. land dispositions), but
  never edits the plan itself.
