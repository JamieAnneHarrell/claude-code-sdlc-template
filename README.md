# claude-code-sdlc-template

`claude-code-sdlc-template` is a self-modifying project seed for
new software projects driven by Claude Code. The distributable
subdirectory [`cc-template/`](cc-template/) ships six slash
commands — `/onboard`, `/bootstrap`, `/deployment-plan`,
`/design-review`, `/exit-test-plan`, `/wind-down` — plus curated
collaboration rules. Consumers copy `cc-template/` into a new
project, drop a design doc into `docs/design/`, and run the
configuration ritual. This repository is itself a downstream
consumer of its own template, so improvements are exercised on
the source before they ship.

> 🛠️ Working ON this template? Read [`CLAUDE.md`](CLAUDE.md) first.
> If you've copied `cc-template/` into a new project, read
> [`cc-template/CLAUDE.md`](cc-template/CLAUDE.md) there.

## Project shape

```
claude-code-sdlc-template/             ← THIS REPO (source-of-truth)
├── CLAUDE.md                         ← source-side session-start (this file describes
│                                       the project; CLAUDE.md describes how to work on it)
├── README.md                         ← you are here
├── TODO.txt                          ← this project's session handoff (gitignored)
├── LICENSE                           ← CC BY-NC-ND 4.0 (covers source-only content)
├── .gitignore
├── rules/                            ← collaboration rules (mirrored to cc-template/rules/)
├── docs/                             ← this project's own design docs
│   ├── REQUIREMENTS.md
│   ├── ARCHITECTURE.md
│   ├── PROJECT_PLAN.md
│   ├── CLAUDE_CODE_PROMPTS.md
│   ├── design-decisions.md
│   ├── open-questions.md
│   └── design/                       ← design intake + /design-review checkpoints
├── .claude/commands/                 ← live commands this project uses on itself
└── cc-template/                      ← THE DISTRIBUTABLE (what gets copied to seed new projects)
    ├── CLAUDE.md                     ← new-project session-start
    ├── README.md                     ← shipping README
    ├── TODO.txt                      ← placeholder handoff
    ├── LICENSE                       ← MIT (covers everything in this subtree)
    ├── .gitignore
    ├── rules/                        ← shipping rules with ONBOARD-FILL placeholders
    └── .claude/                      ← all 6 shipping commands + settings example
```

## Seeding a new project

Copy the **dist subdirectory** (not the whole repo):

```
Copy-Item -Recurse cc-template C:\path\to\new-project
cd C:\path\to\new-project
# Drop a design doc into docs/design/, then run:
#   /onboard
```

(Or use the file explorer — copy `cc-template/`, rename the
destination, open in Claude Code, run `/onboard`.)

## Working on the template itself

Edits to shipping content go in [`cc-template/`](cc-template/).
Edits to project-only files (this README, root `CLAUDE.md`,
source-only docs, root rules) stay outside that subdirectory.
The convention:

- **Universal content** (the 10 rules, design philosophy, command
  files, etc.) is kept identical in both locations. Edit in
  [`cc-template/`](cc-template/) first; copy the change up to
  root if it's a command or rules file this project itself uses.
- **Source-only content** (this README, root `CLAUDE.md`, real
  REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN entries,
  design-decisions entries) lives only at root and never copies
  into the dist.

`/design-review`, `/exit-test-plan`, and `/wind-down` are the
recurring commands this project uses on its own docs. The
configuration commands (`/onboard`, `/bootstrap`,
`/deployment-plan`) only make sense for *new* projects seeded
from the dist — except for this project, which ran them on
itself as Phase 1 (see [docs/PROJECT_PLAN.md](docs/PROJECT_PLAN.md)).

## Why this shape

The template grew complex enough (iterative `/design-review` and
`/exit-test-plan` lifecycles, ten-plus load-bearing invariants)
that copy-paste-without-history became a regression risk. This
restructure preserves the copy-paste workflow (consumers still
copy a directory called `cc-template`) while putting the source
under git so changes are auditable and revertible.

See [`docs/design-decisions.md`](docs/design-decisions.md) for
the full rationale and the alternatives that were rejected.

## Developer setup

Working on this template requires almost nothing — except Claude
Code. The deliverable is markdown files only; there is no language
runtime, no build, no test runner.

This project is **self-consuming**: the source repo uses its own
[`cc-template/`](cc-template/) on itself, so improvements get
exercised here before they ship.

### Prerequisites

**For everyone:**
- **Claude Code** — any distribution (IDE extension, desktop app,
  CLI, or web). No version pin. Maintainers are recommended to use
  **Opus 4.7** (current latest as of 2026-05-24) or the most recent
  high-performance Claude model available.

**Maintainers / contributors** also need:
- **Git** — to clone this repository and submit changes back.
- A markdown-aware editor. Jamie uses VS Code on Windows 11 with
  the Claude Code extension and a PowerShell terminal; the project
  works on macOS and Linux with any shell.

**Consumers seeding a new project** only need the `cc-template/`
subdirectory — clone this repository and copy that subdirectory,
or download a zip and extract it. Distribution mechanism is in
the process of being finalized (handled by `/deployment-plan`).

Cloning the whole repository with git is primarily a
maintainer / contributor activity; consumers don't need to.

### Verify your setup

Maintainers only:

```
git --version
```

Consumers downloading a zip of `cc-template/` don't strictly need
git on PATH.

### Bootstrap

Maintainer / contributor flow:

```
git clone https://github.com/JamieAnneHarrell/claude-code-sdlc-template
```

Open the repo in your editor and edit markdown. That's the entire
bootstrap.

### Run locally

N/A — there is no application to run. Editing markdown files in
your editor IS the development loop.

### Test

No automated test suite. Validation is:

1. Re-read the touched command and rules files for self-consistency.
2. Mental dry-run of the affected stage logic against the
   load-bearing invariants in [`CLAUDE.md`](CLAUDE.md).
3. Real-world use of the template on a new project — that's the
   end-to-end test.

### Secrets and config

None. No `.env` file, no API keys, no database credentials,
nothing committed contains secrets.

## Deployment

Deployment plan is generated by `/deployment-plan` (deferrable;
run when test/prod planning is needed).

## Usage

See [`docs/PROJECT_PLAN.md`](docs/PROJECT_PLAN.md) for the phase
queue and [`docs/CLAUDE_CODE_PROMPTS.md`](docs/CLAUDE_CODE_PROMPTS.md)
for the prompt body to paste into Claude Code at each phase
start.

## License

This repo is **dual-licensed** because of the recursive nature of
the project (this project uses its own `cc-template/` template on
itself).

- The **distributable** at [`cc-template/`](cc-template/) ships
  under **MIT** — see [`cc-template/LICENSE`](cc-template/LICENSE).
  Anyone seeding a new project from `cc-template/` can copy it,
  modify it, ship it, and sell the resulting product. Attribution
  required (retain the MIT LICENSE text).
- The **root `.claude/commands/`** and **root `rules/`** carry the
  same MIT license because they ARE this project's working copy
  of cc-template content — we use cc-template on ourselves the
  same way any consumer would, so those files inherit cc-template's
  MIT terms.
- **Everything else at repo root** — the source-only project
  docs in [`docs/`](docs/), this README, root
  [`CLAUDE.md`](CLAUDE.md), [`docs/design/`](docs/design/),
  [`docs/design-decisions.md`](docs/design-decisions.md), and
  [`docs/open-questions.md`](docs/open-questions.md) — ships
  under **CC BY-NC-ND 4.0**. See [`LICENSE`](LICENSE). These are
  this project's own management docs; they're reference material
  only, not meant to be redistributed.

## Reference

For development context, see [`CLAUDE.md`](CLAUDE.md) and
[`docs/`](docs/).
