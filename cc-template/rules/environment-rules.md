# Environment Rules

Cross-platform conventions, shell handling, and where Claude puts
scratch files. `/onboard` appends project-specific tooling (venv path,
shell prefix, language version) at the bottom.

---

## Cross-platform discipline

### Windows is primary, Linux and macOS are supported

Jamie's primary OS is Windows 11. Linux and macOS are first-class
supported targets — anything that works on Linux but breaks on Windows
is a bug, not a platform quirk to document around.

### PowerShell command quirks

- **Caret escaping:** PowerShell strips `^` from constraint syntax
  like `^5` even inside single quotes. Use `5.*` instead — equivalent
  for our purposes (`>= 5.0, < 6.0`) and survives the shell intact.
  Windows-PowerShell-only quirk; behavior is identical across
  platforms once the receiving tool parses it.
- **Stderr redirection:** Avoid `2>&1` on native PowerShell commands.
  Stderr is already captured cleanly; the redirect wraps it in
  `NativeCommandError` records and inverts `$?` even on a successful
  exit code.

### Path handling

- **Python:** use `pathlib.Path` everywhere. No string concatenation
  with `/` or `os.path.join`. No forward-slash-only assumptions.
- **PHP/Laravel:** use Laravel's `Storage` facade or `base_path()` /
  `storage_path()` helpers. No hardcoded `/` separators.
- **JS/TS:** use `path.join` and `path.sep`. URL paths are not
  filesystem paths.
- **Shell scripts:** prefer cross-platform alternatives or write
  separate `.sh` and `.ps1` versions. Avoid bash-only scripts unless
  the project is explicitly Linux-only (e.g. WSL2 build scripts).

### Line endings

Use `.gitattributes` with `* text=auto` to let git normalize. Don't
commit CRLF-fixing scripts.

---

## Shell handling — when Claude runs commands vs. when Jamie does

Jamie's working environment is fixed and known. **Don't rediscover
it each session.**

### Dangerous commands: Jamie runs all destructive commands

Never run, and never ask for permission to run, recursive forced
deletes. `rm -rf` and its PowerShell equivalent are off limits
entirely.

### Tool choice: match the project's documented shell

Claude has both Bash and PowerShell tools. Use the one that matches
the project's documented shell (this project: PowerShell). Don't
default to Bash because the command would also work there — match
the shell Jamie develops in so command shapes (path separators,
chaining operators, env-var syntax) stay consistent with her
terminal.

Bash is appropriate only when (a) the command is genuinely POSIX-only
with no PowerShell equivalent, or (b) the project's documented shell
is bash (Linux/macOS-only, or WSL2-only build scripts).

### Commands Jamie pastes (PowerShell snippets, commit blocks, docs)

Call binaries **bare** — `pytest`, `ruff format`, `ffprobe`. Her
venv is already activated in her VSCode PowerShell terminal. Don't
probe with `which`, `where`, `Test-Path`, or `Get-Command`.

### Commands Claude runs via its own shell tools

These shells do **not** inherit Jamie's activated venv. Prefix the
venv-relative path:

- **PowerShell:** `.\.venv\Scripts\pytest.exe ...`
- **Bash (Windows git-bash):** `./.venv/Scripts/pytest.exe ...`
- **WSL2 Linux:** `./.venv/bin/pytest ...`

Don't try to activate the venv first — just prefix. This is the only
context where the `.venv\Scripts\` prefix is correct in a command
Claude writes.

### Multi-command operations

Windows PowerShell 5.1 doesn't support `&&` chaining. Use `;` for
unconditional sequencing or `if ($?) { ... }` for conditional. In
Bash, prefer `&&`. For commit handoffs (rule 7) the rule is
stricter: **one command per copyable line**, no chaining.

---

## Scratch files

When Claude needs a temporary file for diagnostics (JSON inspection
copy, throwaway probe, intermediate artifact), use a `tmp/`
directory at the project root. The OS temp directory on Windows is
unreliable, and `/tmp` / `/c/tmp` don't exist on Windows. Don't
invent other repo-local scratch directories. If `tmp/` doesn't
exist and you need it, offer to create it. Application output (real
artifacts the project produces) is separate and goes to the
project's documented output location.

---

## Logging discipline

Use the language's structured logging library, not `print` / `echo`
/ `console.log` in business logic.

- **Python:** `logging` module
- **PHP/Laravel:** `Log::info(...)` / `Log::warning(...)`
- **JS/Node:** winston, pino, etc. — not `console.log` in production
  code

`print` (or equivalent) is reserved for: final user-facing CLI
summary output, CLI help text, progress bars (tqdm and equivalents).

---

## HTML comments are free

Single-line `<!-- ... -->` content is stripped before Claude reads
CLAUDE.md and rules files — put human-only prose there, never content
Claude must act on.

---

## Version freshness

Never assume training-time versions are current — see
`rules/coding-session-rules.md` § Rule 10. Applies before pinning any
version into a project artifact.

---

## Database credentials policy

A backstop — authoritative source is the "Secrets and credentials
hygiene" NFR in `docs/REQUIREMENTS.md` (added by `/onboard` when the
design implies a DB). This rule fires even if the NFR was skipped.

- **App DB user:** never `root`, `admin`, `sa`, or any privileged
  default. Use a project-scoped name (`<slug>_app`, `<slug>_dev`).
- **CI ephemeral DB user:** two-word-with-number (e.g.
  `swift-otter-42`), generated per CI run or per project. Must not
  reuse the app username or telegraph the project slug — leaked
  credentials should leak no project identity.

If a session is about to write a connection string with a violating
username, stop and surface the policy.

---

## Project-specific environment

> *Onboarding fills in this section based on Jamie's answers.*

<!-- ONBOARD-FILL: environment -->

### Language and runtime
- (Onboarding adds e.g. "Python 3.12 in `.venv/`", "PHP 8.3 / Laravel
  11", "Node 20 / TypeScript")

### Venv / environment activation
- (Onboarding adds the project's specific path)

### Build / run / test commands
- (Onboarding adds the canonical command lines)

<!-- /ONBOARD-FILL -->
