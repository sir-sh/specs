# S011 Success Criteria

## Iteration Completion

### S011-SC-001 [P1] All iterations execute when no break/continue
The WorkflowRunner **MUST** execute every iteration in the `forEach:` array when neither `breakOn:` nor `continueOn:` terminates early. The total iteration count **MUST** equal the array length. No iteration **MAY** be skipped or deduplicated.

### S011-SC-002 [P1] Parallel execution completes within timeout
When `parallel: true` is set, all iterations **MUST** complete (or fail) within the configured or default timeout. A worker that does not respond within the timeout **MUST** be treated as failed and the error **MUST** be propagated to the results array or exception handler.

### S011-SC-003 [P1] saveAs produces correct array length
When `saveAs:` is specified, the resulting array length **MUST** exactly match the `forEach:` array length, including `null` or sentinel entries for skipped (continueOn) iterations. Index alignment **MUST** be preserved.

### S011-SC-004 [P2] breakOn terminates loop and preserves results
When `breakOn:` evaluates to truthy, the loop **MUST** terminate immediately after the current iteration completes. The `saveAs:` array (if specified) **MUST** contain results for all iterations up to and including the breaking iteration. Subsequent items **MUST NOT** appear in the results.

### S011-SC-005 [P2] continueOn skips iteration and preserves index
When `continueOn:` evaluates to truthy for an iteration, that iteration **MUST NOT** execute. The results array entry for that index **MUST** be set to `null` or a defined skipped sentinel. Index alignment **MUST** be maintained for all subsequent iterations.

### S011-SC-006 [P1] continueOnError handles per-iteration failures
When `continueOnError: true` is set on a loop step, a failing iteration **MUST NOT** terminate the loop. The error **MUST** be captured in the results array at the corresponding index. When `stopOnError: true` (default), a failing iteration **MUST** terminate the loop and propagate the exception.

## Nested Loops

### S011-SC-007 [P2] Nested loops produce Cartesian product count
For nested `forEach:` loops, the total iteration count **MUST** equal the product of all array lengths. Each inner iteration **MUST** have access to all enclosing loop variables via their respective `as:` or default names.

### S011-SC-008 [P2] Matrix forEach produces correct Cartesian product
When `forEach:` is a map of arrays, the total iteration count **MUST** equal the Cartesian product of all value arrays. Each `{{item}}` **MUST** be an object containing all keys from the `forEach:` map.

## Template Resolution

### S011-SC-009 [P1] forEach template resolves before iteration
The `forEach:` expression **MUST** be resolved as a template before iteration begins. If resolution fails or yields a non-array, the step **MUST** throw an error before any iteration occurs.

### S011-SC-010 [P2] {{item}} and {{index}} accessible throughout iteration
`{{item}}` and `{{index}}` **MUST** be accessible in `shell:`, `task:`, `with:`, `breakOn:`, and `continueOn:` properties within a single iteration. Their values **MUST** reflect the current iteration's state.
