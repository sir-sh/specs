# S015 Interface Requirements

## Schedule Configuration Schema

### File Location
`.sir/schedules.yml` in each layer (project, org, global). Loaded with layer precedence.

### YAML Schema

```yaml
schedules:
  - name: string              # Required. Unique identifier for this schedule.
    workflow: string           # Required. Name of workflow to execute.
    cron: string              # Cron expression (e.g., "0 2 * * *").
    schedule: string          # Natural language (alternative to cron).
    enabled: bool             # Default: true. Set false to disable.
    inputs: object            # Workflow input key-value pairs.
    concurrency: string        # "prevent" | "allow" | "queue". Default: "allow".
    timeout: string           # Duration (e.g., "3600s", "30m"). 0 = no timeout.
    retry:
      attempts: int          # Number of retry attempts. Default: 1 (no retry).
      backoff: string        # "exponential". Default: "exponential".
      delay: string          # Base delay (e.g., "60s"). Default: "60s".
    notifications:
      onSuccess: array       # List of notification specs.
      onFailure: array       # List of notification specs.
    conditions:
      branch: string         # Git branch name to match.
      tag: string            # Git tag pattern to match.
    watch: string            # Filesystem glob pattern for file-watcher triggers.
```

### Notification Spec Format

```yaml
notifications:
  onSuccess:
    - email: admin@example.com
    - slack: "#ops"
    - pagerduty: service-id
  onFailure:
    - email: admin@example.com
    - pagerduty: service-id
```

### Input Interpolation

Inputs support template resolution via `{{variable}}` syntax:

```yaml
inputs:
  target: "production"
  format: "pdf"
  file: "{{event.file}}"   # Set by file-watcher trigger
```

## CLI Commands

### `sir schedules:list`

List all loaded schedules.

**Signature:**
```bash
sir schedules:list [--format=table|json]
```

**Output:**
```
Name             Workflow      Cron             Enabled  Next Run
nightly-backup   backup        0 2 * * *         true     2026-04-25 02:00:00
health-check     monitor       */5 * * * *       true     2026-04-24 15:05:00
```

---

### `sir schedules:add`

Create a new schedule in the nearest layer's `schedules.yml`.

**Signature:**
```bash
sir schedules:add <name> --workflow <workflow> [--cron <expression>] [--schedule <natural-language>] [--enabled] [--inputs <json>] [--concurrency <mode>] [--timeout <duration>] [--retry-attempts <n>] [--retry-delay <duration>]
```

**Arguments:**
- `<name>` — Unique schedule name.

**Options:**
- `--workflow, -w` — Workflow name. Required.
- `--cron, -c` — Cron expression. Required if `--schedule` not given.
- `--schedule, -s` — Natural language schedule. Required if `--cron` not given.
- `--enabled, -e` — Set enabled=true. Default: true.
- `--inputs, -i` — JSON-encoded input map.
- `--concurrency` — `"prevent"` | `"allow"` | `"queue"`. Default: `"allow"`.
- `--timeout, -t` — Duration string (e.g., `"3600s"`).
- `--retry-attempts` — Integer. Default: 1.
- `--retry-delay` — Duration string. Default: `"60s"`.

**Exit codes:**
- `0` — Schedule created.
- `1` — Validation error or workflow not found.

---

### `sir schedules:remove`

Remove a schedule from the nearest layer's `schedules.yml`.

**Signature:**
```bash
sir schedules:remove <name> [--force]
```

**Arguments:**
- `<name>` — Name of schedule to remove.

**Options:**
- `--force, -f` — Skip confirmation prompt.

**Exit codes:**
- `0` — Schedule removed.
- `1` — Schedule not found.

---

### `sir schedules:run`

Start the scheduler daemon (blocking).

**Signature:**
```bash
sir schedules:run [--daemon] [--pid-file <path>]
```

**Options:**
- `--daemon, -d` — Fork to background.
- `--pid-file` — Write daemon PID to file.

**Exit codes:**
- `0` — Clean exit on SIGTERM.
- `1` — Configuration error (invalid schedules.yml).

---

### `sir schedules:test`

Evaluate or execute a named schedule.

**Signature:**
```bash
sir schedules:test <name> [--dry-run] [--execute]
```

**Arguments:**
- `<name>` — Name of schedule to test.

**Options:**
- `--dry-run` — Show due status and resolved inputs without executing.
- `--execute` — Actually execute (default when omitted for backward compat: show dry-run only).

**Output:**
```
Schedule: nightly-backup
Workflow: backup
Cron: 0 2 * * *
Due: Yes (next: 2026-04-25 02:00:00)
Inputs: { "target": "production" }
```

---

### `sir schedules:history`

Show execution history for a schedule.

**Signature:**
```bash
sir schedules:history <name> [--limit <n>] [--format=table|json]
```

**Arguments:**
- `<name>` — Name of schedule.

**Options:**
- `--limit, -n` — Number of recent entries to show. Default: 50. Max: 1000.
- `--format` — Output format. Default: `table`.

**Output:**
```
Name: backup
┌────────────┬──────────────────────┬─────────┬──────────┐
│ Date       │ Time                 │ Status  │ Duration │
├────────────┼──────────────────────┼─────────┼──────────┤
│ 2026-01-12 │ 02:00:15             │ Success │ 45s      │
│ 2026-01-11 │ 02:00:10             │ Success │ 42s      │
│ 2026-01-10 │ 02:00:12             │ Failed  │ 5s       │
└────────────┴──────────────────────┴─────────┴──────────┘
```

---

### `sir schedules:health`

Show scheduler daemon health status.

**Signature:**
```bash
sir schedules:health
```

**Output:**
```
Scheduler Status: Running
Active Schedules: 5
Last Check: 2026-01-12 14:30:00
Next Runs:
  - backup: 2026-01-13 02:00:00
  - health-check: 2026-01-12 14:35:00
```

---

### `sir schedules:validate`

Validate `schedules.yml` files without starting daemon.

**Signature:**
```bash
sir schedules:validate [--layer <path>]
```

**Exit codes:**
- `0` — All schedules valid.
- `1` — One or more validation errors (prints errors to stderr).

## Events / Hooks

### `{{event.file}}` Variable

When a file-watcher trigger activates, the changed file path is interpolated as `{{event.file}}` into the workflow's input resolution context.

### Daemon Lifecycle Events

| Event | Trigger | Action |
|-------|---------|--------|
| `schedule.start` | Execution begins | Log start, update `schedule_history.json` with `pid` |
| `schedule.complete` | Execution ends | Log duration, update history entry |
| `schedule.retry` | Retry triggered | Log retry attempt number |
| `schedule.notification` | Notification dispatched | Log delivery status |