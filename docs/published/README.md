# claude-code-sdlc-template documentation

This folder holds the detailed, audience-facing documentation for the template —
how to adopt it, run it, keep it current, and maintain it. The two READMEs (the
[repository README](../../README.md) and the
[shipping `cc-template/` README](../../cc-template/README.md)) orient and route
here; this is where the depth lives.

## Start here

**If you're adopting the template** — copying it into a new project and running
the workflow:

| Your goal | Read |
|---|---|
| Get a first project set up, fast | [Quick-start](quick-start.md) |
| Understand the workflow and why it's shaped this way | [The SDLC lifecycle](lifecycle.md) |
| Look up exactly what a command does | [Command reference](commands/README.md) |
| Pull later template improvements into your project | [Keeping the template up to date](keeping-up-to-date.md) |
| Recover from a refusal, or spot a workflow drift | [Troubleshooting](troubleshooting.md) |

**If you're maintaining the template** — working on the source-of-truth repository
itself:

| Your goal | Read |
|---|---|
| Change the template safely without breaking an invariant | [Maintaining the template](maintaining-the-template.md) |

## How this relates to the internal docs

The documentation here describes *how to use and maintain* the template. The
project's own internal planning docs — [`REQUIREMENTS.md`](../REQUIREMENTS.md),
[`ARCHITECTURE.md`](../ARCHITECTURE.md), and the rest under [`docs/`](../) —
describe *what the template is and why it's built that way*. The maintainer guide
routes to those for authoritative detail rather than repeating them.
