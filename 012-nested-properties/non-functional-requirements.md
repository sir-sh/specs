# S012 Non-Functional Requirements

## Resolution Performance

S012-NF-001 [P2] The system **MUST** resolve a template string containing N distinct `{{expression}}` tokens (each may be simple or nested) in O(N) time relative to the number of tokens, not O(N^2).

S012-NF-002 [P2] The system **SHOULD** complete resolution of a single template string within 10 milliseconds for typical workflows (up to 50 tokens, values up to 1 KB each), including nested property access paths with up to 5 segments.

S012-NF-003 [P3] The system **SHOULD NOT** allocate more than 3x the input string size in peak memory during a single `Context::resolveTemplate()` call, even when the template contains many repeated nested paths.

## Backward Compatibility

S012-NF-004 [P1] Existing workflow YAML files that use only flat variable expressions (`{{var}}`, `{{shellQuote(var)}}`, `{{default(var, 'fallback')}}`) **MUST** behave identically to before this spec. No change in behavior.

S012-NF-005 [P2] The resolution algorithm **MUST** be deterministic: the same `Context` and template string **MUST** always produce the same output, regardless of prior resolution calls or cache state.

## Security

S012-NF-006 [P1] Nested property access **MUST NOT** introduce any new shell injection vectors. All shell-quoting behavior of `shellQuote()` **MUST** remain identical regardless of whether the argument is flat or nested.

S012-NF-007 [P2] The system **MUST NOT** expose any internal object structure (e.g., class property names, private properties) through nested property access unless those properties are explicitly part of the saved result data.

## Error Tolerance

S012-NF-008 [P3] The system **MUST NOT** enter an infinite loop or stack overflow when processing a template with a path containing more than 20 segments (malicious or accidental). Paths with more than 20 segments **SHOULD** resolve to empty string after the 20th segment is processed.

S012-NF-009 [P3] The system **SHOULD** handle circular reference structures (if an object references itself) gracefully by returning empty string if a path segment revisits an already-visited object reference within the same traversal.

## Memory Stability

S012-NF-010 [P3] The resolution cache (if implemented per S012-FR-013) **MUST** be cleared at the start of each `resolveTemplate()` call to prevent unbounded cache growth across many template resolutions in a single workflow run.

S012-NF-011 [P3] Cache entries **SHOULD** be stored using the full path string as the key (e.g., `"a.b.c"`) to allow simple string comparison without complex object serialization.