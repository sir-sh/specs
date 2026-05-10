# S009 Success Criteria

| ID | Criterion | Validation Method |
|----|-----------|-------------------|
| S009-SC-001 | `sir run <workflow>` executes the workflow and exits 0 on success | Run a workflow with no inputs, verify exit code 0 |
| S009-SC-002 | `sir run --dry-run` prints plan without executing and exits 0 | Run `--dry-run` on any workflow, verify no side effects |
| S009-SC-003 | `sir run --yes` skips confirmation prompts and executes | Run a destructive workflow with `--yes`, verify execution |
| S009-SC-004 | `sir run --non-interactive` skips interactive prompts | Run with missing required input and `--non-interactive`, verify error |
| S009-SC-005 | `sir run <nonexistent>` exits 1 with error message | Run with invalid name, verify exit code and error text |
| S009-SC-006 | `sir plan <workflow>` displays step-by-step plan and exits 0 | Run plan on multi-step workflow, verify all steps listed |
| S009-SC-007 | `sir plan <nonexistent>` exits 1 with error message | Run with invalid name, verify exit code and error text |
| S009-SC-008 | `sir where` displays layers in precedence order and exits 0 | Run where, verify order matches S001 precedence |
| S009-SC-009 | `sir workflows:list` shows all workflows with type and layer | Run list, verify each workflow has type and layer attribution |
| S009-SC-010 | `sir workflows:show <workflow>` displays inputs, steps, metadata | Run show on known workflow, verify all fields present |
| S009-SC-011 | `sir recipes:install <source>` installs pack and exits 0 | Install a valid git pack, verify directory created |
| S009-SC-012 | `sir recipes:install` with invalid source exits 1 | Install from invalid URL, verify exit code 1 |
| S009-SC-013 | `sir recipes:list` shows installed packs | Run list, verify output format matches table structure |
| S009-SC-014 | `sir recipes:remove <id>` removes pack after confirmation | Remove an installed pack (with yes), verify directory gone |
| S009-SC-015 | `sir recipes:remove` cancelled exits 0 | Decline confirmation, verify exit 0 and no removal |
| S009-SC-016 | `sir recipes:update` outputs stub message and exits 0 | Run update, verify stub message and exit 0 |
| S009-SC-017 | Failed workflow step results in exit code 1 | Run workflow with failing step, verify exit code 1 |
| S009-SC-018 | `sir agent tools` lists all workflows, methods, tasks | Run agent tools, verify JSON output with all entities |
| S009-SC-019 | `sir agent plan` delegates to PlanCommand | Run agent plan, verify same output as `sir plan` |
| S009-SC-020 | `sir agent run` shows plan, prompts confirmation, delegates | Run agent run, verify plan shown, confirmation asked, execution on confirm |
| S009-SC-021 | `sir list` shows all available commands | Run list, verify it includes all sir commands |
| S009-SC-022 | All commands follow exit code conventions (0 success, non-zero failure) | Test success and failure cases for each command, verify codes |
| S009-SC-023 | Help text is accurate and complete for all commands | Run each command with `--help`, verify description matches behavior |
| S009-SC-024 | Error messages include the failed entity name | Trigger errors for each command, verify entity name in message |
| S009-SC-025 | `--help` on any command exits 0 | Run `--help` on all commands, verify exit 0 |