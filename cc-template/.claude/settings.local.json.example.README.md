# `.claude/settings.local.json` — usage notes

Copy `settings.local.json.example` to `settings.local.json` and adjust
per project. `settings.local.json` is gitignored — it's per-machine
configuration, not part of the shipped project.

## What goes in here

### `permissions.allow`
List of `Bash(<command pattern>)` strings that Claude is allowed to run
without prompting. The `*` is a wildcard for any trailing arguments.

The example values are the formatter / linter / pre-commit checks
Claude actually runs during the rule-7 commit dry-run. Replace them
with your project's actual tooling.

Don't put test runners here — per rule 8, Jamie runs the tests.
Don't put destructive commands here (rm, git push, etc.) — those
should always prompt.

The `/fewer-permission-prompts` skill can regenerate this allowlist
based on your recent transcripts after a few sessions of real work.

### `env`
Per-project environment variables Claude's own shell tools see. The
most common use is `VIRTUAL_ENV` and `PATH` so `pytest`, `ruff`, etc.
resolve correctly when Claude runs them via Bash or PowerShell.

The example has Windows-style paths. On Linux/macOS, use forward
slashes:

```json
{
  "env": {
    "VIRTUAL_ENV": "/home/jamie/code/<project>/.venv",
    "PATH": "/home/jamie/code/<project>/.venv/bin:<rest of system PATH>"
  }
}
```

### Other settings (optional)
- `model`: pin a model for this project (`opus`, `sonnet`, `haiku`)
- `effortLevel`: `low` | `medium` | `high`
- `autoCompactEnabled`: usually leave default
- See Claude Code docs for the full settings schema.
