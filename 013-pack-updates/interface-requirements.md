# S013 Interface Requirements

## Enhanced packs.lock.json Schema

S013-IF-001 [P1] The `packs.lock.json` lock entry schema **MUST** include the following fields per pack:

```json
{
  "id": "string",
  "url": "string",
  "ref": "string",
  "type": "string",
  "metadata": "object",
  "installed_at": "string (ISO 8601)",
  "commit": "string (40-char Git SHA, required after S013 update)",
  "updated_at": "string (ISO 8601 UTC, required after S013 update)"
}
```

S013-IF-001.a [Sub-criterion] The `commit` field **MUST** be present after any `sir recipes:update` invocation.

S013-IF-001.b [Sub-criterion] The `updated_at` field **MUST** be updated to the current UTC time on every successful update.

S013-IF-001.c [Sub-criterion] Existing lock entries without `commit` or `updated_at` **MUST** be treated as valid; these fields are added lazily on first update.

## Command Signatures

S013-IF-002 [P1] The command `sir recipes:update [--all] [--yes] [--project]` **MUST** update all installed packs.

S013-IF-002.a [Sub-criterion] `--all` **MUST** be required to update all packs (prevents accidental mass update).

S013-IF-002.b [Sub-criterion] `--yes` **MUST** suppress all confirmation prompts when combined with `--all`.

S013-IF-002.c [Sub-criterion] `--project` **MUST** limit the update scope to packs installed under the current project's `.sir/packs/` directory.

S013-IF-003 [P1] The command `sir recipes:update <pack-id> [--ref=<ref>] [--project] [--dry-run]` **MUST** update or preview a single named pack.

S013-IF-003.a [Sub-criterion] `<pack-id>` **MUST** be a required positional argument identifying the pack.

S013-IF-003.b [Sub-criterion] `--ref=<ref>` **MUST** accept a branch name, tag name, or full commit SHA to pin the update target.

S013-IF-003.c [Sub-criterion] `--project` **MUST** search for the pack in the project-level `.sir/packs/` directory first, falling back to global `~/.sir/packs/`.

S013-IF-003.d [Sub-criterion] `--dry-run` **MUST** preview changes without applying them and without modifying the lock file.

## Backup Directory Structure

S013-IF-004 [P1] Backups **MUST** be stored at `.sir/packs/.backup/<pack-id>-<timestamp>/`.

S013-IF-004.a [Sub-criterion] The `.backup/` directory **MUST** be created inside the same parent directory as the pack being backed up (project-level or global).

S013-IF-004.b [Sub-criterion] The timestamp **MUST** use the format `YYYY-MM-DDTHH_MM_SS` (ISO 8601 with colons replaced by underscores).

S013-IF-004.c [Sub-criterion] The backup **MUST** contain a recursive copy of the entire pack directory (all files, hidden files, and subdirectories).

S013-IF-005 [P2] If the `.backup/` directory already exists, new backups **MUST** be created as timestamped subdirectories within it; no existing backups **MUST** be overwritten or deleted.

## Update Display Table Format

S013-IF-006 [P1] When updating all packs (`sir recipes:update` or `sir recipes:update --all`), the output **MUST** display a table with the following columns:

| Column | Description |
|--------|-------------|
| `Pack` | Pack ID |
| `Current` | Current commit SHA (short, 7 chars) |
| `Available` | Remote HEAD commit SHA (short, 7 chars) |
| `Status` | One of: `Updated`, `Up to date`, `Skipped`, `Failed`, `Local changes` |

S013-IF-006.a [Sub-criterion] The table **MUST** be printed to stdout in a human-readable format.

S013-IF-006.b [Sub-criterion] Each row **MUST** be printed as the pack is processed (not after all updates complete), so progress is visible.

S013-IF-006.c [Sub-criterion] If `--dry-run` is used, the Status column **MUST** show `Dry-run` and no actual update occurs.

S013-IF-007 [P2] After the table, a summary line **MUST** be printed: "Updated N packs, M already up to date, F failed, S skipped."

## Exit Codes

S013-IF-008 [P1] Exit code `0` **MUST** be returned when all updates succeeded (or all were already up to date).

S013-IF-008.a [Sub-criterion] Exit code `1` **MUST** be returned when one or more packs failed to update.

S013-IF-008.b [Sub-criterion] Exit code `2` **MUST** be returned when the specified pack ID was not found in the lock file.
