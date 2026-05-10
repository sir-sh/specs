# S009 Non-Functional Requirements

## Performance

S009-NF-001 [P1] The system **MUST** complete workflow execution within 2 seconds for workflows containing up to 50 steps on standard hardware.

S009-NF-002 [P1] The system **MUST** generate execution plans within 1 second for workflows containing up to 50 steps.

S009-NF-003 [P1] The system **MUST** complete workflow discovery within 2 seconds when there are up to 100 workflows across all layers.

S009-NF-004 [P1] The system **MUST** complete pack installation within 30 seconds for typical git sources (under 10MB repository size).

S009-NF-005 [P2] The system **SHOULD** respond to `sir where` within 500ms regardless of layer count.

## Exit Code Conventions

S009-NF-006 [P1] All commands **MUST** exit with code 0 on successful completion.

S009-NF-007 [P1] All commands **MUST** exit with a non-zero code on failure.

S009-NF-008 [P1] Exit codes **MUST** be consistent with POSIX standards:
- 0 = success
- 1 = general error
- 2 = invalid arguments
- 127 = command not found (not used by sir commands)

## Error Messages

S009-NF-009 [P1] Error messages **MUST** be descriptive and include the failed entity name.

S009-NF-010 [P1] Error messages **MUST** be human-readable, avoiding technical jargon where possible.

S009-NF-011 [P1] Error messages **MUST** provide actionable next steps when applicable.

S009-NF-012 [P2] Error messages **SHOULD** suggest similar workflow names when a lookup fails due to a typo.

## Output Formatting

S009-NF-013 [P1] All table output **MUST** use Laravel Zero's `$this->table()` formatting for consistency.

S009-NF-014 [P1] All status messages **MUST** use Laravel's output decorators (`$this->info()`, `$this->error()`, `$this->warn()`).

S009-NF-015 [P2] Workflow execution output **SHOULD** use distinct visual styling to distinguish between steps (e.g., checkmarks for success, X for failure).

S009-NF-016 [P2] Step output **SHOULD** be indented to show step hierarchy.

## Help Text

S009-NF-017 [P1] All commands **MUST** provide `--help` flag that displays usage information.

S009-NF-018 [P1] Help text **MUST** include the command description, arguments, options, and examples.

S009-NF-019 [P1] Help text **MUST** be accurate and reflect the actual command behavior.

## Non-Interactive Mode

S009-NF-020 [P1] Commands **MUST** work correctly in TTY-less environments (CI/CD pipelines, scripts).

S009-NF-021 [P1] The `--yes` flag **MUST** suppress all confirmation prompts by automatically answering "yes".

S009-NF-022 [P1] The `--non-interactive` flag **MUST** suppress all interactive input gathering and use default values.

S009-NF-023 [P1] When both `--yes` and `--non-interactive` are provided, `--non-interactive` takes precedence for input gathering while `--yes` applies to confirmations.

## Idempotency

S009-NF-024 [P2] Pack removal **SHOULD** be idempotent - safe to run multiple times with the same result.

S009-NF-025 [P2] Running `sir run --dry-run` multiple times **MUST** produce identical output.

## Compatibility

S009-NF-026 [P1] The CLI **MUST** be compatible with PHP 8.2 and above.

S009-NF-027 [P1] The CLI **MUST** work on Linux, macOS, and Windows (via WSL/Git Bash).

## Logging

S009-NF-028 [P3] Command invocations **MAY** be logged to a file for audit purposes.

S009-NF-029 [P3] Failed command executions **MAY** log the full stack trace for debugging.

## Security

S009-NF-030 [P2] Pack sources **SHOULD** be validated before cloning (URL format check).

S009-NF-031 [P2] The system **SHOULD NOT** execute workflows from packs without explicit user confirmation.

## Input Validation

S009-NF-032 [P1] Workflow names **MUST** be validated to prevent path traversal attacks.

S009-NF-033 [P1] Input values **MUST** be sanitized before being passed to shell commands.