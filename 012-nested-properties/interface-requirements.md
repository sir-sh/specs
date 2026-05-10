# S012 Interface Requirements

## Template Expression Syntax (S004 Extension)

This section extends the template expression syntax defined in S004-IF-001 through S004-IF-003.

### Form 1: Simple Variable Substitution (Unchanged)

S012-IF-001 [P1] The syntax `{{variableName}}` is unchanged from S004-IF-001. When the expression contains no dots, it **MUST** be resolved as a top-level variable name in the Context variable store exactly as in v1.

---

### Form 2: Dot-Notation Property Access (New in S012)

S012-IF-002 [P1] The syntax `{{expression}}` where `expression` contains one or more period (`.`) characters **MUST** be treated as a nested property path. Each segment separated by `.` is traversed in order.

S012-IF-003 [P1] When resolving a nested path, the system **MUST** apply the following segment-type rules:
- A segment consisting entirely of decimal digits (`0`-`9`) **MUST** be treated as an array index (integer cast before use).
- Any other segment **MUST** be treated as an object property name (string key).

S012-IF-004 [P1] The resolution algorithm **MUST**:
1. Split expression by `.` into segments.
2. Begin with the root variable (first segment).
3. For each remaining segment:
   a. If current value is an array and segment is a non-negative integer, access by integer index.
   b. Else if current value is an object (stdClass or associative array with string keys), access by string key.
   c. Else stop and return empty string.
4. Cast final value to string via PHP `(string)` and substitute.

S012-IF-005 [P1] When any segment cannot be accessed (property does not exist, array index out of bounds, current value is scalar), the entire expression **MUST** resolve to `""`. No exception or warning may be emitted.

---

### Form 3: shellQuote Function (Extended in S012)

S012-IF-006 [P2] The `shellQuote()` function argument syntax is extended from flat variable names to also accept dot-notation nested paths. Flat variable names **MUST** continue to work exactly as in S004-IF-002.

S012-IF-007 [P2] When `shellQuote()` receives a nested path, the system **MUST** resolve the path first, producing a string value, then pass that value to PHP's `escapeshellarg()`.

**Examples:**
- `{{shellQuote(file)}}` -- `file` is top-level variable; unchanged from S004
- `{{shellQuote(file.name)}}` -- resolve `file.name` first, then `escapeshellarg()`
- `{{shellQuote(a.b.c)}}` -- resolve `a.b.c` first, then `escapeshellarg()`

S012-IF-008 [P2] When the nested path resolves to empty string, `shellQuote()` **MUST** return `''` (empty string, shell-quoted).

---

### Form 4: default Function (Extended in S012)

S012-IF-009 [P2] The `default()` function first-argument syntax is extended from flat variable names to also accept dot-notation nested paths. Flat variable names **MUST** continue to work exactly as in S004-IF-003.

S012-IF-010 [P2] When `default()` receives a nested path, the system **MUST** resolve the path first. If the result is a non-empty string, it **MUST** be returned. Otherwise, the fallback literal **MUST** be returned.

**Examples:**
- `{{default(missing, 'fallback')}}` -- unchanged from S004
- `{{default(config.path, '/default')}}` -- resolve `config.path`; if non-empty return it; else return `/default`
- `{{default(obj.deep.missing, 'a.b')}}` -- resolve path (empty); return literal `a.b` (the dot in the fallback is NOT treated as nested access; fallback literals preserve their content verbatim)

---

## YAML Schema: Step Definition (S003 Extension)

S012-IF-011 [P1] The step definition schema defined in S003-IF-001 is unaffected by S012. Nested property access **MUST** apply to all string values within a step definition, including:
- The `shell` property of a shell step
- Any string value in the `with:` argument map
- The `name` property of a step (if it contains template expressions)

**Example: shell step with nested property**
```yaml
steps:
  - name: list-clone
    task: git.clone
    saveAs: clone
    with:
      url: "{{input.url}}"
  - name: verify
    shell: "ls {{clone.path}}"
    continueOnError: false
```

**Example: method step with nested property in arguments**
```yaml
steps:
  - name: build
    task: workspace.init
    saveAs: ws
  - name: configure
    method: config.set
    with:
      key: "output.path"
      value: "{{ws.root}}/dist"
```

---

## Context API Additions

S012-IF-012 [P1] The system **MUST** add a private or protected method `Context::resolveNestedProperty(string $expression): mixed` that traverses a dot-separated `$expression` against the current Context variable store and returns the resolved value as-is (not yet cast to string). The method **MUST NOT** throw for missing properties; it **MUST** return `null` if the path cannot be fully traversed.

S012-IF-013 [P1] The system **MUST** extend `Context::resolveTemplate(string $template): string` to:
1. Detect whether an expression contains a `.` character.
2. If no dot: behave as in S004 (flat variable lookup).
3. If dot present: call `resolveNestedProperty()` and cast the result to string.

S012-IF-013.a [P1] Backward compatibility: expressions without dots **MUST** continue to behave exactly as before. There **MUST NOT** be any change to the public API surface.

---

## Task saveAs Integration

S012-IF-014 [P1] When a task step returns an object or array via `saveAs`, all top-level keys of that result **MUST** become accessible via dot-notation from subsequent steps.

**Example:**
```yaml
steps:
  - name: clone
    task: git.clone
    saveAs: clone
    with:
      url: "{{input.url}}"
      branch: main
  - name: build
    shell: "cd {{clone.path}} && {{clone.command}} build"
```

In the above, `clone` is the saved result object with properties `path`, `branch`, `url`, `command`, etc. All are accessible via `{{clone.<property>}}`.

S012-IF-014.a [P1] Only top-level keys of the saved result are directly accessible as `{{saveAs.key}}`. Nested access to deeper levels of the result requires explicit dot notation: `{{saveAs.key.subkey}}`.

---

## Interaction with Functions

S012-IF-015 [P2] Functions **MAY** be nested inside each other. Template resolution **MUST** evaluate innermost expressions first.

**Resolution order examples:**

| Expression | Resolution Order |
|------------|-----------------|
| `{{shellQuote(a.b)}}` | 1. Resolve `a.b` to string value. 2. Pass to `escapeshellarg()`. |
| `{{default(a.b, 'x')}}` | 1. Resolve `a.b`. 2. If non-empty return it; else return `x`. |
| `{{shellQuote(default(a.b, 'x'))}}` | 1. Resolve `default(a.b, 'x')`. 2. Pass result to `shellQuote()`. |
| `{{a.b.c.d}}` | 1. Traverse `a` -> `b` -> `c` -> `d`. 2. Return final scalar as string. |

---

## Supported Characters in Variable/Property Names

S012-IF-016 [P1] Top-level variable names **MUST** match `[a-zA-Z_][a-zA-Z0-9_]*` (unchanged from S004).

S012-IF-017 [P1] Property/array-key segment names **MUST** follow:
- String keys: `[a-zA-Z_][a-zA-Z0-9_]*` (same rules as variable name)
- Array index: `[0-9]+` (decimal integer only; no sign, no decimal point)

S012-IF-018 [P1] The following segment patterns **MUST** resolve to `""`:
- Empty segment: `{{a..b}}`
- Leading dot: `{{.workspace}}`
- Trailing dot: `{{workspace.}}`
- Non-integer string on array: `{{items.three}}` where `items` is a plain array
- Negative integer: `{{items.-1}}`

---

## Summary of Expression Forms

| ID | Form | Syntax | Example | Status |
|----|------|--------|---------|--------|
| S012-IF-001 | Simple variable | `{{name}}` | `{{name}}` | Unchanged from S004 |
| S012-IF-002 | Dot-notation | `{{a.b.c}}` | `{{workspace.root}}` | NEW in S012 |
| S012-IF-003 | Array index | `{{arr.0}}` | `{{items.0}}` | NEW in S012 |
| S012-IF-006 | shellQuote (flat) | `{{shellQuote(name)}}` | `{{shellQuote(file)}}` | Unchanged |
| S012-IF-007 | shellQuote (nested) | `{{shellQuote(obj.prop)}}` | `{{shellQuote(file.name)}}` | Extended in S012 |
| S012-IF-009 | default (flat) | `{{default(name, 'x')}}` | `{{default(count, 0)}}` | Unchanged |
| S012-IF-010 | default (nested) | `{{default(obj.prop, 'x')}}` | `{{default(config.path, '/')}}` | Extended in S012 |