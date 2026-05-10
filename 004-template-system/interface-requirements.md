# S004 Interface Requirements

## Template Expression Syntax

### Form 1: Simple Variable Substitution

```
{{variableName}}
```

The expression is delimited by `{{` (opening) and `}}` (closing). The content between these delimiters is the variable name. Whitespace inside the delimiters is trimmed before lookup.

**Examples:**
- `{{name}}` -- looks up variable `name` in the current `Context`
- `{{ name }}` -- same as above (whitespace trimmed to `name`)
- `{{  name  }}` -- same as above (whitespace trimmed to `name`)

**Variable name rules:**
- Valid characters: alphanumeric (`a-z`, `A-Z`, `0-9`) and underscore (`_`)
- First character **MUST NOT** be a digit
- Case-sensitive: `Name` and `name` are distinct variables
- No dot notation, no bracket notation in v1 simple form

**Resolution:** The variable's value is cast to string via PHP `(string)` and substituted in place of the entire expression.

**Behavior when missing:** Returns empty string `""`. No exception, no warning.

---

### Form 2: Shell Quote Function

```
{{shellQuote(variableName)}}
```

The `shellQuote()` function wraps the named variable's value in PHP's `escapeshellarg()`, producing a shell-safe single-quoted literal.

**Argument:** Must be a simple variable name. Nested paths and function calls are not supported.

**Examples:**
- `{{shellQuote(file)}}` -- with `file` = `"my file.txt"` produces `'my file.txt'`
- `{{shellQuote(cmd)}}` -- with `cmd` = `"$HOME"` produces `'$HOME'` (literal)

**Behavior when variable is undefined:** Returns `''` (empty string, shell-quoted).

**Security property:** The output is safe to embed in a shell command passed to `Process::fromShellCommandline()` without word-splitting or command injection.

---

### Form 3: Default Fallback Function

```
{{default(variableName, 'fallback')}}
{{default(variableName, "fallback")}}
{{default(variableName, fallback)}}
```

The `default()` function returns the variable's value if it exists and is non-empty, otherwise returns the fallback literal.

**Arguments:**
1. `variableName` -- must be a simple variable name (required)
2. `fallback` -- a literal value, optionally quoted. Single quotes, double quotes, and surrounding whitespace are trimmed from the fallback.

**Examples:**
- `{{default(missing, 'fallback')}}` -- with no `missing` variable resolves to `fallback`
- `{{default(missing, "default value")}}` -- resolves to `default value`
- `{{default(count, 0)}}` -- resolves to `0` (integer fallback; unquoted `0` becomes string `0`)

**Behavior when variable is defined but empty string:** Per S004-FR-004.b, the fallback should be returned. **Current v1 implementation note:** The implementation returns the empty string rather than the fallback when the variable exists but is `""`. This is a known deviation from the intended spec.

**Comma inside fallback:** The comma in `{{default(v, 'a,b')}}` is part of the fallback literal after quote stripping. No parsing of comma as argument separator inside the fallback value.

---

### Escaped Delimiters

```
{{\variable}}
```

A backslash before `{{` or `}}` is treated as a literal backslash character, not an escape sequence. The backslash is retained, and the `{{` or `}}` is not treated as a template expression delimiter.

**Example:** `{{\name}}` is interpreted as literal `{{\name}}` -- no substitution occurs.

---

## Invalid and Edge-Case Expressions

| Expression | Behavior |
|------------|----------|
| `{{}}` | Empty expression. No substitution; returns `{{}}` literally. |
| `{{unclosed` | Unclosed expression. No substitution; returns `{{unclosed` literally. |
| `{{{name}}}` | Extra opening brace. No substitution on the inner `{{name}}`; the outer `{` is literal. Result: `{{name}}` literally. |
| `{{ }}` | Whitespace-only inside delimiters. Variable name is empty after trim; resolves to empty string; net result is `{{ }}` unchanged (empty string substituted for empty name). |

---

## Call-Site API

### `Context::resolve(string $template): string`

Replaces all `{{...}}` expressions in `$template` with resolved values from the current Context variable store.

**Parameters:**
- `$template` -- the input string containing zero or more template expressions

**Returns:** The resolved string with all expressions substituted

**Throws:** Does not throw for missing variables or malformed expressions

---

### `Context::setVariables(array $vars): void`

Merges `$vars` into the variable store. Existing keys are overwritten.

---

### `Context::setInputs(array $inputs): void`

Merges `$inputs` into the variable store. Inputs are accessible as `{{inputName}}`.

---

### `WorkflowRunner::resolveWithArguments(array $with): array`

Applies `Context::resolve()` to every string value in `$with`. Non-string values pass through unchanged.

**Parameters:**
- `$with` -- associative array of step arguments

**Returns:** Associative array with strings resolved, non-strings unchanged

---

## Supported Characters in Variable Names

```
[a-zA-Z_][a-zA-Z0-9_]*
```

Examples of valid names: `name`, `userName`, `user_name`, `NAME`, `_private`, `var2`, `myVar_foo`

Examples of invalid names: `2nd` (starts with digit), `my-var` (contains hyphen), `my.var` (contains dot)

---

## Summary of Supported Expressions

| Form | Syntax | Missing Variable Behavior |
|------|--------|-------------------------|
| Simple | `{{name}}` | Empty string |
| shellQuote | `{{shellQuote(name)}}` | Returns `''` (empty quoted) |
| default | `{{default(name, 'fallback')}}` | Returns fallback literal |