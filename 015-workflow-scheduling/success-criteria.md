# S015 Success Criteria

## Schedule Execution

S015-SC-001 [P1] Schedule executes within 60 seconds of due time.
- The daemon evaluates schedules every 60 seconds. A schedule due at T **MUST** begin execution no later than T + 60 seconds.

S015-SC-002 [P1] Missed runs detected and executed on restart.
- After daemon restart, all intervals that passed while the daemon was stopped **MUST** be detected and executed before the next scheduled time.

S015-SC-003 [P1] No duplicate executions for a schedule.
- With `concurrency: prevent`, at most one execution **MUST** be active for a given schedule at any time.

## Notifications

S015-SC-004 [P1] Notifications delivered within 30 seconds of completion.
- On workflow success or failure, the notification pipeline **MUST** be initiated within 30 seconds.

## History

S015-SC-005 [P2] History query returns within 100ms.
- The `sir schedules:history` command **MUST** load and display history for a single schedule in under 100ms for up to 1,000 entries.

## Memory

S015-SC-006 [P2] Daemon memory usage stays below 50MB.
- The `SchedulerDaemon` process **MUST** remain under 50MB resident memory over a 24-hour run.

## Configuration

S015-SC-007 [P1] Schedule configuration validates against schema.
- A `sir schedules:validate` command (or auto-validation on load) **MUST** reject `schedules.yml` files that do not conform to the schema with a descriptive error.

## CLI Output

S015-SC-008 [P1] `sir schedules:list` shows correct next-run time.
- The `Next Run` column **MUST** display a human-readable timestamp derived from the cron expression and current time.

## Persistence

S015-SC-009 [P1] History retained for 30 days.
- Entries in `schedule_history.json` older than 30 days **MUST** be pruned on daemon startup.