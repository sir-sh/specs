# S008 Success Criteria

| ID | Criterion | Measure |
|----|-----------|---------|
| S008-SC-001 | All three subcommands respond without error | `sir agent tools`, `sir agent plan <workflow>`, `sir agent run <workflow>` each exit with code 0 when invoked correctly |
| S008-SC-002 | tools output includes all discoverable items | `sir agent tools` output contains every workflow from `WorkflowLoader::discoverWorkflows()`, every method from `MethodRegistry::discoverMethods()`, and every task from `TaskRegistry::all()` |
| S008-SC-003 | plan output matches PlanCommand output | Running `sir agent plan <workflow>` produces identical output to `sir plan <workflow>` for the same workflow |
| S008-SC-004 | run delegates to RunWorkflowCommand | Running `sir agent run <workflow>` after confirming produces identical output to `sir run <workflow>` for the same workflow |
| S008-SC-005 | Default action is tools | `sir agent` with no action argument runs `tools`; verified by calling `sir agent` and observing the tool listing output |
| S008-SC-006 | Unknown action shows help and exits 0 | `sir agent invalid-action` displays help text and exits with code 0, not 1 |
| S008-SC-007 | Missing workflow argument (plan) exits 1 | `sir agent plan` prints error and exits with code 1 |
| S008-SC-008 | Missing workflow argument (run) exits 1 | `sir agent run` prints error and exits with code 1 |
| S008-SC-009 | Confirmation cancellation exits 0 | `sir agent run <workflow>` answered with `n` outputs `Cancelled.` and exits with code 0 |
| S008-SC-010 | JSON tools output is valid JSON | `sir agent tools --json` parses successfully with `json_decode()` and yields an associative array with `workflows`, `methods`, and `tasks` keys |
| S008-SC-011 | JSON plan output is valid JSON | `sir agent plan <workflow> --json` parses successfully with `json_decode()` and yields an associative array with a `steps` key |
| S008-SC-012 | --yes skips confirmation | `sir agent run <workflow> --yes` proceeds to execution without prompting, confirmed by observing no `Execute this workflow?` text in output |
| S008-SC-013 | Agent can round-trip tools then plan | After calling `sir agent tools`, an AI can call `sir agent plan <workflow-name-from-list>` and receive a valid plan |
| S008-SC-014 | Tool discovery is not cached | Calling `sir agent tools` twice in succession, after adding a new workflow file between calls, shows the new workflow in the second call |