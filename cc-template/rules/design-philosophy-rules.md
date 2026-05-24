# Design Philosophy Rules

## KISS — Keep It Simple, Silly

Default to the simplest thing that solves the actual need. Smaller
footprint, fewer moving parts, fewer bugs.

The rule isn't "no complexity ever" — real needs sometimes require
real complexity. The rule is: **don't add complexity without
discussing the benefit first.** Simplicity is the default; complexity
has to earn its place.

Before adding a layer, abstraction, dependency, or module, name what
you'd add and why, and state the simpler alternative. Let Jamie pick.
Silently adding complexity is the failure mode.

Watch for: "while we're at it…", "it would be cleaner if…", "this
will be useful later…", "let me make this configurable…", "I'll add
a fallback in case…". Each is a signal to stop and check.

This pairs with rule 4 (no unsolicited design decisions) and
dependency justification. KISS is the default direction; rule 4 is
the gate on changing it.

---

## Progressive Disclosure

Design interfaces that are simple by default and complex by access. The
common case must require zero configuration and no prior knowledge.
Advanced features are always accessible but never mandatory or visually
prominent. When choosing between exposing a control and tucking it
behind a flag, subcommand, or optional kwarg, **tuck it**. Never remove
capability — just don't foreground it.

**Quick-recall token:** *iPhone, not Android. Macintosh, not PC.*

### Applied across surfaces

- **CLI:** positional args + smart defaults for the happy path; flags
  and subcommands gate advanced control. `tool input.mp4` should do the
  right thing; `tool --advanced-flag` exists for the 5% case.
- **Functions:** simple required args; optional kwargs carry advanced
  options. `process(path)` not `process(path, mode, retries, timeout,
  cache_strategy)`.
- **Config:** sane in-code defaults; config file is an optional override,
  never required. The tool runs without a config file.
- **APIs:** required parameters are the minimum to express intent;
  everything else is optional and well-named.
- **UI:** the primary action is one click away. Settings live behind a
  gear, not in the main viewport.

### What this rules out

- "Power-user mode" that hides functionality behind setup. If it's a
  feature, it's discoverable.
- Required configuration before first run. The tool must work with
  zero config.
- Flag soup at the top level of a CLI. Group advanced flags under
  subcommands or `--advanced-...` prefixes.
- "You must read the docs to use this" — first-run output should
  educate.

### When to break the rule

When the operation is destructive or expensive (deleting data,
spending money, sending messages), require an explicit affirmative.
Progressive disclosure is about default behavior, not safety
guardrails.

---

## Other guiding principles

### Don't add capability you don't need

A bug fix doesn't need surrounding cleanup. A one-shot operation
doesn't need a helper. Three similar lines is better than a premature
abstraction. No half-finished implementations either.

### Don't validate scenarios that can't happen

Trust internal code and framework guarantees. Only validate at system
boundaries (user input, external APIs). Don't use feature flags or
backwards-compatibility shims when you can just change the code.

### Default to no comments

Only add a comment when the WHY is non-obvious: a hidden constraint, a
subtle invariant, a workaround for a specific bug, behavior that would
surprise a reader. If removing the comment wouldn't confuse a future
reader, don't write it.

Don't explain WHAT the code does — well-named identifiers do that.
Don't reference the current task, fix, or callers ("used by X", "added
for the Y flow", "handles the case from issue #123") — those belong in
the PR description.
