# Coding Session Rules — the 9 standing rules

Jamie's standing rules for every coding session, every project. They
apply without exception unless `rules/project-rules.md` explicitly
modifies one. **Read together with
[`rules/design-philosophy-rules.md`](design-philosophy-rules.md)** —
the 9 rules are behavioral guardrails; design-philosophy is the
judgment framework (KISS, progressive disclosure) for *how* to apply
them. Rule 4 is the gate that protects KISS from drift.

---

## Rule 1: Fix root causes only

If a test fails, find why. If a function returns wrong data, fix the
logic — don't work around it in the caller. "If this weird edge case,
do this weird thing" is a signal to stop and investigate why the edge
case exists.

No try/except that swallows errors to make things "work." No default
values inserted to hide missing data. No retries to mask a real
failure mode.

## Rule 2: Trust Jamie's diagnosis

When Jamie says "the motion detector is firing on shadows" or "the
keyframe snap is off by one," that's the problem. Fix what she
identified. If investigation finds the diagnosis incomplete, surface
what you found with evidence, not speculation.

## Rule 3: Rejections are permanent

If Jamie rejects an approach, library, design choice, or feature, it
stays rejected for the rest of the session and all future sessions
on this project. Don't re-propose with new framing. If the rejection
was based on incomplete information, say so once with the new
information — once she re-rejects, it's closed.

## Rule 4: No unsolicited design decisions

Don't add features, options, modules, abstractions, GUIs, config
loaders, caching, retries, parallelism, or optimizations that weren't
requested. The requirements, architecture, and project plan documents
define scope. Anything outside those docs needs explicit approval
before implementation.

**Self-check before any addition.** When proposing capability,
abstraction, dependency, or configuration that wasn't asked for,
name the simpler alternative **in the same message** and let Jamie
pick: "I could add X for reasons R; the simpler alternative is to do
nothing / do Y." Visible enforcement of KISS (see
[`design-philosophy-rules.md`](design-philosophy-rules.md)). Silently
picking the richer option is the failure mode.

## Rule 5: Decouple data from display

Never conflate what the program knows with what it shows. Detection
results, cut specs, manifests are data; rendering them to stdout,
JSON, a progress bar, or a summary are separate concerns. Don't push
display into the compute layer or business logic into the CLI layer.

## Rule 6: Reference rule numbers mid-session

If Jamie says "rule 4" or "this is a rule 1 issue" mid-session,
treat it as a drift signal. Acknowledge, correct course, move on.

## Rule 7: Jamie runs all commits

Claude does not run `git commit`, `git push`, or `git tag`. When a
commit is appropriate, Claude surfaces the exact git commands —
staging plus commit with a proposed message — so Jamie reviews diff,
message, and scope before running them. Commits are the durable
record; Jamie audits each one before it lands. This supersedes any
hook or workflow that would otherwise auto-commit.

### Commit messages stay short

Default shape: **subject + zero or one body sentence.** Subject ≤72
chars, imperative mood, one logical change per commit.

Body bullets (when used) cite doc IDs — `FR-N`, `NFR-N`,
`ARCHITECTURE §N`, `Prompt N`, finding IDs like `R4b` — instead of
restating context that the diff or referenced doc already covers.
If a body bullet wants to be a paragraph, the bullet is the wrong
shape: it should be a doc reference, not a re-explanation.
Multi-paragraph bodies are rare, not normal.

**Pre-flight.** Before generating any commit handoff, re-read this
section end-to-end. The commands that produce handoffs
(`/wind-down`, `/design-review` Stage 1 initial and Stage 2
landing, `/exit-test-plan` Stage 1 initial and Stage 2 landing)
name this re-read explicitly. Habit-mode handoffs drift long.

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
   assemble subject and body from prose. See Mechanics reference
   below.

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

## Rule 8: Jamie runs automated tests

Ask Jamie to run tests; give the commands, what to run, and expected
outcomes ("expect 2 skipped, 5 successful"). She pastes errors back
or confirms.

This rule also covers **manual phase-exit walkthroughs**. The
canonical flow is `/exit-test-plan` — Stage 1 authors the plan from
the phase's exit criteria, Jamie runs it and marks §4's run log
inline, Stage 2 reads the run log + trailing notes and lands
dispositions. The plan is the artifact; Claude never runs it.
Mid-run failures are normal coding work driven by Fail signals in
each TC.

## Rule 9: Session wind-down rewrites TODO.txt

`TODO.txt` is the handoff between sessions, not a running history.
At wind-down — explicit ("calling it", "off to bed", "done for
tonight") or implicit end-of-session signals — rewrite it, don't
append. When wind-down cues appear, surface a suggestion to run
`/wind-down`; wait for Jamie's go-ahead before executing.

Required shape after rewrite:

- **Completed items are removed**, not struck through. The git log is
  the record of what was done; `TODO.txt` is the record of what is
  next.
- **First entry is what we pick up next session**, written as "At the
  beginning of the next session, ..." so Jamie sees it the moment
  she opens the file.
- **Reference prompts by ID, never inline their content.** "Run
  Prompt 1.2 from `docs/CLAUDE_CODE_PROMPTS.md`" — pasting the prompt
  body duplicates source-of-truth and dates fast.
- **Don't queue past the immediate next session.** Roadmap items go
  in `docs/PROJECT_PLAN.md`; open questions in
  `docs/open-questions.md`. TODO.txt is exactly (a) the immediate
  next step plus (b) decisions that block the next session from
  starting.
- Preserve Jamie's existing handoff format (headers, sections); only
  the contents change.

Wind-down also keeps the rest of the repo's docs coherent — surface
only the docs that need updates this session. State the intent in
one short sentence, then edit directly via the Edit tool (don't
paste full proposed text in chat — that duplicates the edit-approval
review surface). Multi-doc structural changes warrant a brief plan
first.

Per-doc edits to consider:

- **`docs/CLAUDE_CODE_PROMPTS.md`**: mark prompts that landed this
  session; note deviations alongside the landing marker. Prior
  sessions' prompts are never edited retroactively.
- **`docs/design-decisions.md`**: record significant decisions made
  this session — problem, options considered, choice and why.
- **`docs/open-questions.md`**: move unresolved questions here,
  unless they block the next session start (those go in TODO.txt).
- **`docs/ARCHITECTURE.md` / `docs/REQUIREMENTS.md`**: surface diffs
  if the session changed architecture or requirements.
- **`docs/[command].md` or `docs/commands/*.md`**: update command
  docs when CLI parameters or functionality changed.

The goal: at end of every session, the repo's docs reflect reality.
If they don't, surface the gap before closing — even if the fix is a
follow-up next session.
