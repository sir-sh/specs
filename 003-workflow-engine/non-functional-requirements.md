# S003 Non-Functional Requirements

### Execution Performance

S003-NF-001 [P2] The workflow engine **MUST** display step timing (elapsed seconds) for every executed step without significantly impacting execution speed.

S003-NF-003 [P3] The workflow engine **SHOULD** handle workflows with 100 or more steps without degraded performance. Step overhead (template resolution, context updates) **MUST** be O(1) per step.

### Shell Execution

S003-NF-002 [P2] Shell step execution **MUST** support a configurable timeout. If a shell command exceeds the timeout, the process **MUST** be terminated and a `RuntimeException` **MUST** be thrown with message `"Step {N} timed out after {seconds} seconds"`.

S003-NF-004 [P2] Shell step stdout and stderr **MUST** be streamed to the terminal in real-time during execution. Buffered output **MUST NOT** be held until after the process completes.

### Output Formatting

S003-NF-005 [P2] Step output headers **MUST** use the format `"=== Step {N}: {name} [{type}] ==="` and **MUST** appear exactly once per step before execution begins.

S003-NF-006 [P2] Error messages **MUST** be clearly distinguishable from success output, using a distinct prefix such as `[ERROR]` or `x` and **MUST** include the step number, step name, and original error message.

### Error Message Clarity

S003-NF-007 [P2] All runtime exceptions thrown by the workflow engine **MUST** include the step index and step name in their message to aid debugging.

S003-NF-008 [P3] The engine **SHOULD** suggest corrections when a method or task name is slightly misspelled (Levenshtein distance of 1), by appending `"Did you mean: {suggestion}?"` to the error message.