# S014 Success Criteria

## Execution Success

S014-SC-001 [P1] A remote execution **MUST** be considered successful when:
- The workflow completes on the remote target without errors
- All steps execute in the correct order
- The final result is returned to the local machine
- Exit code reflects the workflow outcome

S014-SC-001.a A remote execution **MUST** be considered failed when:
- Connection to the remote target cannot be established
- Authentication fails
- The workflow fails on the remote target
- A timeout or resource limit is exceeded

## Security Success

S014-SC-002 [P1] Secrets **MUST NOT** appear in plaintext in:
- Log files
- Configuration files on disk
- Network traffic (except over encrypted channels)
- Error messages displayed to users

S014-SC-002.a SSH private keys **MUST** never be transmitted over the network in plaintext.

S014-SC-002.b All remote connections **MUST** use encrypted channels (SSH, TLS).

## Performance Success

S014-SC-003 [P1] Streaming output latency **MUST** be under 500ms for remote execution output.

S014-SC-003.a Package transfer time **SHOULD** be optimized by compression. A 10MB workflow package **SHOULD** transfer in under 5 seconds on a typical network connection.

S014-SC-004 [P2] Cached dependencies **SHOULD** reduce subsequent execution transfer time by at least 50% when cache is valid.

## Isolation Success

S014-SC-005 [P1] When sandbox mode is enabled, the remote execution **MUST** be isolated from the host system:
- No filesystem access outside designated paths
- No network access when `network: false`
- Resource limits enforced

## Reliability Success

S014-SC-006 [P1] The system **MUST** provide clear error messages for all failure scenarios.

S014-SC-006.a Error messages **MUST** include actionable guidance for resolving the issue.

S014-SC-007 [P2] The system **SHOULD** support automatic retry for transient failures (network interruption, temporary unavailability).

## Compatibility Success

S014-SC-008 [P1] The `--remote` flag **MUST** work with all supported URI schemes (ssh, docker, k8s, aws-lambda).

S014-SC-008.a The `--remote-config` flag **MUST** correctly load and validate remote configurations from YAML files.

S014-SC-009 [P2] Remote execution **SHOULD** work identically across all supported executor types (same workflow, different remote targets produce equivalent results).