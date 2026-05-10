# S008 Interface Requirements

## Command Signature

### S008-IF-001 [P1] AgentCommand Signature

```
sir agent {action=tools} {workflow?} {--yes}
```

S008-IF-001.a The `action` argument accepts: `tools`, `plan`, `run`. Default value is `tools`.

S008-IF-001.b The `workflow` argument is optional and positional. Required when `action` is `plan` or `run`.

S008-IF-001.c The `--yes` flag is optional. When provided, skips the confirmation prompt in `run`.

## Human-Readable Output Format

### S008-IF-002 [P1] `sir agent tools` Output Format

```
Available Tools (Workflows & Methods):

Workflows:
  • <workflow-name-1>
  • <workflow-name-2>

Methods:
  • <method-name-1> - <description>
  • <method-name-2> - <description>

Built-in Tasks:
  • <task-name-1> - <description>
  • <task-name-2> - <description>
```

S008-IF-002.a Section headings are only printed if the corresponding category has entries.

S008-IF-002.b Method descriptions are omitted from the line if the method has no description.

### S008-IF-003 [P1] `sir agent plan <workflow>` Output Format

Delegates entirely to `PlanCommand`. See S003 interface requirements for PlanCommand output format.

### S008-IF-004 [P1] `sir agent run <workflow>` Confirmation Flow

```
Execution plan for: <workflow>

<plan output>

Execute this workflow? [y/N]
```

S008-IF-004.a If the user answers `y` or `yes` (case-insensitive), execution proceeds.

S008-IF-004.b If the user answers `n`, `no`, or ENTER (default), output is `Cancelled.` and execution stops.

### S008-IF-005 [P1] Error Output Format

Errors are printed via `$this->error('message')` which renders in red:

```
Error: message here
```

S008-IF-005.a Validation errors (missing workflow argument) exit with code 1.

## JSON Output Format

### S008-IF-006 [P2] `sir agent tools --json` JSON Schema

```json
{
  "workflows": [
    {
      "name": "string",
      "description": "string",
      "inputs": [
        {
          "name": "string",
          "type": "string",
          "required": true,
          "description": "string"
        }
      ]
    }
  ],
  "methods": [
    {
      "name": "string",
      "description": "string",
      "inputs": [
        {
          "name": "string",
          "type": "string",
          "required": true,
          "description": "string"
        }
      ]
    }
  ],
  "tasks": [
    {
      "name": "string",
      "description": "string",
      "inputs": [
        {
          "name": "string",
          "type": "string",
          "required": true,
          "description": "string"
        }
      ]
    }
  ]
}
```

S008-IF-006.a All keys are required. Empty categories use empty arrays `[]`.

S008-IF-006.b `description` may be `null` if the item has no description.

S008-IF-006.c `required` is `true` for mandatory inputs, `false` for optional.

### S008-IF-007 [P2] `sir agent plan <workflow> --json` JSON Schema

```json
{
  "workflow": "string",
  "steps": [
    {
      "name": "string",
      "type": "string",
      "effects": {
        "network": true,
        "writes": ["path/pattern"],
        "interactive": false
      }
    }
  ]
}
```

S008-IF-007.a `effects` keys are omitted when the value is `false` or absent.

S008-IF-007.b `type` is one of: `task`, `method`, `shell`, `workflow`.

### S008-IF-008 [P2] `sir agent run <workflow> --json` JSON Schema

No distinct JSON output for run itself. When `--json` is used with `run`, it is passed to the underlying `RunWorkflowCommand`. See S003 interface requirements for run JSON output.

## Help Text Format

### S008-IF-009 [P2] Unknown action / help output

```
Sir Agent - AI Operator Interface

Usage:
  sir agent tools              List available tools
  sir agent plan <workflow>    Show execution plan
  sir agent run <workflow>     Execute with confirmation

The agent interface provides a tool surface for AI assistants
to discover and execute workflows safely with confirmation.
```