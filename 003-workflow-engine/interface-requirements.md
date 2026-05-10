# S003 Interface Requirements

## Workflow YAML Schema

S003-IF-001 [P1] A workflow file **MUST** conform to the following YAML structure:

```yaml
description: Optional workflow description string
inputs:
  - name: inputName
    description: Description shown to user
    required: true          # optional, default false
    default: fallbackValue # optional
outputs:                   # optional
  - name: outputName
    description: Description of output
steps:
  - name: Step display name
    method: pack.method.name  # mutually exclusive with task and shell
    task: task-name           # mutually exclusive with method and shell
    shell: "command string"   # mutually exclusive with method and task
    with:                     # arguments passed to method/task; shell ignores this
      argName: value
      anotherArg: "{{templateVariable}}"
    saveAs: variableName      # optional; captures step return value
```

S003-IF-001.a The `steps` array **MUST NOT** be empty. An empty `steps` array **MUST** be treated as valid but produce zero executions.

S003-IF-001.b The `inputs` array **MAY** be omitted. If omitted, the workflow accepts zero required inputs.

## Step Type Constraints

S003-IF-002 [P1] Each step **MUST** contain exactly one of `method`, `task`, or `shell`. A step containing more than one of these keys **MUST** throw `RuntimeException` with message `"Step {N} must have exactly one type key"`.

S003-IF-002.a A step **MUST** contain at least one of `method`, `task`, or `shell`. A step with none of these keys **MUST** throw `RuntimeException` with message `"Step {N} has no type key"`.

## Input Field Definitions

S003-IF-003 [P1] Each input definition **MUST** contain:

- `name` -- required; must be a valid PHP variable name (alphanumeric + underscore, not starting with a digit)
- `description` -- optional; a human-readable string shown during interactive prompts
- `required` -- optional boolean, defaults to `false`
- `default` -- optional; used when input is not provided and `required` is `false`

## Run Command Signature

S003-IF-004 [P1] The `sir run` command signature **MUST** be:

```
sir run <workflow-name> [input-key=input-value ...] [options]
```

**Options:**
- `--dry-run` -- preview without execution
- `--non-interactive` -- skip all prompts
- `--yes` -- skip all confirmation prompts

## Plan Command Signature

S003-IF-005 [P2] The `sir plan` command signature **MUST** be:

```
sir plan <workflow-name>
```

This command **MUST** display step index, type, name, and `saveAs` variable for each step, followed by a summary line showing total step count.

## Workflows:List Command Signature

S003-IF-006 [P2] The `sir workflows:list` command signature **MUST** be:

```
sir workflows:list [--format=table|json]
```

**Output columns:** `Name`, `Type`, `Layer`

## Workflows:Show Command Signature

S003-IF-007 [P2] The `sir workflows:show` command signature **MUST** be:

```
sir workflows:show <workflow-name> [--format=table|json]
```

## WorkflowRunner Return Structure

S003-IF-008 [P1] `WorkflowRunner::run()` **MUST** return an associative array with keys:

- `success` (bool) -- true if all steps completed without exception
- `stepCount` (int) -- total number of steps executed
- `elapsedSeconds` (float) -- total wall-clock time for execution
- `steps` (array) -- array of step result arrays, each containing:
  - `index` (int)
  - `name` (string)
  - `type` (string)
  - `success` (bool)
  - `elapsedSeconds` (float)
  - `output` (mixed, optional)

## Shell Step Return Structure

S003-IF-009 [P1] A shell step **MUST** return an associative array with keys:

- `exitCode` (int) -- process exit code
- `output` (string) -- concatenated stdout
- `errorOutput` (string) -- concatenated stderr
- `success` (bool) -- true if exitCode equals 0