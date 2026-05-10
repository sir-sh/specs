# S013 Non-Functional Requirements

## Update Speed

S013-NF-001 [P2] An update of a single pack **SHOULD** complete in under 60 seconds under normal network conditions, excluding the time to transfer large pack archives.

S013-NF-001.a [Sub-criterion] The `git fetch` operation **MUST** use a timeout of at least 30 seconds to prevent premature abandonment on slow connections.

S013-NF-002 [P3] When updating all packs (`--all`), individual pack updates **SHOULD** be processed sequentially to avoid overwhelming the network or Git hosting services.

## Network Bandwidth Efficiency

S013-NF-003 [P2] The update command **MUST** use `git fetch` (not `git pull`) to fetch only the missing objects, avoiding full pack downloads when possible.

S013-NF-003.a [Sub-criterion] The fetch **MUST** be limited to the single remote (`origin`) and the specific ref being tracked.

S013-NF-004 [P3] If the local pack is already at the remote HEAD, the system **MUST** skip the fetch entirely and report the pack as up to date without consuming bandwidth.

## Backup Disk Usage

S013-NF-005 [P2] The backup created for a pack **MUST NOT** exceed the size of the original pack directory at the time of backup.

S013-NF-005.a [Sub-criterion] Backup **MUST** be a plain filesystem copy, not a compressed archive, so rollback is immediate.

S013-NF-006 [P3] Old backups **SHOULD NOT** be automatically deleted by the update command; manual cleanup of `.sir/packs/.backup/` **SHOULD** be supported via a separate `sir recipes:cleanup` command.

## Rollback Atomicity Guarantee

S013-NF-007 [P1] If a failure occurs after the backup is created but before the lock file is updated, the rollback **MUST** restore the pack directory to its exact pre-update state.

S013-NF-007.a [Sub-criterion] Rollback **MUST** be all-or-nothing: either the pack is fully restored or an error is reported; partial rollbacks **MUST NOT** occur.

S013-NF-007.b [Sub-criterion] If rollback itself fails (e.g., backup directory is also missing or corrupted), the system **MUST** display a fatal error with the location of the backup and **MUST NOT** proceed silently.

S013-NF-008 [P1] During a batch update (`--all`), a failure on pack N **MUST NOT** prevent the system from attempting updates on pack N+1, N+2, etc.

## Lock File Write Atomicity

S013-NF-009 [P1] Updates to `packs.lock.json` **MUST** be atomic: the file is written to a temporary file first, then renamed over the original.

S013-NF-009.a [Sub-criterion] If the write fails (e.g., disk full), the original lock file **MUST** remain intact and unmodified.

S013-NF-010 [P1] The lock file **MUST** be updated only after the pack directory has been successfully updated and validated, never before.

## Error Messaging

S013-NF-011 [P1] Every error message **MUST** be clear and actionable, identifying:
- What failed (pack ID, operation)
- Why it failed (network, validation, conflict)
- What was done to recover (rollback performed, lock file unchanged)

S013-NF-012 [P2] Error messages **MUST NOT** expose raw Git output to end users unless the output is necessary to diagnose the issue (e.g., rejected push policies are relevant).

## Dry-Run Isolation

S013-NF-013 [P1] The `--dry-run` flag **MUST** guarantee zero side effects: no network state changes, no filesystem modifications, no lock file writes.

S013-NF-013.a [Sub-criterion] The command **MAY** perform read-only Git operations (`git fetch --dry-run` if available, or `git ls-remote`) to determine available commits without modifying local state.
