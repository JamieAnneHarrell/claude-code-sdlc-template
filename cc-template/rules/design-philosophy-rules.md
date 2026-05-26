# Design Philosophy Rules

## KISS — Keep It Simple, Silly

Default to the simplest thing that solves the actual need. The rule
isn't "no complexity ever" — real needs sometimes require real
complexity. It's: **don't add complexity without discussing the
benefit first.** Simplicity is the default; complexity has to earn
its place.

Before adding a layer, abstraction, dependency, or module, name what
you'd add and why, and state the simpler alternative **in the same
message**. Let Jamie pick. Silently adding complexity is the failure
mode — "pause and check" is not enough; the simpler alternative must
be visible to Jamie at decision time, not considered internally and
discarded.

Scope-expansion signals to stop and check: "while we're at it…",
"it would be cleaner if…", "this will be useful later…", "let me
make this configurable…", "I'll add a fallback in case…".

Pairs with rule 4 (no unsolicited design decisions). KISS is the
default direction; rule 4 is the gate on changing it.

---

## Progressive Disclosure

Simple by default, complex by access. The common case requires zero
configuration and no prior knowledge. Advanced features are always
accessible but never mandatory or visually prominent. When choosing
between exposing a control and tucking it behind a flag, subcommand,
or optional kwarg, **tuck it**. Never remove capability — just don't
foreground it.

**Quick-recall token:** *iPhone, not Android. Macintosh, not PC.*

### Applied across surfaces

- **CLI:** positional args + smart defaults for the happy path;
  `tool input.mp4` does the right thing, `--advanced-flag` exists
  for the 5% case.
- **Functions:** simple required args; optional kwargs carry advanced
  options. `process(path)`, not `process(path, mode, retries, ...)`.
- **Config:** sane in-code defaults; config file is an optional
  override, never required.
- **APIs:** required params are the minimum to express intent;
  everything else is optional and well-named.
- **UI:** primary action one click away; settings behind a gear.

### What this rules out

- "Power-user mode" hiding functionality behind setup — if it's a
  feature, it's discoverable.
- Required configuration before first run.
- Flag soup at the top level of a CLI — group advanced flags under
  subcommands or `--advanced-...` prefixes.
- "You must read the docs to use this" — first-run output educates.

### When to break the rule

Destructive or expensive operations (deleting data, spending money,
sending messages) require an explicit affirmative. Progressive
disclosure governs default behavior, not safety guardrails.

---

## Other guiding principles

### Don't add capability you don't need

Bug fixes don't need surrounding cleanup. One-shot operations don't
need helpers. Three similar lines is better than a premature
abstraction. No half-finished implementations.

### Don't validate scenarios that can't happen

Trust internal code and framework guarantees. Validate only at system
boundaries (user input, external APIs). Don't use feature flags or
backwards-compatibility shims when you can just change the code.

### Default to no comments

Add a comment only when the WHY is non-obvious: a hidden constraint,
subtle invariant, workaround for a specific bug, surprising behavior.
If removing the comment wouldn't confuse a future reader, don't write
it.

Don't explain WHAT the code does — well-named identifiers do that.
Don't reference current task, fix, or callers ("used by X", "added
for the Y flow") — that belongs in the PR description.
