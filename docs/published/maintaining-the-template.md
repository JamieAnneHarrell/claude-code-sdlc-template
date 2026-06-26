# Maintaining the template

This guide is for someone working on the **source-of-truth repository** itself —
changing the commands, the rules, or the structure. It orients you and shows how
to make a change without breaking something downstream. For the authoritative
detail behind each topic, it routes you to the internal docs rather than repeating
them.

If you are *adopting* the template rather than maintaining it, you want the
[Quick-start](quick-start.md) and the [command reference](commands/README.md)
instead.

## The source/dist split

The repository has two physical surfaces in one git repository:

- **Source-of-truth at the repository root** — this project's own planning docs,
  rules, and `.claude/commands/`, so the source can run the commands on its own
  docs.
- **The distributable at `cc-template/`** — the subdirectory consumers copy to
  seed a new project.

The source-of-truth project is itself a **downstream consumer of its own
template**: improvements are exercised on the source before they ship. The full
rationale and the rejected alternatives are in
[`ARCHITECTURE.md`](../ARCHITECTURE.md) and `docs/design-decisions.md`.

## Duplication discipline

Universal content — the ten rules, the design philosophy, the command files — is
kept **identical** between the root and `cc-template/`. That duplication is
deliberate (the root project consumes its own template) and it creates drift risk.
The discipline that manages it: **edit in `cc-template/` first, then copy the
change up to the root** if it's a command or rules file this project itself uses.

## The two marker systems

Two HTML-comment marker systems carry the reconciliation contract. They are
inverses of each other:

- `<!-- ONBOARD-FILL: <id> -->` wraps **consumer-owned** regions that a refresh
  must never touch (filled by [`/onboard`](commands/onboard.md) and
  [`/bootstrap`](commands/bootstrap.md)).
- `<!-- CC-TEMPLATE-BLOCK: <id> -->` wraps **template-owned** regions that
  [`/refresh-from-repository`](commands/refresh-from-repository.md) keeps in sync,
  each carrying a `template-owned` / `forked` / `removed` state.

The marker syntax and the state encoding are load-bearing — stage detection and
the refresh merge parse them literally. [`ARCHITECTURE.md`](../ARCHITECTURE.md) and
`REQUIREMENTS.md` (NFR-4) pin the exact strings.

## The load-bearing invariant chain

Several conventions are parsed literally by the commands: the status-comment
names, the stage-detection placeholder strings, the zero-padded filename patterns,
and the marker encodings. Changing one without auditing the whole chain causes
silent regressions. The root [`CLAUDE.md`](../../CLAUDE.md) carries the
"Load-bearing invariants — do not break" section; read it before you touch a
command, and when you change an invariant, audit every command and file the chain
names.

## How to change the template safely

1. Make the edit in `cc-template/` first.
2. Audit the invariant chain in `CLAUDE.md` for anything your change touches —
   status comments, placeholder strings, filename pad widths, marker encodings.
3. Copy the change up to the root if it's a command or rules file the source
   itself uses.
4. **Dogfood it.** Run the changed command on this project's own docs before it
   ships. The source repository runs its own commands, so a real exercise is
   always available.

## Keep the bar high for new commands

The template ships a deliberately small command set. **Don't add another command**
without strong justification — each recurring command earned its place by first
proving itself on a live project. Hold template changes to the same KISS and
progressive-disclosure standard the rules describe: simple by default, complexity
only when it earns its place. The design philosophy is in
[`rules/design-philosophy-rules.md`](../../rules/design-philosophy-rules.md).
