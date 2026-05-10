# S001 Success Criteria

S001-SC-001 [P1] **Layer ordering**: For any input path, the returned array **MUST** be ordered such that for indices `i < j`, the layer at index `i` is an equal or deeper filesystem path than the layer at index `j` (nearest first, farthest last). The global `~/.sir` layer, when appended post-walk, is always at the final index.

S001-SC-002 [P1] **No duplicate paths**: The returned array **MUST** contain no duplicate entries. `count(array_unique($layers)) === count($layers)` holds true for every invocation.

S001-SC-003 [P1] **Global layer position**: When the global `~/.sir` directory exists and is not discovered during the walk, it **MUST** appear as the last element of the result array. When it is discovered during the walk, it **MUST** remain at its walk-order position and not be appended again.

S001-SC-004 [P1] **Absolute paths**: Every path in the result array **MUST** be an absolute path (starts with `/`). No relative paths or paths containing `.` or `..` segments are permitted.

S001-SC-005 [P2] **Resolution performance**: The `resolve()` method **MUST** complete in under 50ms for directory trees up to 30 levels deep, measured on standard local filesystems (not network mounts).
