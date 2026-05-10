# S007 Success Criteria

### S007-SC-001 All 6 tasks registered at boot [P1]

After `AppServiceProvider::register()` completes, `TaskRegistry::all()` **MUST** return exactly 6 entries with keys: `workspace.temp`, `files.write`, `files.writeMd`, `git.clone`, `tests.auto`, `coderabbit.review`.

### S007-SC-002 All tasks implement TaskInterface [P1]

Every value returned by `TaskRegistry::all()` **MUST** be an instance of `App\Services\Tasks\TaskInterface`. This is enforced by the type hint on `TaskRegistry::register(string $name, TaskInterface $task)`.

### S007-SC-003 All tasks return arrays [P1]

Every built-in task's `execute()` method **MUST** return an associative `array`. While `TaskInterface::execute()` declares `mixed` as the return type, the built-in implementations **MUST** always return arrays to support `saveAs` context variable storage and downstream template access.

### S007-SC-004 All tasks expose valid metadata [P1]

Every built-in task's `getMetadata()` **MUST** return an array containing at minimum the keys `name` (string), `description` (string), `args` (array), and `effects` (array). The `name` value **MUST** match the registration key used in `AppServiceProvider`.

### S007-SC-005 Required args throw on missing [P1]

For every task argument declared with `required: true` in metadata, calling `execute()` without that argument **MUST** throw `InvalidArgumentException`. Specifically:
- `files.write` without `path` throws
- `files.writeMd` without `path` throws
- `git.clone` without `repo` throws
- `coderabbit.review` without `target` throws

### S007-SC-006 File tasks create parent directories [P1]

Both `files.write` and `files.writeMd` **MUST** create any missing parent directories (via `mkdir` with `recursive: true`) before writing the file. This **MUST** work for paths nested multiple levels deep where none of the intermediate directories exist.
