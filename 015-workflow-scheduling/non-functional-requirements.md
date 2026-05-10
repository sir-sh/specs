# S015 Non-Functional Requirements

## Missed Run Detection

S015-NF-001 [P1] The daemon **MUST** monitor for missed runs on startup by comparing the last-executed timestamp for each schedule against the cron expression. Any intervals that passed while the daemon was stopped **MUST** be executed before normal scheduling resumes.

## Persistence

S015-NF-002 [P1] Schedule state **MUST** be persisted in `.sir/schedule_history.json` in the project layer directory, containing: schedule name, start/end timestamps, exit code, duration, attempt number, and process ID.

S015-NF-002.a The history file **MUST** be append-only during normal operation to minimize I/O.

S015-NF-002.b Entries older than 30 days **MUST** be pruned on daemon startup to bound file size.

S015-NF-007 [P1] Execution history **MUST** be retained for at least 30 days.

## Error Notifications

S015-NF-003 [P1] Notifications **MUST** be delivered on workflow failure (exit code != 0) and on retry exhaustion. Notifications **MAY** be sent on success if configured.

## Reliability

S015-NF-004 [P1] The daemon **MUST** auto-restart via systemd service with `Restart=always` so that it recovers from crashes without manual intervention.

S015-NF-004.a The systemd unit **MUST** be installed at `/etc/systemd/system/sir-scheduler.service` with `After=network.target`.

## Graceful Shutdown

S015-NF-005 [P2] The daemon **MUST** handle SIGTERM by waiting for any in-progress workflow execution (up to the configured timeout), writing the final history entry, and exiting with code 0.

S015-NF-005.a If an in-progress workflow exceeds the timeout during shutdown, the daemon **MUST** send SIGKILL and exit with code 2.

## Process Isolation

S015-NF-006 [P2] The daemon **SHOULD** run as a low-privilege user (e.g., `sir`) via the systemd unit's `User=sir` directive.

## Performance

S015-NF-008 [P2] The daemon's memory footprint **SHOULD** stay below 50MB over a 24-hour continuous run.

S015-NF-009 [P2] The schedule evaluation cycle **SHOULD** complete within 1 second for 100 schedules.

## Portability

S015-NF-010 [P2] The daemon **MUST** support Docker container execution via a `php application schedules:run` CMD directive.

S015-NF-011 [P2] Alternative operation via system cron **MUST** be documented: `0 2 * * * cd /path && sir run backup`.

## Security

S015-NF-012 [P1] The scheduler **MUST NOT** execute workflows from layers that the current user does not have read permission on.

S015-NF-013 [P2] Notification credentials (email SMTP, Slack webhook, PagerDuty key) **MUST** be stored in environment variables or a secrets manager, not in `schedules.yml` in plain text.

## Observability

S015-NF-014 [P2] The daemon **SHOULD** emit structured log entries (JSON) to stdout for integration with log aggregation systems (e.g., Datadog, CloudWatch).

S015-NF-015 [P2] A `sir schedules:health` command **MUST** expose: scheduler uptime, active schedule count, last-check timestamp, and next 5 scheduled run times.