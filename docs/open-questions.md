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

#### Public GitHub URL for the source repo

*What we know.* Phase 2 ships `/refresh-from-repository` (see
`docs/design-decisions.md` entry 2), which pulls from the public
upstream. The command needs a baked-in URL or a configurable one.

*What would unblock.* Decide the GitHub org/path the source repo
will live at (e.g. `github.com/<org>/jah-template-project`). Affects
the URL constant in `/refresh-from-repository` and any references
in shipping docs that point at the upstream.

*Not blocking today.* Phase 2 is downstream of Phase 1
(post-restructure exit-test-plan). Decide before Phase 2 design
starts.

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

