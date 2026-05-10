# S005 Non-Functional Requirements

## Performance

S005-NF-001 [P2] The system **MUST** complete a pack installation (clone + lock file write) in under 60 seconds on a stable 10 Mbps network connection for repositories with fewer than 100 commits.

S005-NF-002 [P2] The system **SHOULD** use shallow clones (`--depth 1`) to minimize network transfer and disk usage.

S005-NF-003 [P3] The `sir recipes:list` command **MUST** respond in under 500ms regardless of the number of installed packs.

## Disk Usage

S005-NF-004 [P2] The system **MUST** store only one copy of each pack's files on disk (per scope), even if the pack is reinstalled multiple times.

S005-NF-005 [P2] Shallow clones **MUST** be used to minimize disk usage. A full clone **MUST NOT** be performed during pack installation.

S005-NF-006 [P3] Lock files **SHOULD** be kept under 1 MB in size even with 100+ installed packs.

## Network Reliability

S005-NF-007 [P2] The system **MUST** enforce a 300-second timeout on all Git clone operations to prevent indefinite hangs.

S005-NF-008 [P1] The system **MUST** provide a meaningful error message when network failure occurs during clone, including the underlying Git error output.

S005-NF-009 [P2] The system **SHOULD** gracefully handle transient network errors by providing actionable guidance in error messages.

## Security

S005-NF-010 [P2] The system **MUST** shell-quote all template variables in `impl.command` to prevent command injection.

S005-NF-011 [P2] The system **MUST NOT** execute arbitrary code during pack installation beyond the defined Git clone operation.

S005-NF-012 [P3] Pack method execution **SHOULD** respect the `effects.writes` declaration by validating that methods only write to declared paths.

## Scalability

S005-NF-013 [P3] The system **SHOULD** support at least 50 installed packs per scope without performance degradation.

S005-NF-014 [P3] Method discovery **SHOULD** complete in under 2 seconds with 50 packs installed, each with up to 20 methods.

## Compatibility

S005-NF-015 [P1] The system **MUST** work with any Git hosting platform that supports standard Git clone URLs (GitHub, GitLab, Bitbucket, self-hosted).

S005-NF-016 [P2] The system **MUST** correctly parse Git URLs with or without the `.git` suffix.
