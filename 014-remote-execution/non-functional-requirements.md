# S014 Non-Functional Requirements

## SSH Key Security

S014-NF-001 [P1] SSH private keys **MUST** be stored securely and **MUST NOT** be stored in plaintext anywhere in the sir.sh configuration or workflow files.

S014-NF-001.a SSH private keys **MUST** be stored in the filesystem with permissions `0600` (owner read/write only).
S014-NF-001.b SSH private key paths **MAY** be specified using `~` expansion (e.g., `~/.ssh/id_rsa`).
S014-NF-001.c The system **MUST NOT** accept SSH private key contents directly in configuration files.

S014-NF-001.d SSH known hosts verification **MUST** be supported via a `known_hosts` file.
S014-NF-001.e The system **SHOULD** warn when `StrictHostKeyChecking` is disabled.

## Credential Management

S014-NF-002 [P1] Credentials **MUST NOT** be stored in plaintext in any sir.sh configuration file or workflow definition.

S014-NF-002.a API keys, passwords, tokens, and secrets **MUST** be stored in:
- Environment variables (referenced via `{{env:VAR_NAME}}`)
- External secrets vaults (referenced via `{{vault:key-name}}`)
- SSH agent (for SSH key authentication)

S014-NF-002.b The system **MUST NOT** log, print, or expose credential values in output.
S014-NF-002.c The system **MUST NOT** store credentials in workflow package metadata.

S014-NF-002.d Remote configuration files **MUST NOT** contain plaintext passwords. Authentication **MUST** use key-based methods.

## Network Security

S014-NF-003 [P1] All remote connections **MUST** use encrypted channels:
- SSH connections **MUST** use SSH protocol 2 with encryption
- Docker connections **SHOULD** use TLS when connecting to remote Docker hosts
- Kubernetes connections **MUST** use TLS with cluster CA verification

S014-NF-003.a Plaintext connections (without encryption) **MUST** be rejected by default.

S014-NF-003.b Certificate verification **MUST** be enforced for TLS connections where a CA is available.

## Isolation

S014-NF-004 [P1] Remote execution **MUST** support sandbox isolation with the following guarantees:
- Filesystem isolation: execution environment **MUST NOT** have access to paths outside the workflow working directory unless explicitly configured
- Network isolation: when `network: false`, the execution environment **MUST NOT** be able to make network connections
- Process isolation: remote execution **MUST** run in an isolated process/container/pod

S014-NF-004.a Sandboxed executions **MUST** be terminated if they attempt to violate isolation constraints.

## Reliability

S014-NF-005 [P1] The system **MUST** handle remote execution failures gracefully with the following behavior:
- Connection failures **MUST** produce clear error messages with target identification
- Authentication failures **MUST** produce clear error messages without exposing credentials
- Execution failures **MUST** return the remote exit code and any available output

S014-NF-005.a All errors **MUST** be logged with timestamps and correlation IDs for debugging.

S014-NF-005.b Partial results **SHOULD** be retrieved when possible after failures.

S014-NF-006 [P1] Transfer reliability **MUST** be ensured through:
- Automatic retry with exponential backoff for transient network failures (up to 3 retries)
- Checksum validation of transferred packages
- Resumable transfers where the remote executor supports it

## Performance

S014-NF-007 [P2] Package compression **SHOULD** be applied to workflow packages before transfer to reduce bandwidth usage.

S014-NF-007.a A workflow package of 10MB **SHOULD** compress to under 3MB.

S014-NF-008 [P2] Streaming output latency **MUST** be under 500ms from remote generation to local display.

S014-NF-009 [P2] Caching **SHOULD** be implemented to avoid re-transferring unchanged dependencies:
- Cache key **MUST** be derived from dependency file checksums
- Cache validation **MUST** occur before each remote execution
- Cache hit **MUST** reduce transfer time by at least 50%

## Scalability

S014-NF-010 [P2] The system **SHOULD** support concurrent remote executions:
- Maximum concurrent connections **SHOULD** be configurable
- Queued executions **SHOULD** wait for available connections
- Connection pool exhaustion **MUST** be handled gracefully with clear errors

## Compatibility

S014-NF-011 [P1] Remote execution **MUST** be compatible with:
- SSH protocol 2 compliant servers
- Docker API 1.24+ (Docker Engine 1.12+)
- Kubernetes 1.16+ cluster API
- AWS Lambda invocation API

S014-NF-012 [P1] The system **MUST** work across different operating systems (Linux, macOS, Windows with WSL) for SSH and Docker executors.

## Monitoring

S014-NF-013 [P2] Resource monitoring **SHOULD** be available during remote execution:
- CPU usage reporting
- Memory usage reporting
- Network usage reporting (when not disabled)
- Monitoring data **SHOULD** be accessible via CLI flags and retained for post-execution analysis

S014-NF-013.a Monitoring data **MUST NOT** contain sensitive information.