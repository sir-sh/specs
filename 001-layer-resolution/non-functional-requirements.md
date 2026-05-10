# S001 Non-Functional Requirements

## Performance

S001-NF-001 [P1] The `resolve()` method **MUST** complete within 50ms for directory hierarchies up to 30 levels deep on a local filesystem. The method performs only `is_dir()` checks and `dirname()` calls, with no file reads, network I/O, or recursive directory scans.

## Reliability

S001-NF-002 [P1] The `resolve()` method **MUST NOT** perform any filesystem writes. It is a pure read/query operation. Only `ensureGlobalSirExists()` may write to the filesystem.

S001-NF-003 [P2] If `realpath()` returns `false` (e.g., the provided `$startPath` does not exist), the resolver **SHOULD** handle the failure gracefully. The current implementation relies on the `while ($current && ...)` guard to exit the loop when `$current` is falsy.

## Portability

S001-NF-004 [P1] The `LayerResolver` **MUST NOT** depend on external packages or services. It uses only PHP built-in functions: `getcwd()`, `realpath()`, `dirname()`, `is_dir()`, `getenv()`, `mkdir()`, and `in_array()`.

S001-NF-004.a The home directory detection **MUST** support both Unix (`HOME`) and Windows (`USERPROFILE`) environment variables, checked in that order.
