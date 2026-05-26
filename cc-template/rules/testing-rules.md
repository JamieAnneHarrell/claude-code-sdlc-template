# Testing Rules

Language-neutral test discipline. `/onboard` appends project-specific
tooling (test runner, marker syntax, command lines) at the bottom.

---

## Categories

### Unit tests
Pure logic, no external dependencies. No filesystem fixtures, no
network, no databases, no real subprocess calls. Bulk of the test
suite; runs on every commit, every CI build, every machine.

### Functional tests
Exercise a slice of the app against a real database **of the same
engine used in production** — HTTP routes, ORM queries, migrations,
DB-touching jobs. Run by default. Use a separate test DB from dev,
and never substitute SQLite or a "lighter" engine for speed (see
"No engine substitution" below).

### Integration tests
Exercise external dependencies — real binaries (ffmpeg, ffprobe),
real databases, real network, real hardware. Live behind a
test-runner marker and **skipped by default** in CI and routine
local runs. Run when explicitly requested, typically pre-release or
when a change touches boundary code.

### Test fixture data
Live in a designated `tests/fixtures/` directory. Files must be:
small enough to commit (under a few MB, no full-resolution videos);
stable in name and content (changing a fixture invalidates
downstream tests); documented in `tests/fixtures/README.md` if
non-obvious. Larger integration assets live in a separate,
gitignored test-media folder.

---

## Discipline

### No flaky tests

If a test depends on timing, sleep durations, filesystem ordering,
or "usually passes," fix the test design — don't add retries. A
test that flakes once will flake again at the worst moment.

### No skipped tests as a workaround

`@skip` (or equivalent) is for tests that genuinely don't apply to
the current environment (Windows-only test on Linux). It is not for
broken tests — fix or delete those.

### Tests test behavior, not implementation

Refactoring shouldn't break tests. If a test asserts on private
state or specific call sequences, it's testing implementation. Test
inputs and outputs of public interfaces.

### One assertion per concept

A test can have multiple `assert` lines but should test one concept.
"Parser produces correct output for valid input" is one concept;
"...AND raises on invalid input" is two — split them.

### No engine substitution

If production runs on MariaDB, tests run on MariaDB. If on Postgres,
tests on Postgres. Engine differences in JSON storage / functions,
ENUMs, FULLTEXT, foreign-key enforcement, transaction isolation, and
string collation cause tests to pass while production breaks on the
same query.

Tests that don't need a database don't get one — that's how unit
tests stay fast, not by swapping engines underneath them.

---

## Workflow (rule 8)

Per coding-session rule 8: **Jamie runs the tests, Claude proposes
the commands.** When Claude finishes a change that should be tested:

1. State which tests should run (unit suite, specific test file,
   integration marker).
2. Provide the exact command line for Jamie to paste.
3. State the expected outcome ("expect 47 passed, 2 skipped").
4. Wait for Jamie to confirm success or paste failures back.

Don't run tests as part of routine implementation; Jamie's run is
the canonical pass/fail signal.

---

## Manual phase-exit walkthroughs

Phases in `docs/PROJECT_PLAN.md` typically end with exit criteria a
human verifies by clicking through the running system. Canonical
flow is `/exit-test-plan`:

- **Stage 1** reads the phase's exit criteria, the Prompt body in
  `docs/CLAUDE_CODE_PROMPTS.md` (including its deviation footer),
  and the implementation files those criteria touch, then writes a
  walkthrough at `docs/test-plans/phase-NNN-exit.md`. Each test case
  has a three-block shape: **Steps** (numbered, imperative),
  **Expected** (bulleted, verifiable, requirements cited inline),
  **Fail signals** (named regression modes pointing at code sites
  or requirements).
- Jamie runs the plan top-down and marks §4's run log inline.
  Mid-run failures route to normal coding sessions using the TC's
  Fail signals as starting points.
- **Stage 2** reads the marked run log + trailing notes, walks each
  finding, performs root-cause analysis where warranted, and lands
  dispositions across the project's docs (test plan §5,
  `docs/design-decisions.md`, `docs/open-questions.md`, `TODO.txt`).

See `.claude/commands/exit-test-plan.md` for the full spec. The plan
is the test artifact; Claude does not run the steps.

---

## Project-specific tooling

`/onboard` appends a section here describing this project's test
runner, marker syntax, and canonical command lines Jamie pastes.
