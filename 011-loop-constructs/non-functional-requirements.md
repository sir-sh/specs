# S011 Non-Functional Requirements

## Parallel Execution Performance

### S011-NF-001 [P1] Parallel execution completes within 30s per worker
When `parallel: true` is specified, each worker **MUST** complete its iteration within 30 seconds under normal load. A worker that exceeds this threshold **MUST** be treated as failed and the loop **MUST** propagate the timeout error.

### S011-NF-001.a [P1] maxWorkers limits concurrent workers
The system **MUST NOT** spawn more than `maxWorkers` concurrent workers. If `maxWorkers` is not specified, the system **MUST** use a reasonable default limit that **MUST NOT** exceed the number of available CPU cores multiplied by 2.

### S011-NF-002 [P2] Parallel loop throughput scales with maxWorkers
Throughput (iterations per second) **SHOULD** scale approximately linearly with `maxWorkers` for CPU-bound iterations up to the number of available CPU cores. Beyond CPU core count, scaling **MAY** degrade due to contention.

## Memory Usage

### S011-NF-003 [P1] Memory bounded by maxWorkers, not iteration count
For parallel loops with `saveAs:`, the system **MUST NOT** hold all iteration results in memory simultaneously beyond what is needed for `maxWorkers` concurrent iterations. Results **SHOULD** be collected incrementally to avoid memory exhaustion on large arrays.

### S011-NF-003.a [P1] Large iteration arrays (1000+ items) **MUST** stream results
When `forEach:` contains more than 1000 items, the implementation **SHOULD** use a streaming approach that writes results to a temporary buffer rather than accumulating them in a single in-memory array.

### S011-NF-004 [P2] Memory usage is predictable for nested loops
Nested loops **MUST NOT** multiply memory requirements by the product of all array lengths. Memory **SHOULD** be scoped to the innermost active iteration.

## Streaming Output

### S011-NF-005 [P2] Streaming output during parallel loops
When `parallel: true` is active, the system **SHOULD** stream output from each worker as it becomes available rather than buffering all output until all iterations complete. Streamed output **MUST** be annotated with the iteration index to preserve ordering context.

### S011-NF-005.a [P2] Output streaming **MUST NOT** interleave across workers in a way that obscures which iteration produced which output line.

## Graceful Degradation

### S011-NF-006 [P1] Worker failure does not crash the loop
When a worker process fails (crashes, segfaults, or is killed) during a parallel loop, the system **MUST**:
- Detect the worker failure
- Record the failure as an error in the results array
- Continue processing remaining iterations if `continueOnError: true`
- Terminate the loop if `stopOnError: true` (default)

### S011-NF-006.a [P1] A crashed worker **MUST NOT** leave orphaned processes. The parent process **MUST** reap all child pids on loop completion or termination.

### S011-NF-007 [P2] System resource exhaustion handling
If the system runs out of file descriptors, memory, or other critical resources during a parallel loop, the system **MUST**:
- Fail gracefully with a descriptive error message
- Clean up any partially initialized workers
- Preserve already-completed results where possible

### S011-NF-008 [P2] Timeout during parallel execution
If an iteration exceeds the configured or default timeout, the system **MUST**:
- Kill the stalled worker
- Record a timeout error in the results array at the corresponding index
- Continue or terminate based on `continueOnError` / `stopOnError`

## Serial Execution

### S011-NF-009 [P1] Serial loop memory usage is O(1) per iteration
For serial loops (parallel: false), memory usage **MUST** be constant with respect to iteration count. Only the current iteration's result **MAY** be held in memory before being appended to the `saveAs:` array.

### S011-NF-010 [P2] Serial loop execution order is deterministic
Serial loop iterations **MUST** execute in array order from index 0 to n-1. Results **MUST** be collected in the same order.