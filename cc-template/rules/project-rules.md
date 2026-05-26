# Project-Specific Rules

Project-specific scope, dependency allowlist, and defaults live here.
Universal rules (rule 1–9) live in `rules/coding-session-rules.md`.
The general rules above the divider apply to every project; the
section below the divider is populated by `/onboard`.

---

## General project rules (every project)

### Scope discipline

- **MVP is decided during onboarding.** Nothing outside MVP scope
  ships in v0.1.
- **Roadmap is roadmap** — not MVP, not "while we're at it." Items
  flagged as roadmap stay roadmap until promoted explicitly.
- **Ambiguity goes to the docs first, then to Jamie.** Check
  `docs/REQUIREMENTS.md` and `docs/PROJECT_PLAN.md` first; ask Jamie
  if the docs don't answer.

### Dependency justification

When adding a runtime dependency, the commit message must say **why**
in concrete terms.

- **Acceptable:** "ffprobe wrapping needs robust subprocess
  handling," "webrtcvad-wheels provides prebuilt Windows binaries,"
  "click is the project's chosen CLI framework."
- **Unacceptable:** "might be useful," "nicer API," "everyone uses
  it," "saves a few lines."

Dev dependencies (test runner, formatter, linter) follow the same
rule with a lower bar — those choices are usually settled during
onboarding.

### No unsolicited features

Rule 4 in project context: don't add capability that wasn't asked
for. If a session request implies a feature ("can you wire that up
so it also handles X?"), confirm scope before implementing X.

### Commit and branch discipline

- One logical change per commit; ask Jamie whether to bundle or
  split if multiple changes are in flight.
- Subject ≤72 chars, imperative mood ("Add motion detector", not
  "Added"/"Adds").
- Per rule 7: Claude proposes commit commands; Jamie runs them.
- Don't push or tag without explicit instruction.
- Branch naming: `phase-N-<short-description>` for phase work,
  `fix/<short-description>` for post-MVP fixes.

---

## Project-specific scope and constraints

> *Onboarding fills in this section based on Jamie's answers.*

<!-- ONBOARD-FILL: project-scope -->

### MVP scope statements
- (Onboarding adds e.g. "No GUI in MVP", "No ML models in MVP", "Single
  user only in MVP")

### Runtime dependency allowlist
- (Onboarding adds the chosen libraries with one-line rationale each)

### Out-of-scope until further notice
- (Onboarding lists rejected approaches and roadmap items so future
  Claude doesn't re-propose them)

<!-- /ONBOARD-FILL -->
