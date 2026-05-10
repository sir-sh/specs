# S010 Success Criteria

| ID | Criterion |
|----|-----------|
| S010-SC-001 [P1] | Steps with `if:` conditions that evaluate to `true` **MUST** execute normally. Steps with conditions that evaluate to `false` **MUST** be skipped. |
| S010-SC-002 [P1] | Skipped steps **MUST NOT** fail the workflow. The workflow result `success` field **MUST** be `true` when all executed steps succeed and all skipped steps were skipped due to condition evaluation (not errors). |
| S010-SC-003 [P1] | All operator types (comparison, logical, string, existence) **MUST** produce correct boolean results when evaluated against appropriate input values. |
| S010-SC-004 [P2] | A step with `continueOnError: true` that throws an exception **MUST NOT** halt workflow execution; subsequent steps **MUST** continue. |
| S010-SC-005 [P2] | A step with `timeout: N` that exceeds `N` seconds **MUST** be terminated and throw a `RuntimeException`. When `continueOnError: true`, the workflow continues; otherwise it halts. |
| S010-SC-006 [P2] | Multi-line `if:` conditions **MUST** be parsed correctly, ignoring newline whitespace, and produce the same result as equivalent single-line conditions. |
| S010-SC-007 [P2] | The `sir plan` command **MUST** display the `if:` condition and the `timeout` value for each conditional step. |
| S010-SC-008 [P3] | Adding conditional logic **MUST NOT** regress the execution time of workflows with no conditional steps by more than 10%. |
