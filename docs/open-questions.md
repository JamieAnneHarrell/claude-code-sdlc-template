# Open Questions and Engineering Notes

Working surface for unresolved questions, deferred decisions,
abandoned approaches, and things worth revisiting. Update as answers
are found or decisions are made.

`/wind-down` proposes additions to this file at session end. `/onboard`
seeds it with any open questions that the design intake left
unresolved.

## How this file relates to other tracking

- **`TODO.txt`** — questions that *must* be resolved at the start of
  the next session belong there, not here. This file is for genuinely
  open / non-blocking items.
- **`design-decisions.md`** — once a question gets answered with a
  real decision, it moves there.
- **`PROJECT_PLAN.md`** — when an open question becomes scoped work,
  it becomes a phase or a phase deliverable.

## Categories

Use these section headers. Add a category section only when there's a
real entry to put under it; don't pre-create empty sections.

### Deferred User Stories
Things we've thought about and decided not to build yet. Format:
*Context* (what / why), *Proposed approach*, *Open sub-questions*.
Lift the name into PROJECT_PLAN.md when it's time to build.

### Known Limitations
Things the current implementation does NOT handle gracefully, with
the conditions that trigger the limitation and a mitigation if there
is one. Document so future debugging starts here.

### Abandoned Approaches
Things we tried, why they failed, and what replaced them. Useful when
someone (Claude or human) tries the same thing again — the file
reminds them why it didn't work.

### Open Questions
Questions we haven't yet answered, with what we know so far and what
would unblock an answer.

---

### Open Questions

#### Sharpen rule 7 commit-message brevity constraints

*What we know.* The 2026-05-25 friction session added a "re-read
rule 7 end-to-end before drafting" trigger to `/wind-down`,
`/design-review` Stage 1 initial + Stage 2 landing, and
`/exit-test-plan` Stage 1 initial + Stage 2 landing. That addresses
*not* re-reading the rule; it doesn't address the rule's text
itself, which today says "body is at most a few sentences plus an
optional short bullet list" — soft enough that drift is easy.

*Options to consider.* Phase 1.2's R4b is already scoped to rework
rule 7 brevity ("subject + zero or one body sentence" as default;
bullets cite doc IDs; PowerShell mechanics relegated to a reference
sub-section). Specific constraints to consider during R4b:

- Body ≤ 3 sentences, OR ≤ 4 bullets — pick one shape.
- Each bullet ≤ 1 line at typical terminal width.
- Subject ≤ 60 chars (tighter than current 72).
- "If the body is more than the subject's elaboration, it's too long."

*What would unblock.* Phase 1.2 (`Prompt 1.2` in
`docs/CLAUDE_CODE_PROMPTS.md`) executes the R4b rework; this OQ
captures the candidate constraints to consider then.

#### CLAUDE.md merge strategy for `/refresh-from-repository --merge`

*What we know.* Rules files have `<!-- ONBOARD-FILL: ... -->`
marker blocks that demarcate downstream-owned content from
template-owned content. `CLAUDE.md` does not — `/onboard` currently
rewrites it wholesale based on project answers. So the
`--merge` stage can cleanly merge rules files (take upstream content
outside markers, preserve downstream content inside markers) but
needs a different strategy for `CLAUDE.md`.

*Options to consider.*
- Add equivalent `<!-- ONBOARD-FILL: ... -->` markers to
  `CLAUDE.md` during `/onboard` so the same merge logic applies.
- Diff `CLAUDE.md` against the upstream version and surface
  differences for Jamie to disposition manually (no auto-merge).
- Skip `CLAUDE.md` from `--merge` entirely; only refresh commands
  and rules. Downstream re-runs `/onboard` if they want
  `CLAUDE.md` regenerated.

*What would unblock.* Phase 2 design session that walks the merge
strategy explicitly. Should pick one of the above (or invent a
fourth) and live with the implications.

### Deferred User Stories

#### Multi-agent-rules.md output shape under-specified in /onboard

*Context.* `/onboard` rewrites `rules/multi-agent-rules.md`
wholesale based on the mode Jamie picks. The spec in
`.claude/commands/onboard.md` Step "rules/multi-agent-rules.md"
is thin — for `explore-plus-plan` it says only "Explore + Plan.
State when each is appropriate." ds-niche-stream (the most
mature downstream consumer) evolved a much richer shape that
`/onboard` would not naturally produce for a new project:

- "Use Explore for:" / "Don't use Explore for:" structured
  lists with concrete examples
- "Use Plan for:" / "Don't use Plan for:" same structure
- "What's NOT enabled in this project" section (no
  implementation agents, no worktrees, with rationale)
- Briefing rule embedded directly in the file rather than
  referenced from global `~/.claude/CLAUDE.md` (the current
  spec says reference; ds-niche-stream embeds, which is more
  robust to global-file edits)

A new project onboarded today gets the sparse shape, not the
evolved shape.

*Proposed approach.* Address via a `/design-review` checkpoint
after this project's `/onboard` runs. The review can compare
this project's freshly-produced `rules/multi-agent-rules.md`
against ds-niche-stream's evolved version and decide:

- Whether to beef up `onboard.md`'s spec to describe the
  Use For / Don't Use For structure, OR
- Whether to ship ds-niche-stream's structured shape as a
  scaffold in the placeholder file itself (with `/onboard`
  filling in mode-specific details).

*Open sub-questions.* Briefing rule — embed per-project (ds-
niche-stream's pattern) or reference from global (current
`/onboard` spec)? Embedding survives global-file edits;
referencing avoids duplication.

#### In-flight artifact status callout at top of CLAUDE.md

*Context.* ds-niche-stream's CLAUDE.md opens with a 🟡 callout
summarizing current in-flight artifact state (e.g. "Phase 1.3 exit
`AWAITING-DISPOSITIONS` — paused on... Next action is
`/design-review` ... Phase 2 is blocked behind the landing"). At
session start this greets the developer with the single most
important piece of "where are we" context without requiring a
TODO.txt or test-plan scan. ds-niche-stream appears to maintain
this manually.

*Proposed approach.* Surface as a `/wind-down` enhancement:
detect when an artifact is mid-flight (any
`docs/design/design-review-checkpoint-*.md` is
`AWAITING-DECISIONS`, or any `docs/test-plans/phase-*-exit.md` is
`AWAITING-DISPOSITIONS`), and write/refresh a 🟡 callout block at
the top of CLAUDE.md describing the state and the next action.
Clear the callout when nothing is mid-flight. Marker-block
delimited so it doesn't conflict with the rest of CLAUDE.md.

*Open sub-questions.* Whether `/wind-down` should also detect
`/exit-test-plan` polish-loop states (Stage 1 addendum branch,
Stage 2 open-another-round) and reflect those in the callout.
Whether the callout belongs in CLAUDE.md or in a separate file
like `STATUS.md` that CLAUDE.md links to. Whether to introduce a
fourth status comment family (something like
`<!-- INFLIGHT-STATUS: ... -->`) for machine-parseable state.

#### "Project quick orientation" section in onboarded CLAUDE.md

*Context.* ds-niche-stream (the most mature downstream consumer)
evolved a rich `## Project quick orientation` section in its
CLAUDE.md below the banner. Contents include a multi-paragraph
project description, stack bullets that go beyond the banner
(Admin UI, Public stack, Database engine, Formatter / static
analysis, Main source path with a pointer into
`docs/ARCHITECTURE.md`), and full format/lint commands. Neither
`/onboard` nor `/bootstrap`'s spec produces this section — the
banner alone (Project / Language / Multi-agent / Shell / Run /
Test / Developer setup) is terse. A new project onboarded today
would miss the things that come up often at session start: DB
engine, source-tree location, public-vs-admin stack split for
web projects.

*Proposed approach.* Address via the same `/design-review`
checkpoint that handles `multi-agent-rules.md` (above), after
this project's `/onboard` + `/bootstrap` run. The review can
compare the freshly-produced CLAUDE.md against ds-niche-stream's
shape and decide whether `/bootstrap` should write a "Project
quick orientation" section alongside the banner, or whether the
banner alone is enough (KISS — don't add a section every project
has to maintain).

*Open sub-questions.* If the section is added, which command
owns it — `/onboard` (most project context is already collected
there) or `/bootstrap` (knows the stack + commands)? How much
overlaps with the banner before the duplication starts hurting?

#### Placeholder qualifiers persist into configured CLAUDE.md

*Context.* The dist's `cc-template/CLAUDE.md` placeholder is
written for the unconfigured state and includes phrasings that
become stale once onboarding completes:

- `## Reading order at session start (configured projects)` —
  the `(configured projects)` parenthetical is a flag for
  pre-onboard readers; after `/onboard` it's redundant.
- Collaboration rules section: `rules/environment-rules.md —
  ... Project-specific environment is filled in by /bootstrap.`
  is stale after `/bootstrap` runs.
- Same for `rules/multi-agent-rules.md — ... (filled in by
  /onboard based on the chosen mode).` after `/onboard` runs.

Both `/onboard` and `/bootstrap` specs say "keep collaboration
rules and references unchanged," so these stale phrasings
persist forever in every onboarded project. ds-niche-stream has
them cleaned up — likely Jamie's manual edit.

*Proposed approach.* Two fix shapes:

- **A (KISS).** Reword the placeholder so it reads correctly in
  both states (drop `(configured projects)`, drop the "filled
  in by" qualifiers). No command-side logic added. Apply to
  both root and `cc-template/`.
- **B.** Update `/onboard` and `/bootstrap` specs to strip the
  stale qualifiers as part of their CLAUDE.md rewrites. More
  work, more invariants.

Recommend A; defer the decision to the same `/design-review`
checkpoint that handles the other CLAUDE.md drift findings
above.

*Open sub-questions.* Whether shape A breaks the placeholder's
readability for a developer encountering an unconfigured
template for the first time — the qualifiers serve a real
purpose pre-onboard.

#### Universal-rules customization partitions

*Context.* FR-11 says universal rules content — the 9
coding-session rules, design-philosophy, and the placeholder rules
files (environment / multi-agent / project / testing) — ships
identical between root and `cc-template/`. NFR-9 acknowledges the
duplication is deliberate but creates drift risk.

This session's `/bootstrap` filled the
`<!-- ONBOARD-FILL: environment -->` block in root
`rules/environment-rules.md` with Jamie-specific maintainer
content (PowerShell on Windows 11, VSCode + Claude Code
extension). That content is correct for the source-of-truth
project but should NOT appear in `cc-template/rules/environment-rules.md`,
which must stay generic for downstream consumers who may use a
different shell or editor.

The existing `<!-- ONBOARD-FILL: ... -->` marker pattern handles
this single case — the dist keeps the placeholder; the source
fills it during its own `/bootstrap` run. But the partition lives
at the bottom of one specific file, and the pattern doesn't
generalize. Other rules files may have sections that ought to
differ between source and dist, or that downstream consumers
might legitimately need to customize per project:

- `rules/coding-session-rules.md` rule 7: PowerShell-quoting
  mechanics that are Windows-specific (would be different on a
  bash-primary project)
- `rules/design-philosophy-rules.md`: any project-specific
  overlays on the general principles
- `rules/testing-rules.md`: language-specific test commands (a
  Python project's "run the tests" line differs from a PHP one)
- `rules/project-rules.md`: already has a project-scope
  ONBOARD-FILL block, but other sections may need similar
  treatment as the rule grows

*Options to consider.*
- **A. Expand the ONBOARD-FILL pattern.** Add
  `<!-- ONBOARD-FILL: ... -->` markers to whichever sections of
  whichever rules files might need divergence — universal content
  outside markers (template-owned), project-specific content
  inside (downstream-owned). `/refresh-from-repository --merge`
  (Phase 2.1) already needs to preserve `ONBOARD-FILL` content,
  so extending the pattern keeps merge semantics identical.
- **B. Allow per-file divergence.** Relax FR-11 / NFR-9 to
  "universal sections of universal files identical." Specific
  files (e.g. `environment-rules.md`) can diverge wholesale
  between root and dist. Higher drift risk unless an automated
  check (Phase 3) compares only the universal sections.
- **C. Status quo.** Keep all rules files universal-identical.
  Treat Jamie's bootstrap-time fill in root
  `environment-rules.md` as a single-file exception (the
  ONBOARD-FILL block is *itself* the partition, and the dist's
  unfilled placeholder IS the consumer-customizable surface).
  Don't generalize.

*What would unblock.* Audit each universal rules file
section-by-section for "ought to differ between source and dist"
content. This could fold into Phase 1.2 (rules + CLAUDE.md
cleanup pass) which already touches every rules file, or get its
own `/design-review` checkpoint. Decision affects
`/refresh-from-repository --merge` semantics (Phase 2.1) and
Phase 3 regression-check shape (what counts as "drift" vs. what
counts as "legitimate divergence").

#### Source-only release/build helper command

*Context.* Today the dist subdir is kept in sync with the source
manually — edit in the dist when a shipping change is needed,
copy up to root when it's a command this project itself uses.
If drift between root copies and dist copies of the same file
becomes painful, a small source-only command (e.g. `/sync-root-to-dist`
or `/release`) could automate the diff/copy step.

*Proposed approach.* Defer until pain is felt. Manual sync is the
v1 answer and git diff at commit time catches accidental drift.

*Open sub-questions.* Whether the sync is dist→root or root→dist
(i.e. which side is canonical for shared content like the recurring
commands). Current convention: edit in dist first.

#### Regression-test automation for the distributable

*Context.* The dist is a copy-paste seed that must continue to work
end-to-end (full /onboard → /design-review → /bootstrap → ...
chain). Today regression checking is manual ("copy cc-template/ to
a sandbox dir and run the chain"). A scripted check could diff the
current dist against a tagged baseline and flag invariant-breaking
changes (status comment renames, ONBOARD-FILL marker drift,
zero-pad width changes in checkpoint/test-plan filenames).

*Proposed approach.* Phase 3 roadmap. Specifics deferred until we
have a real regression to motivate the work.

