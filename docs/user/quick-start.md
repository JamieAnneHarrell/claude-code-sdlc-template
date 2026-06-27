# Quick-start: from template to a project in the loop

This is the shortest path from "I have the template" to "my project is set up and
I'm in the working rhythm." Follow the steps in order; each one sets up the next.

For what each command does in depth, see the [command reference](commands/README.md).
For why the workflow is shaped this way, see [The SDLC lifecycle](lifecycle.md).

## Before you begin

You need:

- **Claude Code** — any distribution (IDE extension, desktop app, CLI, or web).
- **A design intake** — the specification onboarding builds from. The
  recommended way to create one is `/product-visioning`, an interactive session
  you run in step 3; if you already have a design document, you'll drop it into
  `docs/design/` instead. Either way, the richer the intake, the richer the plan
  the template generates from it.

## 1. Copy the template into a new directory

Copy the `cc-template/` directory out of the template repository and rename the
copy to your project's name. Copy `cc-template/` itself, not the whole
repository.

```
# PowerShell
Copy-Item -Recurse cc-template C:\path\to\my-project

# bash
cp -r cc-template ~/path/to/my-project
```

The renamed directory *is* your new project. There is no nested
`cc-template/cc-template/` inside it.

## 2. Read the rules before you run anything

Open `rules/` and read each file. The template ships an opinionated collaboration
agreement — ten standing rules and a design philosophy — and those rules shape
every command you are about to run.

Treat them as a starting agreement, not law. If a rule doesn't match how you want
to work, edit it now, before the first command runs. You own these files; editing
them is expected. (Later, [`/refresh-from-repository`](commands/refresh-from-repository.md)
will respect any rule you have made your own — see
[Keeping a project up to date](keeping-up-to-date.md).)

## 3. Open the project in Claude Code and create your design intake

Open the renamed directory in Claude Code, then run:

```
/product-visioning
```

This is the recommended way to start. `/product-visioning` is an interactive
session that works through what you're building and why, then writes the first
PRD (`docs/design/PRD-<slug>-001.md`) — the specification onboarding reads in the
next step. Re-run it until the PRD says what you mean.

**Already have a design document?** Drop it into `docs/design/` instead (markdown
preferred, multiple files are fine) and continue to step 4 — onboarding reads a
dropped design doc just as well as a generated PRD.

## 4. Run `/onboard`

```
/onboard
```

Onboarding reads your intake, asks a handful of setup questions (project name,
language, multi-agent mode, scope), and writes the planning documents:
`REQUIREMENTS.md`, `ARCHITECTURE.md`, `PRODUCT_VISION.md`, `PROJECT_PLAN.md`, and
`CLAUDE_CODE_PROMPTS.md`. When it finishes, your project is configured.

## 5. Run `/design-review`, then `/bootstrap`

```
/design-review
```

This is a sanity check on what onboarding produced: did the requirements,
architecture, and plan come out of your intake cleanly? Mark up the checkpoint it
writes, then re-run `/design-review` to walk the decisions and land it.

```
/bootstrap
```

Bootstrap plans your developer environment and writes the README "Developer
setup" section. Run it once to write the instructions, then again to validate
them. It is the last step before you write code.

## 6. You're in the loop

Your project now settles into a repeating rhythm: write code against the phase
prompts, close each session with [`/wind-down`](commands/wind-down.md), test phase
exits with [`/exit-test-plan`](commands/exit-test-plan.md), and plan the next
movement with [`/product-visioning`](commands/product-visioning.md) when the
current one lands. [The SDLC lifecycle](lifecycle.md) walks the whole rhythm.
