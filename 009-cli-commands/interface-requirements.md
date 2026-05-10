# S009 Interface Requirements

## Command Signatures

### Workflow Execution

```
sir run <workflow> [--dry-run] [--non-interactive] [--yes]
```

**Description**: Execute a named workflow.

**Arguments**:
- `workflow` (required) - Name of the workflow to execute

**Options**:
- `--dry-run` - Show plan without executing
- `--non-interactive` - Skip all interactive prompts
- `--yes` - Auto-confirm all prompts

**Success Output** (exit 0):
```
Running workflow: {name}
  ✓ Step 1: {step_name} ({step_type})
  ✓ Step 2: {step_name} ({step_type})
Workflow completed successfully.
```

**Error Output** (exit 1):
```
Workflow '{name}' not found.
Run 'sir workflows:list' to see available workflows.
```

---

```
sir plan <workflow>
```

**Description**: Show execution plan without running.

**Arguments**:
- `workflow` (required) - Name of the workflow to plan

**Output** (exit 0):
```
Plan for workflow: {name}

Steps:
  1. [{step_type}] {step_name}
     {step_description if available}
  2. [{step_type}] {step_name}
     {step_description if available}
```

**Error Output** (exit 1):
```
Workflow '{name}' not found.
```

---

```
sir where
```

**Description**: Show resolved `.sir/` layers in precedence order.

**Output** (exit 0):
```
Resolved .sir/ layers (highest precedence first):

  1. /path/to/project/.sir/         (project)
  2. ~/.sir/                        (global)

Total: 2 layers
```

---

```
sir workflows:list
```

**Description**: List all available workflows.

**Output** (exit 0):
```
Available workflows:

  Name                Type      Layer
  ------------------  --------  ----------------
  deploy              method    project
  hello               shell     global
  backup              task      project

Total: 3 workflows
```

---

```
sir workflows:show <workflow>
```

**Description**: Show detailed information about a workflow.

**Arguments**:
- `workflow` (required) - Name of the workflow

**Output** (exit 0):
```
Workflow: deploy

Inputs:
  Name         Required  Default  Description
  ----------   --------  -------  -------------------------
  environment  yes       -        Target environment
  verbose      no        false    Enable verbose output

Steps:
  1. [shell]   build
     Runs: build.sh --env={environment}
  2. [method]  deploy
     Class: App\Deployer
     Method: deploy
  3. [task]    notify
     Task: slack:notify
```

**Error Output** (exit 1):
```
Workflow '{name}' not found.
```

---

### Pack Management

```
sir recipes:install <source> [--project]
```

**Description**: Install a pack from a git source.

**Arguments**:
- `source` (required) - Git URL or shorthand (e.g., `user/pack-name`)

**Options**:
- `--project` - Install to project-level `.sir/packs/`

**Success Output** (exit 0):
```
Installing pack from: {source}
  Cloning... done
  Installing dependencies... done
Pack '{id}' installed successfully.
```

**Error Output** (exit 1):
```
Failed to install pack from: {source}
{error_details}
```

---

```
sir recipes:list [--project]
```

**Description**: List installed packs.

**Options**:
- `--project` - Show only project-level packs

**Output** (exit 0):
```
Installed packs:

  ID                Source                           Installed
  ----------------  ------------------------------  ----------------
  deploy-tools      https://github.com/user/packs   2026-04-20
  notify-slack      https://github.com/team/slack   2026-04-18

Total: 2 packs
```

---

```
sir recipes:remove <pack-id> [--yes]
```

**Description**: Remove an installed pack.

**Arguments**:
- `pack-id` (required) - Identifier of the pack to remove

**Options**:
- `--yes` - Skip confirmation

**Success Output** (exit 0):
```
Pack '{id}' removed successfully.
```

**Confirmation Prompt**:
```
Are you sure you want to remove pack '{id}'? [y/N]
```

**Cancelled Output** (exit 0):
```
Removal cancelled.
```

**Error Output** (exit 1):
```
Pack '{id}' not found.
```

---

```
sir recipes:update
```

**Description**: Update installed packs (stubbed in v1).

**Output** (exit 0):
```
This feature is not yet implemented.
See S013 for pack update functionality.
```

---

### AI Agent Commands

```
sir agent tools
```

**Description**: List available tools for AI agent consumption.

**Output** (exit 0) - JSON format:
```json
{
  "workflows": [
    {
      "name": "deploy",
      "type": "method",
      "layer": "project",
      "inputs": [
        {"name": "environment", "required": true}
      ]
    }
  ],
  "methods": [
    {"class": "App\\Deployer", "method": "deploy", "description": "Deploys the application"}
  ],
  "tasks": [
    {"name": "notify", "description": "Sends notifications"}
  ]
}
```

---

```
sir agent plan <workflow>
```

**Description**: Show plan for AI agent (delegates to `sir plan`).

**Output**: Same as `sir plan <workflow>`

---

```
sir agent run <workflow> [--yes]
```

**Description**: Run workflow with AI agent confirmation flow.

**Arguments**:
- `workflow` (required) - Name of the workflow

**Options**:
- `--yes` - Auto-confirm

**Output**:
```
Plan for workflow: {name}

[Steps as in sir plan]

Confirm execution? [y/N]
```

---

### Laravel Zero Commands

```
sir list
```

**Description**: List all available sir commands (auto-generated).

**Output**: Standard Laravel Zero command list format.

---

### Error Output Format

All error messages follow this format:

```
ERROR: {message}
{optional: suggestion or next step}
```

Examples:
```
ERROR: Workflow 'nonexistent' not found.
Run 'sir workflows:list' to see available workflows.

ERROR: Missing required input: environment
Use --non-interactive to skip input prompts.
```

---

### Confirmation Prompt Format

```
{icon} {message} [y/N]
```

Examples:
```
? Are you sure you want to remove pack 'old-pack'? [y/N]
! Warning: This action cannot be undone. Continue? [y/N]
```

---

### Input Gathering Prompt Format

```
{input_name} ({required/optional}) [{default}]:
```

Example:
```
environment (required) []:
```