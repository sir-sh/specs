# S005 Success Criteria

| ID | Criterion | Measurable Outcome |
|----|-----------|-------------------|
| S005-SC-001 | Pack cloned to correct path | Given `sir recipes:install github:sir-sh/example-pack@main`, the repository is cloned to `~/.sir/packs/example-pack` (global) or `{cwd}/.sir/packs/example-pack` (project). The directory contains the cloned files. |
| S005-SC-002 | Lock file accurately reflects state | After install, `~/.sir/packs.lock.json` contains a `packs.example-pack` entry with `url`, `ref`, `type`, `metadata`, and `installed_at` fields populated correctly. |
| S005-SC-003 | Removal is clean | After `sir recipes:remove example-pack`, the pack directory does not exist and the lock file entry is removed. No residual files remain in the pack directory. |
| S005-SC-004 | Methods discoverable post-install | After installing a pack with methods defined in `methods/*.json`, those methods appear in the MethodRegistry discovery results (verified via `sir agent tools` or equivalent S006 verification). |
| S005-SC-005 | Clone timeout enforced | A clone operation that exceeds 300 seconds **MUST** be terminated and raise an error. Measured via manual timeout test or mock slow Git server. |
| S005-SC-006 | All three source formats parse correctly | `github:owner/repo@ref`, `git:url#ref`, and `owner/repo@ref` all parse to the correct `url` and `ref` values without throwing. Each format is unit-tested individually. |
| S005-SC-007 | Project vs global scope isolation | Installing the same pack ID once globally and once with `--project` results in two separate directories and two separate lock files, with no interference. |
| S005-SC-008 | Stub commands produce expected output | `sir recipes:update` outputs a message containing "remove and reinstall". `sir recipes:apply` outputs a no-op informational message. Both exit with code 0. |
| S005-SC-009 | sir-pack.json validation | Installing a pack without `sir-pack.json` throws `RuntimeException`. Installing a pack with invalid JSON throws `RuntimeException`. Both prevent lock file updates. |
| S005-SC-010 | Idempotent reinstall | Installing the same pack twice succeeds without error. The second install overwrites the first. The lock file reflects the second install's timestamp. |
