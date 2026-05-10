# S013 Success Criteria

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S013-SC-001 | Lock file has `commit` and `updated_at` after update | Given a pack `example-pack` installed pre-S013 (no `commit` field), after `sir recipes:update example-pack` the lock file entry contains both `commit` (full SHA) and `updated_at` (ISO 8601). |
| S013-SC-002 | Rollback restores exact pre-update state | Given a pack `example-pack` at commit `abc123`, when the update validation fails, then the pack directory is restored byte-for-byte to its pre-update state (verified by `git diff` returning empty). The lock file still shows `abc123`. |
| S013-SC-003 | `--dry-run` never modifies files or lock file | Given a pack `example-pack` with updates available, after running `sir recipes:update example-pack --dry-run`, the pack directory is unchanged, no backup exists, and `packs.lock.json` is byte-identical to before the command. |
| S013-SC-004 | `--all --yes` proceeds without any prompts | Given 3 packs installed with updates available, running `sir recipes:update --all --yes` produces no interactive prompts (tested via scripted execution with PTY suppression). All available updates are applied. |
| S013-SC-005 | Backup is created at `.sir/packs/.backup/` | After running `sir recipes:update example-pack`, a directory `.sir/packs/.backup/example-pack-<timestamp>/` exists and is a recursive copy of the pack directory before update. |
| S013-SC-006 | Pack with no remote changes is skipped | Given a pack `example-pack` whose remote HEAD matches local `commit`, running `sir recipes:update example-pack` reports "already up to date" and makes no backup or lock file changes. |
| S013-SC-007 | `--project` only updates project-level packs | Given a global pack `global-pack` and a project pack `project-pack` installed, running `sir recipes:update --project` updates only `project-pack`. The global pack is untouched (verified by commit hash unchanged). |
| S013-SC-008 | Local changes prompt appears | Given a pack `example-pack` with uncommitted changes, running `sir recipes:update example-pack` (without `--all --yes`) shows a conflict resolution prompt. The pack is not updated until a choice is made. |
| S013-SC-009 | `--ref` pins pack to specified ref | Given a pack `example-pack` tracking `main`, running `sir recipes:update example-pack --ref=v1.0.0` updates the lock file `ref` field to `v1.0.0` and `commit` to the SHA of that tag. |
| S013-SC-010 | Update summary shows counts | After `sir recipes:update --all`, a summary line shows: "Updated N pack(s), M already up to date, K failed." |
