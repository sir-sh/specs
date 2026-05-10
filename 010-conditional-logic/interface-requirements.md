# S010 Interface Requirements

## Step YAML Schema Additions

S010-IF-001 [P1] The step definition schema defined in S003-IF-001 **MUST** be extended to include the following additional properties:

```yaml
step:
  # Existing properties
  name: string
  method: string              # mutually exclusive with task and shell
  task: string                # mutually exclusive with method and shell
  shell: string               # mutually exclusive with method and task
  with: object                # arguments passed to method/task
  saveAs: string              # captures step return value

  # New properties
  if: string                 # optional; condition expression evaluated before execution
  continueOnError: bool      # optional; default false; do not halt workflow on step failure
  timeout: int                # optional; max execution time in seconds; 0 means no limit
```

S010-IF-001.a A step **MAY** contain any subset of `if:`, `continueOnError:`, and `timeout:` in any combination with its type key (`method`, `task`, or `shell`).

S010-IF-001.b All three new properties **MUST** be optional. Steps without any of these properties execute unconditionally, without timeout, and with normal error handling.

## Condition Expression Operator Reference

S010-IF-002 [P1] The `ConditionEvaluator` **MUST** support the operators defined in the table below. All operators **MUST** be used within a template expression string (e.g., `if: "{{var}} == 'value'"`).

| Category | Operator | Syntax | Description |
|----------|----------|--------|-------------|
| Comparison | `==` | `{{a}} == {{b}}` | Value equality; string-aware |
| Comparison | `!=` | `{{a}} != {{b}}` | Value inequality |
| Comparison | `>` | `{{a}} > {{b}}` | Greater than (numeric) |
| Comparison | `<` | `{{a}} < {{b}}` | Less than (numeric) |
| Comparison | `>=` | `{{a}} >= {{b}}` | Greater than or equal (numeric) |
| Comparison | `<=` | `{{a}} <= {{b}}` | Less than or equal (numeric) |
| Logical | `&&` | `{{a}} && {{b}}` | Logical AND |
| Logical | `\|\|` | `{{a}} \|\| {{b}}` | Logical OR |
| Logical | `!` | `!{{a}}` | Logical NOT (prefix) |
| String | `contains` | `{{str}} contains 'sub'` | Substring containment |
| String | `startsWith` | `{{str}} startsWith 'pre'` | Prefix match |
| String | `endsWith` | `{{str}} endsWith 'suf'` | Suffix match |
| String | `matches` | `{{str}} matches 'pattern'` | PCRE regex match |
| Existence | `exists` | `exists varName` | Variable is defined in Context |
| Existence | `isEmpty` | `isEmpty varName` | Variable is defined and equals `""` |

S010-IF-002.a Template expressions inside `{{ }}` **MUST** be resolved to their string values before operator evaluation.

S010-IF-002.b The `exists` and `isEmpty` operators **MUST NOT** use `{{ }}` around the variable name; they take a bare variable name as their operand.

S010-IF-002.c The `matches` operator **MUST** treat the right-hand operand as a PCRE regular expression and **MUST** return `false` (with a warning) if the pattern is invalid.

## WorkflowRunner Return Structure Update

S010-IF-003 [P1] The `WorkflowRunner::run()` return structure defined in S003-IF-008 **MUST** be extended so that each step result array includes:

- `skipped` (bool, optional) -- true when the step was skipped due to condition evaluation
- `reason` (string, optional) -- present when `skipped` is true; one of `"Condition not met"`, `"Invalid condition syntax"`, or `"Timeout exceeded"`
- `continueOnError` (bool, optional) -- present when the step had `continueOnError: true`

## Skipped Step Return Structure

S010-IF-004 [P1] A skipped step **MUST** produce a step result array with the following structure:

```php
[
  'index' => int,
  'name' => string,
  'type' => string,
  'success' => true,
  'skipped' => true,
  'reason' => "Condition not met" | "Invalid condition syntax" | "Timeout exceeded",
  'elapsedSeconds' => float,
]
```

S010-IF-004.a The `success` field **MUST** be `true` for skipped steps so that they do not cause the overall workflow result `success` to become `false`.
