# S002 Interface Requirements

## Config File Schema

Each `.sir/` layer may contain zero, one, or both of the following config files.

### `config.yml`

Top-level structure: plain YAML map (associative array when parsed).

```yaml
# Example: project-level config
database:
  host: localhost
  port: 3306
  username: root
  password: secret

log:
  level: debug
  file: app.log

timeout: 30

plugins:
  - name: yaml-plugin
    enabled: true
  - name: json-plugin
    enabled: false
```

### `config.json`

Top-level structure: JSON object (associative array when decoded).

```json
{
  "database": {
    "host": "localhost",
    "port": 3306,
    "username": "root",
    "password": "secret"
  },
  "log": {
    "level": "debug",
    "file": "app.log"
  },
  "timeout": 30,
  "plugins": [
    { "name": "yaml-plugin", "enabled": true },
    { "name": "json-plugin", "enabled": false }
  ]
}
```

## ConfigLoader API

### `ConfigLoader::loadConfig(?string $startPath = null): array`

Loads and merges configuration from all resolved `.sir/` layers.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `$startPath` | `?string` | No | Starting directory for layer resolution. Defaults to `getcwd()`. |

**Returns:** `array` -- merged configuration array, or empty array if no config files exist.

**Throws:** Nothing. All errors are logged and skipped.

### `ConfigLoader::get(array $config, string $key, mixed $default = null): mixed`

Accesses a nested configuration value using dot notation.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `$config` | `array` | Yes | Configuration array from `loadConfig()`. |
| `$key` | `string` | Yes | Dot-notation key (e.g., `database.host`). |
| `$default` | `mixed` | No | Value returned when key does not exist. Defaults to `null`. |

**Returns:** The resolved value or `$default`.

## Layer Precedence

When multiple layers provide the same top-level keys:

```
Layer order (processed):  [global] -> [mid] -> [project]
Merge result:             global values overridden by mid, then by project
```

Within a single layer, if both `config.yml` and `config.json` exist:

```
Processing order within layer: config.yml first, config.json second
Result: JSON values override YAML values for same keys within that layer
```

## Merged Configuration Example

Given:

**~/.sir/config.yml** (global):
```yaml
database:
  host: localhost
  port: 3306
log:
  level: debug
```

**~/.sir/config.json** (global):
```json
{
  "timeout": 30
}
```

**./.sir/config.yml** (project):
```yaml
database:
  password: secret
log:
  file: app.log
```

**Merged result** (processed global first, project second):
```php
[
  'database' => [
    'host'     => 'localhost',   // from global YAML
    'port'     => 3306,           // from global YAML
    'password' => 'secret',       // from project YAML
  ],
  'log' => [
    'level' => 'debug',           // from global YAML
    'file'  => 'app.log',         // from project YAML
  ],
  'timeout' => 30,                // from global JSON
]
```

## Supported Data Types

| Type | YAML Example | JSON Example | Merged As |
|------|-------------|--------------|-----------|
| String | `name: hello` | `"name": "hello"` | Scalar |
| Integer | `port: 3306` | `"port": 3306` | Scalar |
| Float | `rate: 1.5` | `"rate": 1.5` | Scalar |
| Boolean | `enabled: true` | `"enabled": true` | Scalar |
| Null | `value: ~` | `"value": null` | Scalar |
| Array (list) | `plugins: [- a, b]` | `"plugins": ["a", "b"]` | Concatenated |
| Object (map) | `db: {host: localhost}` | `"db": {"host": "localhost"}` | Recursively merged |