# S007 Non-Functional Requirements

### Task Execution Performance

S007-NF-001 [P1] The `git.clone` task **MUST** enforce a timeout of 300 seconds to prevent indefinite hangs on slow or unreachable remotes.

S007-NF-001.a On timeout, the task **MUST** throw a `RuntimeException` with a message indicating the operation timed out and including the target repository URL.

S007-NF-002 [P1] The `tests.auto` task **MUST** enforce a timeout of 600 seconds to prevent indefinite hangs on test suites that never complete.

S007-NF-002.a On timeout, the task **MUST** throw a `RuntimeException` with a message indicating which test command timed out.

S007-NF-003 [P2] Task execution **SHOULD** complete within a reasonable time relative to the operation type; file I/O tasks (`files.write`, `files.writeMd`) **SHOULD** complete in under 1 second for typical payloads.

### Security

S007-NF-004 [P1] The `files.write` task **MUST** prevent path traversal attacks by rejecting `path` arguments that escape the project root when resolved.

S007-NF-004.a The task **MUST** resolve the final normalized path and verify it lies within an allowed directory list (project root, temp dir, or explicitly configured write paths).

S007-NF-004.b On path traversal detection, the task **MUST** throw a `RuntimeException` with a message that does not reveal the attempted traversal path.

S007-NF-005 [P1] The `git.clone` task **MUST** verify SSH host keys when cloning via SSH URLs (e.g., `git@github.com:...`) by using `StrictHostKeyChecking=yes` or the equivalent Git configuration.

S007-NF-005.a The task **MUST NOT** leak repository credentials into logs or error messages.

S007-NF-006 [P1] The `git.clone` task **MUST NOT** pass credentials embedded in the repository URL (e.g., `https://user:password@github.com/...`) to the underlying git process.

S007-NF-007 [P2] All task output **SHOULD** redact sensitive values (tokens, passwords, SSH keys) from log messages and error reports.

### Reliability

S007-NF-008 [P1] Every built-in task **MUST** provide a meaningful error message on failure that describes what went wrong and, where applicable, what the caller provided.

S007-NF-008.a Error messages **MUST NOT** expose internal implementation details (e.g., stack traces, file paths of internal caches) to the workflow engine.

S007-NF-009 [P1] No built-in task **MUST** swallow exceptions silently; any caught exception **MUST** be re-thrown as a `RuntimeException` or `InvalidArgumentException` with a user-friendly message.

S007-NF-010 [P1] The `workspace.temp` task **MUST** throw a `RuntimeException` if the temporary directory cannot be created, including the underlying OS error message.

S007-NF-011 [P1] The `files.write` and `files.writeMd` tasks **MUST** throw a `RuntimeException` if the write operation fails after parent directory creation, including the file path and underlying OS error.

S007-NF-012 [P1] The `git.clone` task **MUST** throw a `RuntimeException` with the git stderr output when the clone process exits with a non-zero code.

S007-NF-013 [P1] The `tests.auto` task **MUST** throw a `RuntimeException` with a descriptive message when no test framework is detected (no `package.json` with `scripts.test` and no `composer.json` with `scripts.test`).

### Built-in Task Discovery

S007-NF-014 [P1] The `TaskRegistry` **MUST** successfully load and register all six built-in tasks at application boot without throwing any exception.

S007-NF-014.a If any task fails to load, the `AppServiceProvider` **MUST** throw a `RuntimeException` during container compilation identifying the failing task.

S007-NF-015 [P1] When the `WorkflowRunner` receives a task name that is not registered in the `TaskRegistry`, it **MUST** throw a `RuntimeException` with a message that lists all available task names.

S007-NF-015.a The error message **MUST** be deterministic and include the six built-in task names in a stable order.

S007-NF-016 [P2] The `TaskRegistry` **SHOULD** allow tasks to be resolved by exact name only (no aliases), ensuring that tool discovery via `sir agent tools` returns a consistent set of names.

### Interface Stability

S007-NF-017 [P1] The `TaskInterface` contract **MUST** remain stable; adding a new built-in task **MUST** only require implementing `execute(array $args, Context $context): mixed` and `getMetadata(): array` without changes to the interface.

S007-NF-017.a The interface **MUST NOT** add additional required methods after the spec is marked stable.

S007-NF-018 [P1] The `TaskRegistry` API (`register()`, `get()`, `has()`, `all()`) **MUST** remain stable; no method signatures **MAY** change in ways that break existing callers.

S007-NF-019 [P2] Task metadata schema **SHOULD** remain backward-compatible; new optional fields **MAY** be added to `getMetadata()` output without breaking existing consumers.

### Scalability

S007-NF-020 [P2] The `TaskRegistry` **SHOULD** support registration of up to 50 tasks without measurable performance degradation during task resolution.