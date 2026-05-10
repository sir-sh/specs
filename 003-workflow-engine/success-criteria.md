# S003 Success Criteria

| ID | Criterion |
|----|-----------|
| S003-SC-001 [P1] | All three step types (`method`, `task`, `shell`) **MUST** execute correctly and produce expected outputs when invoked with valid parameters |
| S003-SC-002 [P1] | Context propagation **MUST** work end-to-end: a `saveAs` variable set by step N **MUST** be resolvable via `{{varName}}` in all subsequent steps N+1 through N+K |
| S003-SC-003 [P2] | The output of `sir plan <workflow>` **MUST** match the steps that would be executed in the same workflow run -- same count, same order, same step types and names |
| S003-SC-004 [P2] | Layer precedence **MUST** be correct: a workflow discovered in a nearer layer **MUST** shadow any workflow with the same name in a farther layer |