# S002 Success Criteria

| ID | Criterion | Priority |
|----|-----------|----------|
| S002-SC-001 [P1] | `loadConfig()` returns a non-null array (empty or populated) for every call, including when no config files exist. | P1 |
| S002-SC-002 [P1] | Nearest-layer values override parent-layer values for all scalar and array keys after a three-layer deep merge. | P1 |
| S002-SC-003 [P1] | Both `config.yml` and `config.json` are loaded and merged within the same layer, with JSON values taking precedence over YAML values for the same keys. | P1 |
| S002-SC-004 [P1] | `get($config, 'database.host')` returns the correct nested value for any depth of dot notation up to 10 levels. | P1 |
| S002-SC-005 [P1] | `get($config, 'nonexistent.key', 'default')` returns `'default'` when no part of the path exists. | P1 |
| S002-SC-006 [P1] | All edge case scenarios in S002-EC-001 through S002-EC-009 produce the expected behavior described in edge-cases.md. | P2 |
| S002-SC-007 [P2] | Config loading completes in under 500ms for configurations with up to 10 layers and 100 total keys, measured at runtime. | P2 |
| S002-SC-008 [P2] | Memory usage during `loadConfig()` does not exceed 5MB for configurations with up to 10 layers and 100 total keys. | P2 |