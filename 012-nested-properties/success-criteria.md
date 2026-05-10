# S012 Success Criteria

## Template Resolution Correctness

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S012-SC-001 | All existing Context tests pass | `vendor/bin/phpunit tests/ContextTest.php` reports 100% green. No regression in existing behavior. |
| S012-SC-002 | Simple dot-notation resolves correctly | `Context` with `workspace = ["root" => "/tmp/ws"]` resolves `{{workspace.root}}` to `"/tmp/ws"`. |
| S012-SC-003 | Deep nesting resolves correctly | `Context` with `data = ["user" => ["profile" => ["name" => "Alice"]]]` resolves `{{data.user.profile.name}}` to `"Alice"`. |
| S012-SC-004 | Array index access resolves correctly | `Context` with `items = ["a", "b", "c"]` resolves `{{items.0}}` to `"a"` and `{{items.2}}` to `"c"`. |
| S012-SC-005 | Missing nested property resolves to empty string | `Context` with `data = ["key" => "val"]` resolves `{{data.missing}}` to `""` and `{{data.missing.deep}}` to `""`. No exception thrown. |
| S012-SC-006 | Mixed flat and nested variables resolve correctly | `Context` with `base = "/app"` and `ws = ["root" => "/tmp"]` resolves `{{base}}{{ws.root}}` to `"/app/tmp"`. |

## Function Integration

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S012-SC-007 | shellQuote works with nested properties | `Context` with `file = ["name" => "my file.txt"]` resolves `{{shellQuote(file.name)}}` to `'my file.txt'` (escapeshellarg output). |
| S012-SC-008 | default works with nested properties | `Context` with `cfg = ["path" => "/tmp"]` resolves `{{default(cfg.path, "/default")}}` to `"/tmp"`. With missing path `{{default(cfg.missing, "/default")}}` resolves to `"/default"`. |

## Workflow Integration

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S012-SC-009 | Step result accessible via nested property after saveAs | Workflow with task returning `{"path": "/ws"}` and `saveAs: clone` allows `{{clone.path}}` in subsequent step arguments. |
| S012-SC-010 | Nested properties resolve in shell step commands | Shell step with command `"ls {{clone.path}}"` resolves correctly to `"ls /ws"`. |
| S012-SC-011 | Nested properties resolve in method step with args | Method step with `with: { folder: "{{clone.path}}/repo" }` resolves to `folder = "/ws/repo"`. |

## Error Handling

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S012-SC-012 | Out-of-bounds array index resolves to empty string | `Context` with `items = ["a", "b"]` resolves `{{items.99}}` to `""`. No exception. |
| S012-SC-013 | Scalar intermediate value terminates path gracefully | `Context` with `val = "text"` resolves `{{val.property}}` to `""`. No exception. |
| S012-SC-014 | Null intermediate value resolves to empty string | `Context` with `val = null` resolves `{{val.property}}` to `""`. No exception. |

## Backward Compatibility

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S012-SC-015 | Flat variable without dots behaves identically to v1 | `Context` with `name = "World"` resolves `{{name}}` to `"World"`. Identical to v1 behavior. |
| S012-SC-016 | shellQuote on flat variable unchanged | `Context` with `file = "my file.txt"` resolves `{{shellQuote(file)}}` to `'my file.txt'`. Identical to v1. |
| S012-SC-017 | default on flat variable unchanged | `Context` with `missing = null` resolves `{{default(missing, "fallback")}}` to `"fallback"`. Identical to v1. |

## Performance

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S012-SC-018 | Template resolution remains O(N) in token count | Resolution of N distinct `{{expression}}` tokens completes in O(N) time relative to N. |
| S012-SC-019 | Simple variable resolution not slowed by nested path support | Resolution of `{{var}}` (flat, no dots) takes the same time as before S012. |
| S012-SC-020 | Repeated nested paths within same template use cache | In a template with `{{a.b.c}}` appearing 3 times, the path traversal occurs once; subsequent occurrences use cached result. |