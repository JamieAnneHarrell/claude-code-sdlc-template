# Keeping the template up to date

A project seeded from this template keeps its **own copy** of the commands and
rules — it does not track the upstream template automatically. When the template
improves, [`/refresh-from-repository`](commands/refresh-from-repository.md) pulls
those improvements into your project. This page is about updating that template
*machinery* — the slash commands and the collaboration rules — not your project's
own product code.

## When and why to run it

Run it in your project whenever you want later template improvements:

```
/refresh-from-repository
```

Refresh fetches the latest upstream into a throwaway staging area, **shows you
what would change and asks before touching anything**, then — on your go-ahead —
**replaces** the slash commands with the upstream versions and **block-merges** the
universal rules and the template-owned sections of `CLAUDE.md` against what you
have now.

It reviews the download before applying it. The slash commands are instructions
Claude will later execute, so pulling them from the internet is a trust boundary.
Refresh reads the fetched content first, looking for anything that doesn't belong,
summarizes what would change, and asks you to approve. Decline, and your project
is left exactly as it was. There is no flag to skip this review.

## What it touches, and what it never touches

Your customizations are safe by design:

- Content inside `<!-- ONBOARD-FILL: ... -->` blocks — your project-specific
  scope, environment, and tooling — is never touched.
- Content **outside** the `<!-- CC-TEMPLATE-BLOCK: ... -->` markers is yours
  forever; refresh ignores it.

## The marker-state model

Each template-owned region of a rules file or `CLAUDE.md` is wrapped in a
`CC-TEMPLATE-BLOCK` marker with a stable id. Each block carries one of three
states, and that state — recorded right in your files — is the only memory refresh
keeps. There is no separate state file to commit or worry about.

| State | Meaning | What refresh does |
|---|---|---|
| `template-owned` | The default — the block tracks upstream. | Updates it to match upstream. |
| `forked` | You took ownership of this block. | Leaves it alone; never asks again. |
| `removed` | A tombstone — you deleted the block on purpose. | Respects it; never re-adds the block. |

When refresh finds a template-owned block you have edited, it asks once — per
block — whether to **take upstream**, **keep mine**, or **hand-merge**. Choosing
"keep mine" marks the block `forked` so it is never asked about again. When a block
is gone, it asks once whether you removed it on purpose; if so, it writes a
`removed` tombstone. Moving a rule out into your own section counts as removing it:
your relocated copy is yours and simply stops receiving upstream updates.

A project onboarded before this marker system existed gets a one-time migration on
its first refresh — refresh inserts the markers for you, aligning to upstream, and
asks about any section that has diverged.

## The two flags

- `/refresh-from-repository --refresh-skills-only` — update only the slash
  commands and merge nothing. Useful for inspecting new refresh logic before
  letting it touch your rules.
- `/refresh-from-repository --no-claudemd` — refresh the commands and rules but
  leave `CLAUDE.md` alone. For a heavily-customized `CLAUDE.md`.

## Pinning to a specific version (vendored lock-in)

To pin to an exact upstream version for audit or reproducibility, copy the
`cc-template/` directory from the upstream commit you want into your own repository
as a subdirectory. With a `cc-template/` subdir present,
`/refresh-from-repository` runs in **source mode**: it treats that local subdir as
the upstream and syncs it to your project root, with the same block-merge and
ask-once behavior. Because that upstream is your own local copy, source mode shows
a plain change summary instead of the security review. Upgrading the pin is a
deliberate act — you re-vendor a newer `cc-template/`.

## Installing the command where it doesn't exist yet

A self-updating command can't install itself: a project seeded before
`/refresh-from-repository` existed has no command to run. Bootstrap it once by
hand — this is exactly a skills-only refresh done manually:

1. Clone or download the upstream repository (or use your local `cc-template/`
   subdirectory).
2. Copy the command file(s) from the upstream `cc-template/.claude/commands/` into
   your project's `.claude/commands/`. At minimum copy `refresh-from-repository.md`;
   copying all of them gets you every command's latest version at once.
3. Run `/refresh-from-repository`. With the command now present, it performs the
   one-time pre-marker migration and block-merges the rest. From then on, refresh
   keeps itself current like any other command.
