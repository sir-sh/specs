# S007 Interface Requirements

## TaskInterface Contract

### S007-IF-001 [P1] TaskInterface Method Signatures

```php
namespace App\Services\Tasks;

use App\Services\Context;

interface TaskInterface
{
    public function execute(array $args, Context $context): mixed;
    public function getMetadata(): array;
}
```

S007-IF-001.a The `execute` method **MUST** accept `$args` as an associative array of resolved arguments and `$context` as the current workflow execution context.

S007-IF-001.b The `execute` method **MUST** return `mixed`, but all built-in implementations return `array`.

S007-IF-001.c The `getMetadata` method **MUST** accept no arguments and return an associative array.

## TaskRegistry API

### S007-IF-002 [P1] TaskRegistry Method Signatures

```php
namespace App\Services\Tasks;

class TaskRegistry
{
    public function register(string $name, TaskInterface $task): void;
    public function get(string $name): ?TaskInterface;
    public function all(): array;
    public function has(string $name): bool;
}
```

S007-IF-002.a `register()` **MUST** store the task instance keyed by `$name`. If a task with the same name already exists, it **MUST** be silently overwritten.

S007-IF-002.b `get()` **MUST** return the `TaskInterface` instance for the given name, or `null` if not registered.

S007-IF-002.c `all()` **MUST** return the full `array<string, TaskInterface>` map of registered tasks.

S007-IF-002.d `has()` **MUST** return `true` if a task with the given name is registered, `false` otherwise.

## workspace.temp Input/Output Schema

### S007-IF-003 [P1] workspace.temp Schema

**Input args:**

| Arg | Type | Required | Default | Description |
|-----|------|----------|---------|-------------|
| `prefix` | `string` | No | `sir-workspace` | Prefix for the temp directory name |
| `repo` | `string` | No | _(none)_ | Repository name; triggers creation of a repo subdirectory |

**Output (without `repo`):**

```php
[
    'workspace' => string,  // Absolute path to created temp directory
]
```

**Output (with `repo`):**

```php
[
    'workspace' => string,  // Absolute path to created temp directory
    'repoPath'  => string,  // Absolute path to repo subdirectory within workspace
]
```

S007-IF-003.a The `workspace` value **MUST** match the pattern `{sys_get_temp_dir()}/{prefix}-{uniqid}`.

S007-IF-003.b The `repoPath` value **MUST** equal `{workspace}/{basename(repo, '.git')}`.

## files.write Input/Output Schema

### S007-IF-004 [P1] files.write Schema

**Input args:**

| Arg | Type | Required | Default | Description |
|-----|------|----------|---------|-------------|
| `path` | `path` | Yes | _(none)_ | Absolute or relative file path to write to |
| `content` | `string` | Yes | `''` | Content to write to the file |

**Output:**

```php
[
    'path'  => string,  // The path that was written to (same as input)
    'bytes' => int,     // Number of bytes written (strlen of content)
]
```

S007-IF-004.a The `bytes` value **MUST** equal `strlen($content)`, representing the raw byte count of the written content.

## files.writeMd Input/Output Schema

### S007-IF-005 [P1] files.writeMd Schema

**Input args:**

| Arg | Type | Required | Default | Description |
|-----|------|----------|---------|-------------|
| `path` | `path` | Yes | _(none)_ | Absolute or relative file path to write to |
| `content` | `string` | Yes | `''` | Markdown body content |
| `title` | `string` | No | `null` | Optional H1 title to prepend |

**Output:**

```php
[
    'path'  => string,  // The path that was written to
    'bytes' => int,     // Number of bytes written (strlen of final markdown)
]
```

S007-IF-005.a When `title` is provided, the written content **MUST** be `"# {title}\n\n{content}"` and `bytes` **MUST** reflect the length of this combined string.

S007-IF-005.b When `title` is `null`, the written content **MUST** be `content` verbatim and `bytes` **MUST** equal `strlen($content)`.

## git.clone Input/Output Schema

### S007-IF-006 [P1] git.clone Schema

**Input args:**

| Arg | Type | Required | Default | Description |
|-----|------|----------|---------|-------------|
| `repo` | `string` | Yes | _(none)_ | Git repository URL or path |
| `folder` | `path` | No | `getcwd() . '/' . basename(repo, '.git')` | Target directory for the clone |
| `branch` | `string` | No | _(none)_ | Branch to clone; adds `--branch` flag |

**Output:**

```php
[
    'path'   => string,  // Absolute path to cloned directory
    'repo'   => string,  // The repo URL/path that was cloned
    'output' => string,  // stdout from git clone
]
```

S007-IF-006.a The `path` value **MUST** be the resolved target folder (either the explicit `folder` arg or the derived default).

## tests.auto Input/Output Schema

### S007-IF-007 [P1] tests.auto Schema

**Input args:**

| Arg | Type | Required | Default | Description |
|-----|------|----------|---------|-------------|
| `cwd` | `path` | No | `getcwd()` | Working directory to detect and run tests in |

**Output:**

```php
[
    'command'     => string,  // The detected test command ('npm test' or 'composer run-script test')
    'exitCode'    => int,     // Process exit code (0 = success)
    'output'      => string,  // stdout from test run
    'errorOutput' => string,  // stderr from test run
    'markdown'    => string,  // Formatted markdown summary of results
    'success'     => bool,    // true if exitCode === 0
]
```

S007-IF-007.a The `command` value **MUST** be one of `'npm test'` or `'composer run-script test'`.

S007-IF-007.b The `markdown` value **MUST** include the test type, command, exit code, and output formatted with markdown headers and code blocks.

S007-IF-007.c The `success` value **MUST** be `true` when `exitCode === 0` and `false` otherwise.

## coderabbit.review Input/Output Schema

### S007-IF-008 [P2] coderabbit.review Schema

**Input args:**

| Arg | Type | Required | Default | Description |
|-----|------|----------|---------|-------------|
| `target` | `string` | Yes | _(none)_ | Target branch for review |
| `base` | `string` | No | `main` | Base branch to compare against |
| `cwd` | `path` | No | `getcwd()` | Working directory for the review |

**Output (coderabbit CLI method):**

```php
[
    'method'   => 'coderabbit-cli',
    'output'   => string,   // stdout from coderabbit review
    'markdown' => string,   // Formatted review output
]
```

**Output (gh CLI fallback method):**

```php
[
    'method'   => 'gh-cli',
    'message'  => 'Manual PR creation needed',
    'markdown' => string,   // Instructions for manual PR creation
]
```

S007-IF-008.a The `method` key **MUST** indicate which CLI was used: `'coderabbit-cli'` or `'gh-cli'`.

## Metadata Structure Schema

### S007-IF-009 [P1] getMetadata() Return Schema

Every task's `getMetadata()` return **MUST** conform to this structure:

```php
[
    'name'        => string,  // Dot-notation task identifier (e.g., 'workspace.temp')
    'description' => string,  // Human-readable description
    'args'        => [
        'argName' => [
            'type'     => string,        // 'string' or 'path'
            'required' => bool,          // Present when true
            'default'  => mixed,         // Present when there is a default
        ],
        // ... additional args
    ],
    'effects'     => [
        'writes'  => array|null,   // Array of path patterns affected, or absent
        'network' => bool|null,    // Whether the task requires network, or absent
    ],
]
```

S007-IF-009.a The `name` key **MUST** match the string used in `TaskRegistry::register()`.

S007-IF-009.b The `effects` array **MUST** declare `network: true` for tasks that make network calls (`git.clone`, `coderabbit.review`).

S007-IF-009.c The `effects` array **SHOULD** declare `writes` with path patterns for tasks that create files or directories (`workspace.temp`, `files.write`, `files.writeMd`, `git.clone`).
