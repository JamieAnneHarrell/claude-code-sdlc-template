# Coding Session Rules — the 10 standing rules

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
Y." Visible enforcement of KISS (see
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

Claude does not run `git commit`, `git push`, or `git tag`. When
Claude would offer commit commands, that is a definitive signal to
invoke `/wind-down` instead — `/wind-down` owns the commit-handoff
ritual, including the pre-commit dry-run sequence, message brevity,
and PowerShell mechanics (its Step 4). Commits are the durable
record; Jamie audits each one before it lands. This supersedes any
hook or workflow that would otherwise auto-commit.

## Rule 8: Jamie runs automated tests

Ask Jamie to run tests; give the commands, what to run, and expected
outcomes ("expect 2 skipped, 5 successful"). She pastes errors back
or confirms.

Rule 8 also covers **manual phase-exit walkthroughs** via
`/exit-test-plan` — distinct from the automated runs above: human-only
verification of interactive behavior at phase transitions, where
Claude authors the plan but never runs the steps.

## Rule 9: Invoke the owning skill instead of inlining its ritual

When Claude would offer a behavior that a skill canonically owns,
invoke (or suggest) that skill instead of performing the behavior
inline. The skill's ownership is the source of truth for the
ritual's exact shape. See `rules/project-rules.md` § "Skills own
rituals" for the principle behind this gate.

- **Offering commit commands → `/wind-down`** (rule 7).
  `/wind-down` owns the commit handoff.
- **End-of-session cues** — "calling it", "off to bed", "done for
  tonight", "that's enough for today", or implicit wind-down
  signals — → suggest `/wind-down` and wait for Jamie's go-ahead.
  `/wind-down` owns the `TODO.txt` rewrite and the doc-coherence
  sweep. Don't run it autonomously.

## Rule 10: Never assume training-time versions are current

Version recall is stale-prone. Before pinning any version into an
artifact, flag it as possibly-stale, offer to look up current
stable/LTS, and let Jamie pick. A version already in a design doc,
REQUIREMENTS, or prior session does **not** bypass this — ask whether
the pin was intentional or a training-time residual. Skills that write
or review design docs carry this explicitly.
