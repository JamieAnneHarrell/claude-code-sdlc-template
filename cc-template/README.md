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
   template is meant to be customized per project. The universal rule
   content sits inside `<!-- CC-TEMPLATE-BLOCK: <id> -->` markers. Edit
   in place freely — when you later run `/refresh-from-repository` it
   asks, per changed block, whether to take the upstream version or
   keep yours (keeping yours marks the block as yours so it's never
   asked about again). You don't have to do anything special now to
   protect an edit. See "Keeping a project up to date" below.

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

## Keeping a project up to date

Once a project is seeded, it keeps its own copy of the commands and
rules — it does **not** track the upstream template. To pull later
improvements, run, in the project:

```
/refresh-from-repository
```

Refresh fetches the latest upstream into a throwaway staging area,
**shows you what would change and asks before touching anything**, then
— on your go-ahead — **replaces** the shipped slash commands with the
upstream versions and **block-merges** the universal rules and the
template-owned sections of `CLAUDE.md` against what you have now.

**It reviews the download before applying.** The slash commands are
instructions Claude will later execute, so pulling them from the
internet is a trust boundary. Before changing a single file in your
project, refresh reads the fetched content — commands first — looking
for anything that doesn't belong (injected directives, attempts to
exfiltrate data or disable safeguards, surprising shell), summarizes
what would change, and asks you to approve. If you decline, the staging
copy is discarded and your project is left exactly as it was. There is
no flag to skip this; the review is quick.

Your customizations are safe by design:

- Content inside `<!-- ONBOARD-FILL: ... -->` blocks (your
  project-specific scope, environment, tooling) is never touched.
- Content **outside** the `<!-- CC-TEMPLATE-BLOCK: ... -->` markers is
  yours forever — refresh ignores it.
- If you edited a rule *inside* a template block, refresh asks, one
  block at a time, whether to **take upstream**, **keep mine**, or
  **hand-merge**. It never silently overwrites an edit. Choosing "keep
  mine" marks that block as **yours** (`state=forked` in the marker) so
  refresh leaves it alone from then on — it won't ask again.
- If a template block is gone, refresh asks once whether you removed it
  on purpose; if so it writes a small tombstone
  (`state=removed`) and never re-adds the block.

This is all the memory refresh needs — the marker states live right in
your rules / `CLAUDE.md` files. There is no separate machine-managed
state file to commit or worry about.

**Moving a rule out is the same as owning it.** If you lift a template
rule into your own section or file, refresh treats it as removed from
the template (it works strictly within its own marked blocks and does
not go hunting through your other files for it). Your relocated copy is
yours; it simply won't receive upstream updates. If you want a rule to
keep tracking upstream, leave it in its `CC-TEMPLATE-BLOCK`.

A project onboarded before the marker system existed gets a one-time
migration on its first run: refresh inserts the markers for you,
aligning to upstream, and asks about any section where your content has
diverged.

Two optional flags:

- `/refresh-from-repository --refresh-skills-only` — update only the
  slash commands, merge nothing. Useful to inspect the new refresh
  logic before letting it touch your rules.
- `/refresh-from-repository --no-claudemd` — refresh commands and
  rules but leave `CLAUDE.md` alone. For heavily-customized
  `CLAUDE.md` files.

**Pinning to a specific version (vendored lock-in).** If you need to
pin to an exact upstream version — for audit or reproducibility — copy
the `cc-template/` directory from the upstream commit you want into
your own repo as a subdirectory. With a `cc-template/` subdir present,
`/refresh-from-repository` runs in **source mode**: it treats that
local subdir as the upstream and syncs it to your project root, with
the same block-merge and ask-once behavior. Because that upstream is
your own local copy (already under your control), source mode shows a
plain change summary instead of the security review. Upgrading the pin
is a deliberate action you take (re-vendor a newer `cc-template/`); the
routine sync from subdir to root stays stable and auditable.

### Installing the command into a project that doesn't have it yet

A self-updating command can't install itself — a project seeded before
`/refresh-from-repository` existed has no command to run. Bootstrap it
once by hand (this is exactly a `--refresh-skills-only` pass done
manually):

1. **Get the upstream command files.** Clone or download
   `github.com/JamieAnneHarrell/claude-code-sdlc-template` (or, for a
   vendored/source-mode setup, use your local `cc-template/`
   subdirectory).
2. **Copy the command file(s)** from the upstream
   `cc-template/.claude/commands/` into your project's
   `.claude/commands/`. At minimum copy `refresh-from-repository.md`;
   copying all of them is the same as a skills-only refresh and gets
   you the latest version of every command at once.
3. **Run `/refresh-from-repository`** in your project. With the command
   now present, it performs the one-time pre-marker migration (inserts
   `CC-TEMPLATE-BLOCK` markers, aligning to upstream and asking about
   any diverged section) and block-merges the rest. From then on,
   refresh keeps itself current like any other command.

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
│   │   ├── bootstrap.md           ← /bootstrap slash command
│   │   ├── deployment-plan.md     ← /deployment-plan slash command
│   │   ├── design-review.md       ← /design-review slash command
│   │   ├── exit-test-plan.md      ← /exit-test-plan slash command
│   │   ├── wind-down.md           ← /wind-down slash command
│   │   └── refresh-from-repository.md ← /refresh-from-repository slash command
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
