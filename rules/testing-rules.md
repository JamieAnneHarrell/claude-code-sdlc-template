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

### Functional tests
Tests that exercise a slice of the application against a real database
of the **same engine used in production** — HTTP routes, ORM queries,
migrations, jobs that touch the DB. They run by default (they are not
integration tests in the marker sense), but they need a real engine
because behavior diverges across engines.

Functional tests must use a separate test database from the dev
database, and must use the production engine — never substitute SQLite
or another "lighter" engine for speed. See "No engine substitution"
under Discipline below.

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

### No engine substitution

If production runs on MariaDB, tests run on MariaDB. If production
runs on Postgres, tests run on Postgres. Do not run functional tests
against SQLite (or against a different SQL engine — including MySQL
when production is MariaDB, or vice versa) for speed. Engine
differences in JSON storage and functions, ENUMs, FULLTEXT,
foreign-key enforcement defaults, transaction isolation, and string
collation cause tests to pass while production breaks on the same
query.

Tests that don't need a database don't get one — that's how unit tests
stay fast, not by swapping engines underneath them.

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

This project has **no traditional test suite**. The deliverable
is markdown only — no language runtime, no compilable code, no
unit / functional / integration test categories in the usual
sense. The categories and discipline sections above apply for any
*future* tooling (e.g. Phase 3 regression-test automation), but
they're aspirational right now.

What functions as testing today:

1. **Re-read the six command files end-to-end after a change.**
   Catches: status name mismatches, file ownership overlap,
   broken prereq references, drift in the design-review or
   test-plan filename / placeholder conventions.
2. **Mental dry-run** against a hypothetical consumer project
   (Python CLI, PHP/LAMP, Node/web). Catches: prompts that don't
   make sense for a real stack, outputs that contradict each
   other.
3. **Live test (the canonical walkthrough).** Copy `cc-template/`
   to a sandbox directory, drop a known-good design doc into
   `docs/design/`, then run the full chain in order: `/onboard`
   → `/design-review` (stage 1 initial) → mark up the checkpoint
   → `/design-review` (stage 2 walks dispositions) → research
   session → re-run `/design-review` (stage 1 addendum) →
   `/design-review` (stage 2 lands) → `/bootstrap` → Phase 0 work
   → `/exit-test-plan` (stage 1 initial) → run, fill §4 →
   `/exit-test-plan` (stage 2 dispositions) → polish session
   lands fixes → `/exit-test-plan` (stage 1 addendum) → run
   §6.1, fill its run log → `/exit-test-plan` (stage 2 lands) →
   between-phase `/design-review` → `/deployment-plan` (both
   full and deferred paths) → `/wind-down`. Inspect outputs
   against each command file's spec.

When Phase 3 regression-test automation ships, the live test
remains the highest-confidence check; the scripted check is a
faster pre-flight that catches invariant-breaking changes (status
comment renames, ONBOARD-FILL marker drift, zero-pad-width
changes) before they reach the live test.

### Test runner

None. The "tests" are: re-reading command files (human eye),
mental dry-run (human reasoning), and the live-test walkthrough
(human execution).

### Marker syntax for integration tests

Not applicable — there are no integration tests as such. The
closest analog is the live-test walkthrough's between-phase
`/design-review` checkpoints and the `/exit-test-plan` flows;
those are *the* integration tests for this project.

### Expected test counts

Not applicable. The relevant counts are:
- **Six command files** at both root `.claude/commands/` and
  `cc-template/.claude/commands/` (12 total).
- **Three status comments** in `cc-template/CLAUDE.md`.
- **Two ONBOARD-FILL marker pairs** across the rules files.

Phase 3's regression-test automation may pin these as
machine-checked invariants.
