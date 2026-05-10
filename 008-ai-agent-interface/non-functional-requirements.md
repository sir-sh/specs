# S008 Non-Functional Requirements

## Performance

S008-NF-001 [P1] Tool discovery **MUST** complete within 2 seconds for up to 100 workflows, 50 methods, and 20 tasks.

S008-NF-002 [P1] Plan generation **MUST** complete within 1 second for workflows with up to 50 steps.

S008-NF-003 [P1] The `tools` subcommand **MUST NOT** cache discovery results between invocations; each call **MUST** re-scan the filesystem.

## Reliability

S008-NF-004 [P1] If a workflow file becomes unreadable between `tools` and `plan`, the error **MUST** be surfaced clearly with the workflow name and error reason.

S008-NF-005 [P1] Invalid YAML in a workflow file **MUST NOT** cause `tools` to fail; invalid entries **MUST** be silently skipped with a warning logged.

## Output Clarity

S008-NF-006 [P1] Human-readable output **MUST** use distinct visual styling (via Laravel output styles) to differentiate section headings, item names, and descriptions.

S008-NF-007 [P1] JSON output **MUST** be formatted with 2-space indentation for readability.

## Compatibility

S008-NF-008 [P2] The agent interface **SHOULD** produce output compatible with MCP (Model Context Protocol) tool schema expectations.

S008-NF-009 [P2] The agent interface **SHOULD** work in non-interactive (TTY-less) environments; confirmation prompts **MUST** return `false` gracefully when no TTY is available.

## Error Handling

S008-NF-010 [P1] Missing workflow argument errors **MUST** include the correct usage string in the message.

S008-NF-011 [P1] Runtime errors from workflow execution **MUST** be propagated to the caller with exit code 1.

## Resource Usage

S008-NF-012 [P2] Memory usage during `tools` **MUST** stay below 128MB even when scanning large directory trees (enforced by PHP memory limit configuration).