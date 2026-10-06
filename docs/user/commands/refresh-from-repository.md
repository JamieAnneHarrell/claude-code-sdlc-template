# `/refresh-from-repository`

Pull later template improvements into a seeded project — replace the commands,
block-merge the rules and `CLAUDE.md`.

**Type:** reference · **Primary reader:** template adopter

## Synopsis

```
/refresh-from-repository
/refresh-from-repository --refresh-skills-only
/refresh-from-repository --no-claudemd
```

- `--refresh-skills-only` — update only the slash commands; merge no rules. Useful
  for inspecting new refresh logic before it touches your rules.
- `--no-claudemd` — refresh the commands and rules but leave `CLAUDE.md` alone.

## When to run

In a seeded project, whenever you want the latest upstream commands and rules. A
seeded project keeps its own copy and does not track upstream automatically — this
is the pull. Two modes are detected automatically: **public mode** fetches from
the upstream GitHub repository; **source mode** treats a local `cc-template/`
subdirectory as the upstream.

## What it does

A seven-step refresh that wholesale-replaces the commands and block-merges the
rules and `CLAUDE.md`:

1. **Detect mode and confirm upstream.** On the first source-mode run it
   content-inspects `./cc-template/` to confirm it is this template.
2. **Fetch to staging.** Public mode shallow-clones into a throwaway location;
   source mode reads the local subdir. Nothing in your tree changes yet.
3. **Review before applying** (public mode) — an adversarial read of the staged
   commands for anything that doesn't belong, plus a change summary, then an
   explicit go/no-go. Declining leaves your project untouched. There is no flag to
   skip this. Source mode shows a plain change summary instead.
4. **Detect merge-logic drift.** If the upstream refresh logic is newer, it
   installs the commands only and asks you to re-invoke so the new logic runs the
   merge.
5. **Replace the commands** wholesale. With `--refresh-skills-only`, it stops
   here. Otherwise it also replaces the license file
   (`.claude/claude-code-sdlc-template-license.md`) and the output style
   (`.claude/output-styles/claude-code-sdlc-template.md`), and creates
   `.claude/settings.json` if absent or sets only its `outputStyle` key if
   present. Then it adds any template doc skeleton (`docs/design-decisions.md`,
   `docs/open-questions.md`, `docs/documentation-guidance.md`) that is absent —
   never overwriting one you have.
6. **Reconcile the rules and `CLAUDE.md`** block by block, matched by marker id,
   editing in place. It also makes two targeted edits outside the blocks: it
   updates the session-start directive at the top of `CLAUDE.md` to upstream's
   wording, and adds the provenance comment as line 1 of any rules file or
   `CLAUDE.md` that lacks it. See
   [Keeping the template up to date](../keeping-up-to-date.md) for the full
   marker-state model (`template-owned` / `forked` / `removed`) and the one-time
   pre-marker migration.
7. **Report** what changed and what was respected.

## Reads

The local and staged refresh-logic version stamps, `.claude/commands/*.md`,
`.claude/settings.json`, `rules/*.md`, `CLAUDE.md` (block ids and marker states),
and — in source mode — the `cc-template/` subdir.

## Writes / owns

- `.claude/commands/*.md`, the license file, and the output style — wholesale
  replaced.
- `.claude/settings.json` — created when absent; otherwise only the
  `outputStyle` key.
- The doc skeletons — added only when absent.
- The template-owned blocks of `rules/*.md` and `CLAUDE.md` — reconciled in
  place, recording `state=forked` / `state=removed` as you decide — plus the two
  targeted edits (the `CLAUDE.md` session-start directive; the provenance
  comment).

It never touches `ONBOARD-FILL` regions or any other content outside the markers,
and keeps no sidecar state file — the marker states in your files are the only
memory.

## Refuses when

- **Git isn't available** — it points at installing git.
- **Public-mode upstream is unreachable or the clone fails** — it stops before
  changing any file.
- **You decline the step-3 review, or it finds something** — it deletes the
  staging copy and leaves your tree untouched.
- **Source mode's `cc-template/` isn't this template** — it refuses to treat an
  unrelated directory as upstream.
- **An `ONBOARD-FILL` region overlaps a `CC-TEMPLATE-BLOCK`, or a tombstone is
  malformed** — it surfaces the problem rather than guessing.

A newer upstream refresh logic is not a failure — it installs the commands and
asks you to re-invoke.

## Does not do

Run git, push, keep a state file or content hashes, edit `ONBOARD-FILL` or free
regions beyond the two targeted edits, touch any `settings.json` key other than
`outputStyle`, wholesale-overwrite the rules or `CLAUDE.md`, re-add tombstoned blocks,
change `forked` blocks, take a configurable upstream URL, or own a `CLAUDE.md`
status comment.

## See also

[Keeping the template up to date](../keeping-up-to-date.md) ·
[Maintaining the template](../../maintainer/maintaining-the-template.md)
