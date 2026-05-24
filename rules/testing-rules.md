# Testing Rules

Language-neutral test discipline. `/onboard` appends project-specific
tooling (test runner, marker syntax, command lines) to the bottom of
this file when configuring a new project.

---

## Categories

### Unit tests
Pure logic with no external dependencies. Must run with no filesystem
fixtures, no network, no databases, no real subprocess calls. These
form the bulk of the test suite and run on every commit, every CI
build, every machine.

### Integration tests
Tests that exercise external dependencies — real binaries (ffmpeg,
ffprobe), real databases, real network calls, real video files, real
hardware. These live behind a test-runner marker and are **skipped by
default** in CI and in routine local runs.

Integration tests run when explicitly requested, typically before a
release or when a change touches the boundary code.

### Test fixture data
Live in a designated `tests/fixtures/` (or equivalent) directory.
Fixture files must be:
- Small enough to commit (under a few MB, no full-resolution videos)
- Stable in name and content (changing a fixture invalidates downstream
  tests)
- Documented in a `tests/fixtures/README.md` if non-obvious

Larger integration assets live in a separate, gitignored test-media
folder; CI fetches them or skips the integration tests.

---

## Discipline

### No flaky tests

If a test depends on timing, sleep durations, filesystem ordering,
clock skew, or "usually passes," fix the test design — don't add
retries. A test that flakes once will flake again, often at the worst
moment.

### No skipped tests as a workaround

`@skip` (or equivalent) is for tests that genuinely don't apply to the
current environment (Windows-only test on Linux). It is not for tests
that are broken. A broken test gets fixed or deleted.

### Tests test behavior, not implementation

Refactoring should not break tests. If your tests assert on private
state or specific call sequences, they're testing implementation. Test
the inputs and outputs of public interfaces.

### One assertion per concept

A test can have multiple `assert` lines, but it should test one
concept. "Test that the parser produces correct output for valid input"
is one concept; "test that it produces correct output for valid input
AND raises on invalid input" is two — split them.

---

## Workflow (rule 8)

Per coding-session rule 8: **Jamie runs the tests, Claude proposes the
commands.**

When Claude finishes a change that should be tested:
1. State which tests should run (unit suite, specific test file,
   integration marker).
2. Provide the exact command line for Jamie to paste.
3. State the expected outcome ("expect 47 passed, 2 skipped").
4. Wait for Jamie to confirm success or paste failures back.

Don't run tests as part of routine implementation; the tests Jamie
runs are the canonical pass/fail signal.

---

## Manual phase-exit walkthroughs

Phases in `docs/PROJECT_PLAN.md` typically end with exit criteria a
human verifies by clicking through the running system. The canonical
flow is the `/exit-test-plan` command:

- **Stage 1** reads the phase's exit criteria, the Prompt body in
  `docs/CLAUDE_CODE_PROMPTS.md` (including its deviation footer), and
  the implementation files the criteria touch, then writes a
  walkthrough at `docs/test-plans/phase-NNN-exit.md`. Each test case
  uses a three-block shape: **Steps** (numbered, imperative),
  **Expected** (bulleted, verifiable, requirements cited inline),
  **Fail signals** (named regression modes pointing at code sites or
  requirements).
- Jamie runs the plan top-down and marks §4's run log inline.
  Mid-run failures route to normal coding sessions using the TC's
  Fail signals as starting points.
- **Stage 2** reads the marked run log + trailing notes, walks each
  finding, performs root-cause analysis where warranted, and lands
  dispositions across the project's docs (test plan §5,
  `docs/design-decisions.md`, `docs/open-questions.md`, `TODO.txt`).

See `.claude/commands/exit-test-plan.md` for the full spec. The
plan is the test artifact; Claude does not run the steps.

---

## Project-specific tooling

`/onboard` appends a section here describing this project's test
runner, marker syntax, and the canonical command lines Jamie pastes.
