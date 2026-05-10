# S014 Interface Requirements

## Remote Target URI Schemes

S014-IF-001 [P1] The CLI **MUST** support the following remote target URI schemes:

| Scheme | Format | Example |
|--------|--------|---------|
| SSH | `ssh://[user@]host[:port]` | `ssh://deploy@server.com:22` |
| Docker | `docker://image[:tag]` | `docker://php:8.2-cli` |
| Kubernetes | `k8s://cluster/namespace` | `k8s://prod-cluster/sir-jobs` |
| AWS Lambda | `aws-lambda://function-name` | `aws-lambda://sir-processor` |

S014-IF-001.a The CLI **MUST** parse the URI scheme to determine the appropriate executor.
S014-IF-001.b Unknown URI schemes **MUST** result in a usage error.

## Remote Configuration File Schema

S014-IF-002 [P1] The remote configuration file **MUST** support the following YAML schema:

```yaml
remotes:
  # SSH remote target
  production:
    type: ssh
    host: server.com
    port: 22
    user: deploy
    key: ~/.ssh/id_rsa
    knownHosts: ~/.ssh/known_hosts

  # Docker remote target
  docker-dev:
    type: docker
    image: php:8.2-cli
    environment:
      NODE_ENV: production

  # Kubernetes remote target
  k8s-prod:
    type: k8s
    cluster: prod-cluster
    namespace: sir-jobs
    context: my-context

  # AWS Lambda remote target
  lambda-processor:
    type: aws-lambda
    functionName: sir-processor
    region: us-east-1
```

S014-IF-002.a The `remotes` key **MUST** be a map of remote names to configurations.
S014-IF-002.b The `type` field **MUST** be one of: `ssh`, `docker`, `k8s`, `aws-lambda`.
S014-IF-002.c SSH remotes **MUST** include `host`, `user`, and `key` fields.
S014-IF-002.d Docker remotes **MUST** include `image` field.
S014-IF-002.e Kubernetes remotes **MUST** include `cluster` and `namespace` fields.
S014-IF-002.f AWS Lambda remotes **MUST** include `functionName` field.

## RemoteExecutor Interface Contract

S014-IF-003 [P1] The `RemoteExecutor` interface **MUST** be defined as follows:

```php
interface RemoteExecutor
{
    /**
     * Execute a workflow on the remote target.
     *
     * @param Workflow $workflow The workflow to execute
     * @param array $inputs Input values for the workflow
     * @return Result The execution result
     */
    public function execute(Workflow $workflow, array $inputs): Result;

    /**
     * Check if this executor is available and configured.
     *
     * @return bool True if available, false otherwise
     */
    public function isAvailable(): bool;

    /**
     * Get the capabilities of this executor.
     *
     * @return array{streaming: bool, artifacts: bool, sandbox: bool, limits: array<string>}
     */
    public function getCapabilities(): array;

    /**
     * Get the type of this executor.
     *
     * @return string One of: ssh, docker, k8s, aws-lambda
     */
    public function getType(): string;
}
```

S014-IF-003.a All executor implementations **MUST** implement `RemoteExecutor`.
S014-IF-003.b The `execute()` method **MUST** throw `RemoteExecutionException` on failure.
S014-IF-003.c The `isAvailable()` method **MUST** return `false` if required dependencies are not available.

## CLI Command Signatures

S014-IF-004 [P1] The CLI **MUST** support the following command signatures:

### sir run (with remote execution)

```
sir run <workflow> [options]

Options:
  --remote <target>     Remote target URI or configured remote name
  --remote-config <file> Path to remote configuration YAML file
  --follow               Stream output in real-time
  --timeout <seconds>    Maximum execution time
  --artifacts <paths>   Comma-separated artifact paths to transfer
```

### sir remote list

```
sir remote list [options]

Options:
  --config <file> Path to remote configuration YAML file

Output:
  Lists all configured remotes with their type and connection status.
```

### sir remote validate

```
sir remote validate <target>

Validates that a remote target is reachable and properly configured.
```

## Workflow Remote Configuration

S014-IF-005 [P1] Workflow files **MAY** include remote execution configuration:

```yaml
description: "Remote workflow"

remote:
  type: docker
  image: php:8.2-cli
  requires:
    - git
    - composer
  environment:
    NODE_ENV: production
  sandbox: true
  limits:
    cpu: "2"
    memory: "4GB"
    timeout: 3600s
    network: false
  cache:
    paths:
      - vendor/
      - node_modules/
    key: "{{checksum('composer.lock')}}"
```

S014-IF-005.a The `remote` block **MUST** override CLI `--remote` flag when both are specified.
S014-IF-005.b When `sandbox: true`, network access **MUST** be disabled unless explicitly enabled.

## Secrets Reference Syntax

S014-IF-006 [P1] Secrets references in workflow steps **MUST** use the following syntax:

```yaml
steps:
  - task: deploy
    secrets:
      API_KEY: "{{vault:api-key}}"
      DB_PASSWORD: "{{env:DB_PASSWORD}}"
```

S014-IF-006.a `{{vault:key-name}}` references secrets from a configured vault.
S014-IF-006.b `{{env:VAR_NAME}}` references environment variables.

## Artifact Reference Syntax

S014-IF-007 [P2] Artifact references in workflow steps **MUST** use the following syntax:

```yaml
steps:
  - task: build
    saveAs: build

  - task: deploy
    remote: production
    artifacts:
      - "{{build.path}}/dist"
      - "{{build.path}}/assets"
```

S014-IF-007.a The `saveAs` field names the output artifact.
S014-IF-007.b The `artifacts` field lists paths to transfer from previous step artifacts.