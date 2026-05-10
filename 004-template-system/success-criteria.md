# S004 Success Criteria

## Template Resolution Correctness

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S004-SC-001 | All existing `ContextTest` unit tests pass | `vendor/bin/phpunit tests/ContextTest.php` reports 100% green, no skips or failures. |
| S004-SC-002 | Simple variable substitution is correct | Given `{{name}}` with `name` set to `"World"`, resolved value equals `"World"`. |
| S004-SC-003 | Missing variable resolves to empty string | Given `{{missingVar}}` with no such variable in Context, resolved value equals `""` (zero-length string). |
| S004-SC-004 | Multiple expressions in one string resolve independently | `"Hello {{first}} and {{last}}"` with `first=Alice`, `last=Bob` resolves to `"Hello Alice and Bob"`. |
| S004-SC-005 | Whitespace inside expression delimiters is tolerated | `"{{ name }}"` and `{{name}}` resolve to the same result for variable `name`. |

## Security: Shell Injection Prevention

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S004-SC-006 | shellQuote wraps values in single quotes via `escapeshellarg` | Given `file` = `"test file.txt"`, `{{shellQuote(file)}}` produces `'test file.txt'` (escapeshellarg output). |
| S004-SC-007 | shellQuote prevents word-splitting on spaces | `Process::fromShellCommandline("cat {{shellQuote(file)}}")` with `file` = `"my file.txt"` does not split into two arguments. Result is a single `cat` argument. |
| S004-SC-008 | shellQuote handles special shell metacharacters safely | Value containing `` $HOME ``, backticks, semicolons, newlines produces a quoted string where the content is treated as a literal argument, not evaluated. |
| S004-SC-009 | Unquoted template injection cannot execute arbitrary shell commands | A workflow step argument set to `"{{userInput}}"` where `userInput` is ``; rm -rf / `` resolves to literal string and does not cause command injection when passed to `Process`. |

## Error Handling

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S004-SC-010 | Missing variable does not throw an exception | Calling `Context::resolve("{{missing}}")` returns `""` and emits no PHP warning or notice. |
| S004-SC-011 | Non-string `with` arguments bypass template resolution | Integer `42`, boolean `true`, array `[]`, and `null` values in the `with` map pass through `resolveTemplate` unchanged (not resolved). |
| S004-SC-012 | Malformed expression (unclosed or empty) does not throw | Strings containing `{{unclosed` or `{{}}` resolve to themselves with no exception. |

## Performance

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S004-SC-013 | Template resolution completes in linear time relative to string length | Resolution of a single template string is O(n) in characters, where n is the combined length of the template and variable values. |
| S004-SC-014 | Resolution of 100 variables in a single context does not cause memory explosion | Memory usage during `Context::resolve()` on a string with 100 distinct `{{var}}` references stays under 2x the size of the input string. |