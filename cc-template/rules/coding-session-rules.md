# Coding Session Rules — the 9 standing rules

These are Jamie's standing rules for every coding session, on every
project. They apply without exception unless a project's
`rules/project-rules.md` explicitly modifies one. Mid-session, if Jamie
references a rule by number ("rule 4", "this is a rule 1 issue"), that
is a drift signal — acknowledge, correct course, move on.

**Read together with [`rules/design-philosophy-rules.md`](design-philosophy-rules.md).**
These 9 rules are the behavioral guardrails; design-philosophy-rules.md
is the judgment framework (KISS, progressive disclosure) that informs
*how* you apply them. Rule 4 in particular is the gate that protects
KISS from drift — load both at session start.

---

## Rule 1: Fix root causes only

Do not paper over symptoms. If a test fails, find why. If a function
returns wrong data, fix the logic — do not add a workaround in the
caller. If you find yourself writing code that says "if this weird
edge case, do this weird thing," stop and investigate why the edge
case exists.

No try/except blocks that swallow errors to make things "work." No
default values inserted to hide missing data. No retries to mask a
real failure mode.

## Rule 2: Trust Jamie's diagnosis

When Jamie says "the motion detector is firing on shadows" or "the
keyframe snap is off by one," that is the problem. Do not second-guess
it. Do not propose that maybe the issue is elsewhere. Fix what she
identified. If after investigation the diagnosis turns out to be
incomplete, surface what you found with evidence, not speculation.

## Rule 3: Rejections are permanent

If Jamie rejects an approach, library, design choice, or feature, it
stays rejected for the rest of the session and all future sessions on
this project. Do not re-propose it with a new framing. Do not bring it
back up three messages later. If you think the rejection was based on
incomplete information, say so once with the new information — but
once she re-rejects, it is closed.

## Rule 4: No unsolicited design decisions

Do not add features, options, modules, or abstractions that were not
requested. Do not decide on your own to add a GUI because "users might
want one." Do not add a config file loader because "it seems useful."
Do not add caching, retries, parallelism, or performance optimizations
unless asked.

If you believe something is needed that wasn't specified, ask first.
Propose it, wait for approval, then implement.

The requirements, architecture, and project plan documents define the
scope. Anything outside those documents needs explicit approval before
implementation.

## Rule 5: Decouple data from display

Never conflate what the program knows with what the program shows.
Detection results are data. Cut specifications are data. Manifests are
data. Rendering them to stdout, to JSON, to a progress bar, to a
summary — those are separate concerns.

This rule is why most architectures have distinct compute and report
modules. Do not push display concerns into the compute layer. Do not
push business logic into the CLI layer.

## Rule 6: Reference rule numbers mid-session

If Jamie says "rule 4" or "this is a rule 1 issue" mid-session,
recognize it as a drift signal and correct course. Acknowledge, fix,
move on.

## Rule 7: Jamie runs all commits

Claude does not run `git commit`, `git push`, or `git tag` directly.
When a commit is appropriate, Claude surfaces the exact git commands —
staging plus commit with a proposed message — so Jamie can review the
diff, message, and scope before running them herself.

Rationale: commits are the durable record of this project. Jamie wants
to audit each one before it lands. This applies even if a hook or
workflow would otherwise make a commit automatic, and it supersedes
any session-level convenience.

Commit messages still follow the project's commit discipline (subject
under 72 chars, imperative mood, one logical change per commit). Claude
proposes the message in the handoff.

### Commit handoff format

Every commit proposal must satisfy these constraints together. Skipping
any one is a regression.

1. **Pre-commit dry-run sequence**, in order:
   1. `<formatter> --check <source paths>` — detect formatting drift
      before staging. (Python: `ruff format --check`.)
   2. `<formatter> <source paths>` — only if step 1 reported drift.
   3. `git add <paths>` — stage the intended files (may include `.md`).
   4. `git status` — confirm no `MM` rows remain.
   5. `pre-commit run` — dry-run hooks against the staged snapshot. If
      any hook reports "files were modified by this hook", re-run
      `git add` and repeat.
   6. `git commit -m "<subject>"` — only after step 5 is green.

2. **One command per copyable line.** Each shell command appears on its
   own line in its own fenced code block (or, in a multi-command block,
   alone on its line with no trailing comment). Never join commands
   with `;`, `&&`, or `||` in a handoff. Jamie copies one line at a
   time and pastes; chained commands force editing before pasting.

3. **Filter formatter inputs to relevant file types.** Ruff does not
   process Markdown; eslint does not process Python. Filter
   `<source paths>` to the formatter's target file types. `git add`
   and `pre-commit run` still cover the full set including `.md`.

4. **Commit messages stay short.** Subject under 72 chars, imperative
   mood. Body is at most a few sentences plus an optional short bullet
   list. Bullets reference requirements or design docs ("FR-2",
   "ARCHITECTURE.md Data Flow") rather than spelling out
   implementation. If a body bullet runs to a paragraph, it is too
   long.

5. **The commit command is one copy/paste-ready block.** Jamie copies
   and runs the `git commit` line verbatim — she does not assemble
   subject and body from prose. This means:
   - Use a single `-m "..."` flag containing the entire message
     (subject, blank line, body, bullets) inside one double-quoted
     string with literal newlines. Both git-bash and the VSCode
     PowerShell terminal accept this form when pasted as one block.
   - **Never** use multiple `-m` flags to stack subject and body.
   - **Never** present the message as separate `Subject:` / `Body:`
     prose sections for Jamie to reassemble. If the body needs editing,
     Jamie edits inside the quoted string before pasting — but the
     handoff itself is always a complete, runnable command.
   - **Never** use here-docs (`<<EOF`) — those are not portable between
     git-bash and PowerShell.
   - Use PowerShell single-quoted here-strings (`@'...'@`).
   - When using here-strings, messages cannot have double quotes in them.
     Avoid single quotes to be extra careful.
   - If the message has no body, a single-line
     `git commit -m "Subject"` is fine.

   Correct shape (one fenced block, one command):
   ```
   git commit -m "Subject under 72 chars

   Short body sentence referencing FR-2.
   - bullet one
   - bullet two"
   ```

## Rule 8: Jamie runs automated tests

Conserve energy by asking Jamie to run the tests; give instructions on
what tests to run, command lines to execute, and expected outcomes
("expect 2 tests skipped and 5 successful"). Jamie pastes any errors
back or confirms no issues.

This rule also covers **manual phase-exit walkthroughs**. The
canonical flow is `/exit-test-plan` — Stage 1 authors the plan from
the phase's exit criteria, Jamie runs it top-down and marks §4's run
log inline, then Stage 2 reads the run log + trailing notes and
lands dispositions. The plan is the artifact; Claude never runs it.
Mid-run failures are normal coding work driven by the Fail signals
inside each TC.

## Rule 9: Session wind-down rewrites TODO.txt

`TODO.txt` is the handoff between sessions, not a running history. At
wind-down — explicit ("calling it", "off to bed", "that's enough for
today", "done for tonight") or implicit (clear end-of-session signals)
— rewrite it, do not append to it.

When wind-down cues appear in conversation, surface a suggestion to run
`/wind-down` (or its equivalent ritual). Don't run it autonomously;
wait for Jamie's go-ahead.

Required shape after rewrite:

- **Completed items are removed**, not struck through, not annotated as
  "done". The git log is the record of what was done; `TODO.txt` is
  the record of what is next.
- **The first entry is what we pick up next session**, written in the
  form "At the beginning of the next session, ..." so Jamie sees it
  the moment she opens the file.
- Remaining entries are the upcoming queue in priority order.
- Preserve the handoff format Jamie is already using in the file
  (headers, sections, etc.); only the contents change.

Wind-down also keeps the rest of the repository documents coherent. Not
every doc changes every session — only surface the ones that apply,
and propose the edits for Jamie to approve:

- **`docs/CLAUDE_CODE_PROMPTS.md`**: when a planned prompt landed this
  session, mark it as landed. If the session deviated significantly
  from the prompt's original plan, note the deviation alongside the
  landing marker. Prompts that landed in prior sessions are never
  edited retroactively.
- **`docs/design-decisions.md`**: record significant design decisions
  made this session — the problem, the options considered, and what we
  chose and why.
- **`docs/open-questions.md`**: move unresolved questions raised this
  session here, *unless* the question must be resolved at the start of
  the next session to make progress — those belong in `TODO.txt`
  instead.
- **`docs/ARCHITECTURE.md` / `docs/REQUIREMENTS.md`**: if the session
  changed architecture or requirements, surface the diff for those
  documents so they stay aligned with the code.
- **`docs/[command].md` or `docs/commands/*.md`**: if the project
  produces user-runnable commands and functionality or command line
  parameters change in a coding session, ensure the command documents
  are updated.

The goal: at the end of every session, the repo's documents reflect
reality. If they don't, surface the gap before closing — even if the
fix is a follow-up next session.
