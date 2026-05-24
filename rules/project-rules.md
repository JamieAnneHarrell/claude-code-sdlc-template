# Project-Specific Rules

These rules apply to **this project** specifically. The universal rules
(rule 1–9) live in `rules/coding-session-rules.md` and apply
everywhere. This file is where the project's MVP scope, dependency
allowlist, and project-specific defaults live.

`/onboard` populates the project-specific sections at the bottom of
this file based on Jamie's answers and the design intake. The general
rules above the divider apply to every project.

---

## General project rules (every project)

### Scope discipline

- **MVP is decided during onboarding.** Nothing outside MVP scope ships
  in v0.1.
- **Roadmap is roadmap.** Not MVP. Not "while we're at it." Items
  flagged as roadmap stay roadmap until promoted explicitly.
- **Ambiguity goes to the docs first, then to Jamie.** If a session
  request is ambiguous about whether something is in-scope, check
  `docs/REQUIREMENTS.md` and `docs/PROJECT_PLAN.md` first; ask Jamie if
  the docs don't answer.

### Dependency justification

When adding a runtime dependency, the commit message must say **why**
in concrete terms.

Acceptable: "ffprobe wrapping needs robust subprocess handling,"
"webrtcvad-wheels provides prebuilt Windows binaries," "click is the
project's chosen CLI framework."

Unacceptable: "might be useful," "nicer API," "everyone uses it,"
"saves a few lines."

Dev dependencies (test runner, formatter, linter) follow the same rule
but the bar is lower — those choices are usually settled during
onboarding.

### No unsolicited features

Restating rule 4 in project context: do not add capability that wasn't
asked for. If a session request implies a feature ("can you wire that
up so it also handles X?"), confirm scope before implementing X. The
design docs and project plan are the contract.

### Commit and branch discipline

- One logical change per commit. If multiple unrelated changes are in
  flight, ask Jamie whether to bundle or split.
- Commit message subject under 72 chars, imperative mood ("Add motion
  detector" not "Added motion detector" or "Adds motion detector").
- Per rule 7: Claude proposes commit commands; Jamie runs them.
- Do not push without explicit instruction.
- Do not tag releases without explicit instruction.
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
