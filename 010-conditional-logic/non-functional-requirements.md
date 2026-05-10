# S010 Non-Functional Requirements

### Condition Evaluation Overhead

S010-NF-001 [P2] Evaluating a condition on a step **MUST** add no more than 1 millisecond of overhead to step startup time when measured in isolation.

S010-NF-002 [P3] The `ConditionEvaluator` **SHOULD** be designed with a clearly separated `evaluate()` public method so that new operators can be added by extending the class without modifying the core parsing logic.

### Output Formatting for Skipped Steps

S010-NF-003 [P2] When a step is skipped due to a false condition, the runner **MUST** display a skip indicator (e.g., `-`) and the reason in the same format used for step status display.

S010-NF-004 [P2] When a step fails due to `continueOnError: false` (normal behavior), the error output **MUST** be clearly distinguishable from skipped steps, using a distinct prefix such as `[ERROR]` or `x`.

### Timeout Enforcement

S010-NF-005 [P2] The timeout mechanism **MUST** interrupt the running process asynchronously without leaving orphaned child processes. The implementation **MUST** use platform-appropriate process termination (e.g., `Process::stop()` on Symfony Process).

### Warning and Diagnostic Output

S010-NF-006 [P2] Warnings about invalid condition syntax or undefined variables **MUST** be written to stderr and **MUST** use a distinct `[WARN]` prefix so that CI systems can capture and surface them.

S010-NF-007 [P3] The condition evaluator **SHOULD** emit a debug-level log entry for each condition evaluated when a debug mode is enabled, showing the raw condition string and the resolved expression before evaluation.
