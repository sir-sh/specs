# S011 Interface Requirements

## YAML Schema for forEach Steps

### S011-IF-001 [P1] forEach YAML schema
A workflow step with iteration **MUST** support the following YAML schema:

```yaml
step:
  # Iteration control
  forEach: string | array    # Collection to iterate over
  as: string               # Item variable name (default: item)

  # Parallel execution
  parallel: bool            # Execute iterations concurrently (default: false)
  maxWorkers: int          # Max concurrent workers (default: 1)
  continueOnError: bool    # Continue loop on iteration failure (default: false)
  stopOnError: bool        # Halt loop on iteration failure (default: true)

  # Flow control
  breakOn: string          # Template expression; terminate loop if truthy
  continueOn: string       # Template expression; skip iteration if truthy

  # Result collection
  saveAs: string           # Variable name to save results array

  # Execution
  shell: string            # Shell command (supports {{item}}, {{index}})
  task: string             # Task reference (supports {{item}}, {{index}})
  with:                    # Task arguments (key: "{{item}}" syntax)
    key: "{{item}}"
```

### S011-IF-001.a [P1] forEach accepts inline arrays
```yaml
steps:
  - shell: "echo {{item}}"
    forEach: ["file1.txt", "file2.txt", "file3.txt"]
```

### S011-IF-001.b [P1] forEach accepts template strings
```yaml
steps:
  - shell: "echo {{item}}"
    forEach: "{{files}}"   # Resolves to array from context
```

### S011-IF-001.c [P2] forEach accepts matrix (object of arrays)
```yaml
steps:
  - task: test.run
    forEach:
      os: ["ubuntu", "macos", "windows"]
      node: ["16", "18", "20"]
    with:
      os: "{{item.os}}"
      nodeVersion: "{{item.node}}"
```

## Implicit Variables

### S011-IF-002 [P1] {{item}} implicit variable
During loop iteration, the current collection element **MUST** be bound to `{{item}}`. This variable is accessible in `shell:`, `task:`, `with:`, `breakOn:`, and `continueOn:` properties.

### S011-IF-002.a [P1] {{item}} is renamed by `as:` property
When `as: myVar` is specified, `{{myVar}}` is bound instead of `{{item}}`.

### S011-IF-003 [P1] {{index}} implicit variable
During loop iteration, the zero-based integer index **MUST** be bound to `{{index}}`. This variable is accessible alongside `{{item}}`.

## Loop Result JSON Structure

### S011-IF-004 [P1] Loop results array structure
When `saveAs:` is specified, the saved variable **MUST** be a JSON array with one entry per `forEach:` item:

```json
{
  "results": [
    { "index": 0, "item": "value1", "result": "step-output-1" },
    { "index": 1, "item": "value2", "result": "step-output-2" },
    { "index": 2, "item": "value3", "result": null }
  ]
}
```

### S011-IF-004.a [P1] Skipped iterations have null result
When `continueOn:` skips an iteration, the result entry **MUST** be `null` or include a `skipped: true` flag to preserve index alignment.

### S011-IF-004.b [P1] Failed iterations include error field
When `continueOnError: true` and an iteration fails, the result entry **MUST** include an `error` field with the exception message.

## PHP Class Interface

### S011-IF-005 [P1] LoopExecutor PHP class
The `LoopExecutor` class **MUST** implement the following interface:

```php
interface LoopExecutorInterface
{
    /**
     * Execute a step with forEach iteration.
     *
     * @param WorkflowStep $step
     * @param array $context
     * @return array Results array (one entry per iteration)
     */
    public function execute(WorkflowStep $step, array $context): array;

    /**
     * Get the current item variable name (respects `as:`).
     */
    public function getItemVariableName(): string;

    /**
     * Get the current index variable name.
     */
    public function getIndexVariableName(): string;
}
```

### S011-IF-006 [P1] ParallelLoopExecutor PHP class
The `ParallelLoopExecutor` class **MUST** extend or implement `LoopExecutorInterface` and add:

```php
interface ParallelLoopExecutorInterface extends LoopExecutorInterface
{
    /**
     * Execute iterations in parallel with bounded concurrency.
     *
     * @param WorkflowStep $step
     * @param array $context
     * @param int $maxWorkers Maximum concurrent workers
     * @param bool $continueOnError
     * @return array Results array
     */
    public function executeParallel(
        WorkflowStep $step,
        array $context,
        int $maxWorkers = 1,
        bool $continueOnError = false
    ): array;
}
```

### S011-IF-007 [P2] NestedLoopExecutor PHP class
Nested loops **MUST** be supported by a `NestedLoopExecutor` that maintains a stack of loop contexts, pushing a new context for each nested `forEach:` and popping on completion.