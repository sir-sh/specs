# S004 Non-Functional Requirements

## Resolution Performance

S004-NF-001 [P2] The system **MUST** resolve a template string containing N distinct `{{expression}}` tokens in O(N) time relative to the number of tokens, not O(N^2).

S004-NF-002 [P2] The system **SHOULD** complete resolution of a single template string within 10 milliseconds for typical workflows (up to 50 tokens, values up to 1 KB each).

S004-NF-003 [P3] The system **SHOULD NOT** allocate more than 3x the input string size in peak memory during a single `Context::resolve()` call.

## ShellQuote Security

S004-NF-004 [P1] The `shellQuote()` function **MUST** use PHP's `escapeshellarg()` under all circumstances. No custom quoting logic **MAY** be introduced.

S004-NF-005 [P1] The `escapeshellarg()` output **MUST** produce a result that, when passed as a single argument to `Process::fromShellCommandline()`, is treated as one argument (no word-splitting on spaces, no glob expansion, no variable substitution).

S004-NF-006 [P2] The `shellQuote()` function **MUST** handle at minimum: spaces, single quotes, double quotes, backticks, `$`, `;`, `|`, `>`, `<`, newlines, and null bytes.

S004-NF-007 [P2] The `shellQuote()` function **MUST NOT** strip or sanitize content in a way that changes its meaning (e.g., `a b` remains `'a b'`, not `'ab'`).

## Memory and Stability

S004-NF-008 [P3] The system **MUST NOT** enter an infinite loop or stack overflow when processing a template string with more than 1,000 nested `{{...}}` expressions (malicious or accidental).

S004-NF-009 [P3] The system **SHOULD** handle circular variable references gracefully (e.g., `a = "{{b}}"`, `b = "{{a}}"`), resolving to empty string on the second reference to prevent infinite recursion.

## Backward Compatibility

S004-NF-010 [P1] Existing workflow YAML files that do not use template expressions **MUST** behave identically to before this spec (no change in behavior for literal strings without `{{...}}`).

S004-NF-011 [P2] The resolution algorithm **MUST** be deterministic: the same Context and template string **MUST** always produce the same output, regardless of prior resolution calls.