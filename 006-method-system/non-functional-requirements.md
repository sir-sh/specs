# S006 Non-Functional Requirements

## Execution Latency

S006-NF-001 [P1] The MethodExecutor **MUST NOT** impose any artificial latency on method execution beyond the inherent cost of spawning a process.

S006-NF-002 [P2] Method discovery **SHOULD NOT** be performed inside the execution hot path. The `MethodRegistry` **MUST** be fully populated before any method is executed (discovery is a separate pre-flight step).

S006-NF-003 [P2] `MethodRegistry::get()` lookup **MUST** be O(1) or effectively constant-time (array key lookup). No filesystem I/O or directory scanning occurs at lookup time.

## Template Resolution in Method Context

S006-NF-004 [P1] Template resolution in shell commands **MUST** occur via the `Context` service's `resolveTemplate()` method, which **MUST** support the `{{variable}}`, `{{shellQuote(variable)}}`, and `{{default(variable, 'fallback')}}` syntaxes.

S006-NF-005 [P1] Template resolution **MUST** be applied before the shell command string is passed to `Process::fromShellCommandline()`, not after.

S006-NF-006 [P1] If a variable referenced in a template expression is not present in Context, the resolution **MUST** leave the expression unexpanded (pass through as-is) rather than producing an error or empty string, unless `{{default(...)}}` syntax is used.

S006-NF-007 [P2] Context **MUST** be populated with validated arguments (via `$context->set()`) **before** template resolution occurs, so that args can be referenced as variables in shell command templates.

## Streaming Output Buffering

S006-NF-008 [P1] When `stream: true`, stdout **MUST** be emitted to the terminal via `echo` and stderr **MUST** be emitted to `STDERR` in real-time as each chunk is received from the subprocess, not batched until process completion.

S006-NF-009 [P2] When `stream: false`, the MethodExecutor **MUST NOT** write any output to the terminal during execution. All output **MUST** be captured in memory and returned in the `output` field of the result array.

S006-NF-010 [P1] Streaming **MUST** work correctly for both `shell` and `exec` implementation types.

S006-NF-011 [P3] Large output volumes **MUST NOT** cause streaming to stall or buffer overflow. The callback-based approach used by `symfony/process` handles arbitrary output sizes without in-memory accumulation of the entire output string before emission.

## Memory Usage

S006-NF-012 [P2] The MethodRegistry **MUST NOT** hold file handles open after `discoverMethods()` completes. Each method JSON file **MUST** be read and closed within `loadMethodsFromDirectory()` before returning.

S006-NF-013 [P3] Memory usage for discovered methods **SHOULD** be proportional to the number of method definitions loaded, not to the size of their files. Large JSON files (within reason) **SHOULD** not cause excessive memory consumption.

## Error Reporting

S006-NF-014 [P1] All `InvalidArgumentException` messages thrown by the MethodExecutor **MUST** be self-contained and descriptive enough to identify which argument or implementation type caused the failure, without requiring access to the method definition.

S006-NF-015 [P2] Shell command execution failure (non-zero exit code) **SHOULD NOT** throw an exception. The failure **MUST** be reported in the result array via `exitCode` and `success: false`.

## Discovery Robustness

S006-NF-016 [P1] Method discovery **MUST NOT** fail the entire process if a single method JSON file is malformed. Malformed files **MUST** be silently skipped and processing **MUST** continue with remaining files.

S006-NF-017 [P2] Duplicate method registrations **MUST** follow last-wins semantics. When two methods share the same fully-qualified name, the later-registered definition **MUST** overwrite the earlier one.
