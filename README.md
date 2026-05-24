# cc-template (source-of-truth project)

This is the **source-of-truth project** for the `cc-template` seed.
The directory you copy when starting a new project is the
[`cc-template/`](cc-template/) subdirectory inside this repo — not
this repo itself.

> 🛠️ Working ON this template? Read [`CLAUDE.md`](CLAUDE.md) first.
> If you've copied `cc-template/` into a new project, read
> [`cc-template/CLAUDE.md`](cc-template/CLAUDE.md) there.

## Project shape

```
jah-template-project/                  ← THIS REPO (source-of-truth)
├── CLAUDE.md                         ← source-side session-start (this file describes
│                                       the project; CLAUDE.md describes how to work on it)
├── README.md                         ← you are here
├── TODO.txt                          ← this project's session handoff (gitignored)
├── .gitignore
├── rules/                            ← collaboration rules (shared between source and dist)
├── docs/                             ← this project's own design docs
│   ├── design-decisions.md
│   ├── open-questions.md
│   └── design/                       ← /design-review checkpoints on ourselves
├── .claude/commands/                 ← live commands this project uses on itself
└── cc-template/                      ← THE DISTRIBUTABLE (what gets copied to seed new projects)
    ├── CLAUDE.md                     ← new-project session-start
    ├── README.md                     ← shipping README
    ├── TODO.txt                      ← placeholder handoff
    ├── .gitignore
    ├── rules/                        ← shipping rules with ONBOARD-FILL placeholders
    ├── docs/                         ← shipping doc stubs
    └── .claude/                      ← all 6 shipping commands + settings example
```

## Seeding a new project

Copy the **dist subdirectory** (not the whole repo):

```
Copy-Item -Recurse cc-template\cc-template C:\path\to\new-project
cd C:\path\to\new-project
# Drop a design doc into docs/design/, then run:
#   /onboard
```

(Or use the file explorer — copy `cc-template/cc-template/`, rename
the destination, open in Claude Code, run `/onboard`.)

## Working on the template itself

Edits to shipping content go in [`cc-template/`](cc-template/). Edits
to project-only files (this README, root `CLAUDE.md`, source-only
docs, root rules) stay outside that subdirectory. The convention:

- **Universal content** (the 9 rules, design philosophy, command
  files, etc.) is kept identical in both locations. Edit in
  [`cc-template/`](cc-template/) first; copy the change up to root if
  it's a command or rules file this project itself uses.
- **Source-only content** (this README, root `CLAUDE.md`, real
  REQUIREMENTS / ARCHITECTURE / PROJECT_PLAN entries, design-decisions
  entries) lives only at root and never copies into the dist.

`/design-review`, `/exit-test-plan`, and `/wind-down` are the
recurring commands this project uses on its own docs. The
configuration commands (`/onboard`, `/bootstrap`, `/deployment-plan`)
only make sense for *new* projects seeded from the dist.

## Why this shape

The template grew complex enough (iterative `/design-review` and
`/exit-test-plan` lifecycles, ten-plus load-bearing invariants) that
copy-paste-without-history became a regression risk. This restructure
preserves the copy-paste workflow (consumers still copy a directory
called `cc-template`) while putting the source under git so changes
are auditable and revertible.

See [`docs/design-decisions.md`](docs/design-decisions.md) for the
full rationale and the alternatives that were rejected.

## Status

This restructure is itself the project's Phase 0. Git initialization
and the public-repo distribution channel (for `/refresh-from-repository`,
the planned pull-model self-update command shipped in the dist) come
in later phases. See [`docs/open-questions.md`](docs/open-questions.md)
for what's still being decided.
