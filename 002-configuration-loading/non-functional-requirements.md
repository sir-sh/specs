# S002 Non-Functional Requirements

### Performance

S002-NF-001 [P1] `loadConfig()` **MUST** complete in under 500ms for configurations with up to 10 layers and 100 total configuration keys across all layers.

S002-NF-002 [P2] `loadConfig()` **SHOULD** complete in under 1s for configurations with up to 20 layers and 500 total configuration keys.

S002-NF-003 [P1] Memory usage during `loadConfig()` **MUST NOT** exceed 5MB for configurations with up to 10 layers and 100 total keys.

### Config File Format Detection

S002-NF-004 [P1] The system **MUST** detect and load `config.yml` using the Symfony YAML parser via `Yaml::parseFile`.

S002-NF-005 [P1] The system **MUST** detect and load `config.json` using PHP's `json_decode` with associative array mode enabled.

S002-NF-006 [P2] The system **SHOULD** detect config file encoding as UTF-8 and handle BOM markers gracefully.

### Error Handling Robustness

S002-NF-007 [P1] The system **MUST** treat a missing config file at a layer as a non-fatal condition and continue processing remaining layers.

S002-NF-008 [P1] The system **MUST** treat a malformed YAML file as a non-fatal condition, skip that file, and log a warning.

S002-NF-009 [P1] The system **MUST** treat a malformed JSON file as a non-fatal condition, skip that file, and log a warning.

S002-NF-010 [P2] When a config file parses to a non-array value (scalar, `null`, `false`), the system **SHOULD** log a warning and skip the file without affecting the merged result.

### Merge Strategy Correctness

S002-NF-011 [P1] The system **MUST** process layers in reverse order (global first, nearest last) using `array_merge_recursive()` to achieve nearest-layer-wins semantics.

S002-NF-012 [P1] Numeric-indexed arrays **MUST** be concatenated across layers, not replaced.

S002-NF-013 [P2] When the same associative key maps to scalar values in both layers, the result **SHOULD** be an array containing both values (parent first, child second).

### Dot-Notation Robustness

S002-NF-014 [P1] `get()` **MUST** return the `$default` value when any segment of the dot-notation path does not exist.

S002-NF-015 [P1] `get()` **MUST** return the `$default` value when an intermediate value in the path is not an array.

S002-NF-016 [P3] The system **SHOULD** handle keys that contain literal dot characters by treating the full key as a single segment (no escaping mechanism required for v1).