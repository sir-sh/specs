# S006 Interface Requirements

## Method JSON Schema

### S006-IF-001 [P1] Method Definition Schema

A method definition **MUST** be a JSON object conforming to the following schema:

```json
{
  "method": "string",
  "description": "string",
  "args": {
    "argName": {
      "type": "string",
      "required": "boolean",
      "description": "string",
      "default": "any"
    }
  },
  "effects": {
    "network": "boolean",
    "writes": ["string"],
    "interactive": "boolean"
  },
  "impl": {
    "type": "string",
    "command": "string | array"
  }
}
```

#### Field Specifications

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `method` | `string` | No | Human-readable method name. If absent, falls back to the filename without `.json`. |
| `description` | `string` | No | Description of what the method does. |
| `args` | `object` | No | Map of argument definitions. Keys are argument names. |
| `args[*].type` | `string` | No | One of: `string`, `int`, `bool`, `path`, `enum`. Defaults to `string`. |
| `args[*].required` | `boolean` | No | If `true`, caller must provide this argument unless a `default` is set. Defaults to `false`. |
| `args[*].description` | `string` | No | Human-readable description of the argument. |
| `args[*].default` | `any` | No | Default value applied when argument is not provided. |
| `effects` | `object` | No | Side-effect metadata for AI inspection. |
| `effects.network` | `boolean` | No | Indicates whether method makes network calls. |
| `effects.writes` | `array` of `string` | No | Paths the method may write to (may contain template expressions). |
| `effects.interactive` | `boolean` | No | Indicates whether method requires interactive input. |
| `impl` | `object` | Yes | Implementation specification. |
| `impl.type` | `string` | Yes | Either `"shell"` or `"exec"`. |
| `impl.command` | `string \| array` | Yes | For `shell`: a command string. For `exec`: an array of command segments. |

### S006-IF-002 [P1] sir-pack.json Schema

Pack metadata **MUST** be a JSON object conforming to:

```json
{
  "namespace": "string"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `namespace` | `string` | No | Prefix applied to all method names from this pack (dot notation). If absent, pack directory basename is used. |

### S006-IF-003 [P1] Shell Execution Result Schema

Shell execution **MUST** return an object with:

```json
{
  "exitCode": "integer",
  "output": "string",
  "errorOutput": "string",
  "success": "boolean"
}
```

### S006-IF-004 [P1] Exec Execution Result Schema

Exec execution **MUST** return an object with the parsed JSON stdout **merged** with:

```json
{
  "exitCode": "integer",
  "errorOutput": "string",
  "success": "boolean",
  "raw_output": "string (only if stdout was not valid JSON)"
}
```

When stdout is valid JSON, the parsed JSON object is returned with `exitCode`, `errorOutput`, and `success` keys merged in. When stdout is not valid JSON, `raw_output` is set to the raw stdout string.

## MethodRegistry Public API

### S006-IF-005 [P1] MethodRegistry Interface

```php
class MethodRegistry
{
    public function __construct(LayerResolver $layerResolver);

    // Discover methods from all layers and packs
    public function discoverMethods(): void;

    // Register a single method (overwrites if exists)
    public function register(string $name, array $definition): void;

    // Get a method definition by name, returns null if not found
    public function get(string $name): ?array;

    // Check if a method exists
    public function has(string $name): bool;

    // Get all registered method definitions keyed by name
    public function all(): array;
}
```

## MethodExecutor Public API

### S006-IF-006 [P1] MethodExecutor Interface

```php
class MethodExecutor
{
    // Execute a method with given arguments
    // Throws InvalidArgumentException on validation failure or unknown impl type
    public function execute(
        array $definition,
        array $args,
        Context $context,
        bool $stream = true
    ): array;
}
```

## Shell vs Exec Behavior Differences

| Aspect | `impl.type: "shell"` | `impl.type: "exec"` |
|--------|----------------------|---------------------|
| Command format | Single shell command string (passed to `Process::fromShellCommandline()`) | Array of command segments (passed to `Process` constructor) |
| Template resolution | Resolved via `$context->resolveTemplate()` before execution | Not resolved (external program receives raw JSON args) |
| Input to process | Command string evaluated by shell | JSON-encoded args via `setInput()` |
| Output parsing | Raw string output returned as `output` | stdout parsed as JSON; if parse fails, `raw_output` is raw string |
| Use case | Shell builtins, pipes, redirections | External programs (Python, Node, etc.) with structured I/O |
