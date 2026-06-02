# cc-template

Template seed for new `ds-*` projects. Provides a working
collaboration agreement (the 10 rules), design-philosophy guidance,
slash commands for onboarding and wind-down, and the directory
structure each new project starts from.

## How to use

These steps are also seeded into `TODO.txt` in the new project as a
walkable checklist and session handoff — follow them in order:

1. **Copy the template** to a new project directory:
   ```
   cp -r cc-template ds-<project-name>
   ```
   (or use the file explorer — copy the folder, rename it.)

2. **Drop a design doc** into `docs/design/`. Markdown is preferred.
   Multiple files are fine. A richer design produces a richer
   generated PROJECT_PLAN; a one-paragraph design produces a thin
   one. See `docs/design/README.md` for what good looks like.

3. **Review the rules.** Read each `rules/*.md` file and make sure it
   matches your own collaboration philosophy — edit anything you
   disagree with. The rules are a starting agreement, not law; the
   template is meant to be customized per project. (Once
   `/refresh-from-repository` ships in Phase 2.1, the universal rule
   content will sit inside `CC-TEMPLATE-BLOCK` markers; you'll be able
   to lift the parts you've customized out of those markers so refresh
   leaves your edits alone.)

4. **Open the new directory in Claude Code** and run:
   ```
   /onboard
   ```
   Onboarding reads the design intake, asks a handful of setup
   questions (project name, language, multi-agent mode, scope
   statements), and generates the rest of the project documents.

5. **Run `/bootstrap`** in the next session to plan the developer
   environment (shell, tooling, dev secrets) and write the README
   Developer setup section. It's a hard prerequisite to Phase 0.

6. **Run Phase 0** by pasting Prompt 0 from
   `docs/CLAUDE_CODE_PROMPTS.md`. This sets up the language tooling,
   CI workflow, and project skeleton.

## What the template ships with

```
cc-template/
├── CLAUDE.md                      ← short index, status flag
├── README.md                      ← this file
├── TODO.txt                       ← session handoff (gitignored in new projects)
├── .gitignore                     ← language-agnostic baseline
├── .claude/
│   ├── commands/
│   │   ├── onboard.md             ← /onboard slash command
│   │   └── wind-down.md           ← /wind-down slash command
│   ├── settings.local.json.example
│   └── settings.local.json.example.README.md
├── rules/                         ← collaboration rules, loaded on demand
│   ├── coding-session-rules.md    (the 10 rules)
│   ├── design-philosophy-rules.md (progressive disclosure)
│   ├── project-rules.md           (scope discipline; rewritten at onboarding)
│   ├── testing-rules.md           (test discipline)
│   ├── environment-rules.md       (cross-platform, shells)
│   └── multi-agent-rules.md       (placeholder; rewritten at onboarding)
└── docs/
    ├── design/                    ← drop your design doc(s) here
    │   └── README.md
    ├── design-decisions.md        ← starts as a header skeleton
    └── open-questions.md          ← starts as a header skeleton
```

After onboarding runs, `docs/` also contains `REQUIREMENTS.md`,
`ARCHITECTURE.md`, `PROJECT_PLAN.md`, `CLAUDE_CODE_PROMPTS.md`, and
the rules files have project-specific sections appended.

## What's pre-decided vs. what onboarding asks

**Pre-decided** (apply to every project, baked into the rules files):
- The 10 collaboration rules (commit handoffs are owned by `/wind-down`).
- Progressive-disclosure design philosophy.
- Cross-platform discipline (Windows primary, Linux/macOS supported).
- Logging over print. Structured logging in non-trivial code.
- Tests live behind a marker if they need real binaries / data /
  network; default test runs are unit-only.

**Asked at onboarding:**
- Project name, slug, description, license.
- Primary language and tooling (test runner, formatter, linter).
- Multi-agent mode (explore-only / explore-plus-plan /
  parallel-worktrees).
- Scope: GUI in MVP? ML in MVP?
- Runtime dependency allowlist with rationale.
- Git: existing repo / fresh init / no git.

## Reading order for a configured project

1. `CLAUDE.md` (short index).
2. `TODO.txt` (gitignored; session handoff).
3. `docs/PROJECT_PLAN.md`.
4. `docs/CLAUDE_CODE_PROMPTS.md` for the current phase.
5. The `rules/` files relevant to whatever the session is doing.

## License

The template itself is MIT. Each project generated from the template
gets its own LICENSE — onboarding writes one based on Jamie's choice
(MIT default).
