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

- **No language runtime.** The deliverable is markdown files. No
  interpreter, compiler, package manager, or build step is part
  of MVP.
- **No GUI in MVP.** No web service, no API, no graphical
  interface. The "product" is the file tree that consumers copy
  and the commands they invoke inside Claude Code.
- **No ML models in MVP.** No statistical or learned components
  anywhere. Everything is deterministic markdown and the
  commands' deterministic behavior.
- **Cross-platform.** Windows 11 is primary; Linux and macOS are
  first-class supported. Anything that works on one but not the
  others is a bug.
- **Self-modifying discipline.** The source-of-truth project is
  itself a downstream consumer of its own template. Improvements
  exercise on the source before they ship to consumers.

### Runtime dependency allowlist

None. The project has no language runtime and therefore no
runtime dependencies. If a future phase introduces a dependency
(e.g. Phase 3 regression-test automation might introduce a small
PowerShell or shell script), it goes through the standard
dependency-justification rule in the General rules section
above.

### Out-of-scope until further notice

- **Markdown linting / style enforcement automation.** Style
  consistency is human-reviewed; not worth tooling investment
  for a content-only repo.
- **A landing page or product website.** Distribution is
  git-based.
- **Translations into non-English locales.** English-only for
  the foreseeable future.
- **Bundling as an installable package** (npm, pip, etc.).
  Distribution remains "copy the `cc-template/` subdirectory."
- **Push-model upstream → downstream updates.** Rejected during
  the source/dist restructure (see
  [`docs/design-decisions.md`](../docs/design-decisions.md)
  "Downstream updates use a pull model, not push"). Pull-only via
  Phase 2's `/refresh-from-repository`.
- **Configurable upstream URL** (for `/refresh-from-repository`).
  The URL is baked in to
  `github.com/JamieAnneHarrell/claude-code-sdlc-template`; not
  user-configurable in v1.
- **A fourth configuration command.** The three configuration
  commands already cover the lifecycle. Adding a fourth needs
  strong justification (per root `CLAUDE.md` "Design principles
  for template changes").
- **Per-command status comments for the recurring commands.** The
  `/design-review` and `/exit-test-plan` lifecycles are tracked
  via per-file frontmatter status on their artifacts, not via a
  fourth status comment in `CLAUDE.md`. This is a load-bearing
  invariant (see root `CLAUDE.md`).

<!-- /ONBOARD-FILL -->
