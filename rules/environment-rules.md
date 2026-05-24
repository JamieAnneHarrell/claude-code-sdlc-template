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

- **Shell escaping:** Discovered during Composer dependency installations.
  PowerShell commands with carats need to be escaped or written differently.
  Constraint syntax `5.*` rather than `^5` when installing because
  PowerShell strips the caret from `^5` even inside single quotes.
  `5.*` and `^5` are equivalent for our purposes (`>= 5.0, < 6.0`).
  This is a Windows-PowerShell-only quirk; the constraint behaves
  identically across platforms once Composer parses it.
- **Shell rediction:** You discivered this during an interactive session.
  Stop appending 2>&1 to native commands in PowerShell. It's wrapping clean 
  stderr in NativeCommandError records. The system prompt covers this and
  you keep ignoring it.

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

Jamie's working environment for these projects is fixed and known.
**Don't rediscover it each session.**

### Dangerous commands: Jamie always runs destructive commands
NEVER run and never ask for permission to run commands such as
recursive forced deletes. rm -rf and powershell equivalent is off 
limits entirely.

### Tool choice: match the project's documented shell

Claude has both a Bash tool and a PowerShell tool. Use the one that
matches the project's documented shell (this project: PowerShell, per
the project-specific section below). Do not default to Bash because
the command would also work there — match the shell Jamie develops in
so command shapes (path separators, vendor binary extensions,
chaining operators, env-var syntax) stay consistent with what she
sees in her terminal.

The Bash tool is appropriate only when (a) the command is genuinely
POSIX-only and has no PowerShell equivalent, or (b) the project's
documented shell is bash (Linux/macOS-only project, or WSL2-only
build scripts).

### Commands Jamie pastes (PowerShell snippets, rule-7 commit blocks, docs)

Call binaries **bare** — `pytest`, `ruff format`, `auto-dailies`,
`ffprobe`. Her venv is already activated in her VSCode PowerShell
terminal. Do not probe with `which`, `where`, `Test-Path`, or
`Get-Command`.

### Commands Claude runs via its own Bash or PowerShell tools

These shells do **not** inherit Jamie's activated venv. Prefix the
venv-relative path:

- **PowerShell:** `.\.venv\Scripts\pytest.exe ...`
- **Bash (Windows git-bash):** `./.venv/Scripts/pytest.exe ...`
- **WSL2 Linux:** `./.venv/bin/pytest ...`

Do not try to activate the venv first — just prefix. This is the only
context where the `.venv\Scripts\` prefix is correct in a command
Claude writes.

### Multi-command operations

PowerShell does not support `&&` chaining in Windows PowerShell 5.1.
Use `;` for unconditional sequencing or `if ($?) { ... }` for
conditional. In Bash, prefer `&&` for "run B only if A succeeded."

For commit handoffs (rule 7) the rule is stricter: **one command per
copyable line**, no chaining. Jamie pastes one line at a time.

---

## Scratch files

When Claude needs to write a temporary file for diagnostic purposes
(JSON inspection copy, throwaway probe, intermediate artifact), use
a defined project temp directory named "tmp" at the project root; OS 
temp directory on Windows is unreliable.

`/tmp` and `/c/tmp` do not exist on Windows. Do not assume them.

Do **not** invent a repo-local scratch directory. Application output
(real artifacts the project produces) is separate and goes to the
project's documented output location. If a project's "tmp" directory
does not exist and you need it, offer to create it.

---

## Logging discipline

Use the language's structured logging library, not `print` /
`echo` / `console.log` in business logic.

- **Python:** `logging` module
- **PHP/Laravel:** `Log::info(...)` / `Log::warning(...)`
- **JS/Node:** a logging library (winston, pino, etc.) — not
  `console.log` in production code

`print` (or equivalent) is reserved for:
- Final user-facing summary output of a CLI run
- CLI help text
- Progress bars (tqdm and equivalents — these are not "print")

---

## Version freshness

Training-corpus version recall is unreliable. By the time a project
gets configured, the "current" Python / Node / Postgres / Laravel /
Docker base image Claude remembers from training is often months or
years stale. Pinning a stale version into a project artifact (README
prerequisites, `<!-- ONBOARD-FILL: environment -->` block,
`docs/DEPLOYMENT.md`) bakes that drift in for the life of the
project.

### The rule

Before pinning **any** software version into a project artifact:

1. **Surface the assumption to Jamie.** Say plainly that the
   version about to be pinned comes from training-corpus recall (or
   from a spec that may itself be stale) and could be out of date.
2. **Offer to look up the current stable / LTS release** via
   `WebSearch` or `WebFetch`. Wait for explicit go-ahead before
   running the lookup. One offer per version, not a blanket
   batched one.
3. **Show what was found** alongside any spec'd or recalled
   version, and let Jamie pick which goes in the file.

Only the version Jamie confirms after that exchange goes into the
artifact.

### Applies even when a version is already specified

A design doc, `docs/REQUIREMENTS.md`, `docs/ARCHITECTURE.md`, an
existing rules file, or a previous session's answer naming a
specific version **does not bypass this check**. The session that
produced the spec may not have had this guardrail. Surface the
spec'd version, name the staleness risk, ask before adopting it
as-is.

### Never use a version for convenience

Do not pick the version that is easy to recall, easy to type, or
matches a familiar example in a command file. Examples in
`/onboard`, `/bootstrap`, and `/deployment-plan` (e.g. "Python
3.12", "Node 20", `python:3.12-slim`) are placeholder text — never
copy them into an artifact as a pin.

### Surfaces in scope

- Language runtimes: Python, Node, PHP, Ruby, Go, Rust, etc.
- Framework majors and LTS lines: Django, Laravel, Express, Next,
  Rails, Spring, etc.
- Database engines: Postgres, MySQL, MariaDB, Redis, MongoDB
  majors.
- Container base image tags: `python:3.12-slim`, `node:20-alpine`,
  `postgres:16`, etc.
- Cloud SDKs and CLIs: `aws-cli`, `gcloud`, `az`, Terraform
  providers, etc.
- Package managers: pip, poetry, uv, npm, pnpm, composer.
- Any pinned dependency that ships in `requirements.txt`,
  `package.json`, `composer.json`, etc., when the version is being
  *chosen* (not when reading what is already there).

### Where this fires in the configuration ritual

- `/onboard` Step 3 — when collecting language and framework
  versions in the answer dialog.
- `/bootstrap` Step 3 (questions) **and** Step 5 (writing the
  README "Prerequisites" subsection and the `<!-- ONBOARD-FILL:
  environment -->` block). The Step 5 check is the load-bearing
  one — even if Step 3 captured a version, ask again before it
  lands in a committed file.
- `/deployment-plan` Step 3 (questions about base images, DB
  engines) **and** Step 5 (writing `docs/DEPLOYMENT.md`).

---

## Database credentials policy

A backstop. The authoritative source is NFR-X "Secrets and credentials
hygiene" in `docs/REQUIREMENTS.md` (added by `/onboard` when the design
implies a DB or stored credentials). This rule exists so any DB-touching
work picks up the policy even if the NFR was skipped.

- **App DB user.** Never `root`, `admin`, `sa`, or any privileged
  default. Use a project-scoped name (e.g. `<slug>_app`,
  `<slug>_dev`). Same rule for prod.
- **CI ephemeral DB user.** Two-word-with-number pattern (e.g.
  `swift-otter-42`). Generated per CI run or per project. Must not
  reuse the app username and must not telegraph the project slug —
  the point is that a leaked credential leaks no project identity.

If a session is about to write a DB connection string with a
violating username, stop and surface the policy.

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
