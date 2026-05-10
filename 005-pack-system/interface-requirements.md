# S005 Interface Requirements

## sir-pack.json Schema

The `sir-pack.json` file is the pack metadata descriptor, placed at the root of a pack Git repository.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["name", "version", "methods"],
  "properties": {
    "name": {
      "type": "string",
      "description": "Unique pack identifier (kebab-case recommended)"
    },
    "version": {
      "type": "string",
      "description": "Semantic version string (e.g., '1.0.0')"
    },
    "description": {
      "type": "string",
      "description": "Human-readable description of the pack"
    },
    "methods": {
      "type": "array",
      "description": "List of method files relative to the methods/ directory",
      "items": {
        "type": "string"
      },
      "example": ["clone.json", "commit.json"]
    },
    "author": {
      "type": "string",
      "description": "Pack author name or organization"
    },
    "license": {
      "type": "string",
      "description": "SPDX license identifier"
    }
  }
}
```

### Example sir-pack.json

```json
{
  "name": "sir-git",
  "version": "1.0.0",
  "description": "Git operations pack for sir.sh workflows",
  "methods": ["clone.json", "commit.json", "push.json"],
  "author": "sir-sh",
  "license": "MIT"
}
```

## packs.lock.json Schema

The `packs.lock.json` file tracks all installed packs at a given scope (global or project).

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["packs"],
  "properties": {
    "packs": {
      "type": "object",
      "description": "Map of pack ID to pack install record",
      "additionalProperties": {
        "type": "object",
        "required": ["url", "ref", "type", "metadata", "installed_at"],
        "properties": {
          "url": {
            "type": "string",
            "description": "Full Git URL used for cloning"
          },
          "ref": {
            "type": "string",
            "description": "Git branch, tag, or commit SHA cloned"
          },
          "type": {
            "type": "string",
            "enum": ["github", "git"],
            "description": "Source type indicator"
          },
          "metadata": {
            "type": "object",
            "description": "Contents of sir-pack.json at install time"
          },
          "installed_at": {
            "type": "string",
            "format": "date-time",
            "description": "ISO 8601 timestamp of installation"
          }
        }
      }
    }
  }
}
```

### Example packs.lock.json

```json
{
  "packs": {
    "example-pack": {
      "url": "https://github.com/sir-sh/example-pack.git",
      "ref": "main",
      "type": "github",
      "metadata": {
        "name": "example-pack",
        "version": "1.0.0",
        "description": "An example pack",
        "methods": ["clone.json"]
      },
      "installed_at": "2026-04-24T10:30:00+00:00"
    }
  }
}
```

## Method Definition JSON Schema

Method definitions are stored in `methods/*.json` files within a pack.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["method", "description", "impl"],
  "properties": {
    "method": {
      "type": "string",
      "description": "Method name used in workflow YAML (e.g., 'git.clone')"
    },
    "description": {
      "type": "string",
      "description": "Human-readable description of what the method does"
    },
    "args": {
      "type": "object",
      "description": "Input arguments specification",
      "additionalProperties": {
        "type": "object",
        "properties": {
          "type": {
            "type": "string",
            "enum": ["string", "number", "boolean", "path"],
            "description": "Argument type"
          },
          "required": {
            "type": "boolean",
            "description": "Whether the argument must be provided"
          },
          "description": {
            "type": "string",
            "description": "Argument description for help text"
          }
        }
      }
    },
    "effects": {
      "type": "object",
      "properties": {
        "network": {
          "type": "boolean",
          "description": "Whether the method makes network requests"
        },
        "writes": {
          "type": "array",
          "items": { "type": "string" },
          "description": "Filesystem paths the method writes to (templated)"
        }
      }
    },
    "impl": {
      "type": "object",
      "required": ["type"],
      "properties": {
        "type": {
          "type": "string",
          "enum": ["shell", "exec"],
          "description": "Implementation type"
        },
        "command": {
          "type": "string",
          "description": "Shell command template using {{variableName}} syntax"
        }
      }
    }
  }
}
```

### Example Method Definition

```json
{
  "method": "git.clone",
  "description": "Clone a git repository",
  "args": {
    "repo": {
      "type": "string",
      "required": true,
      "description": "Repository URL to clone"
    },
    "folder": {
      "type": "path",
      "required": false,
      "description": "Destination folder path"
    }
  },
  "effects": {
    "network": true,
    "writes": ["{{folder}}"]
  },
  "impl": {
    "type": "shell",
    "command": "git clone {{shellQuote(repo)}} {{shellQuote(folder)}}"
  }
}
```

## Pack Directory Structure

A valid pack repository **MUST** conform to the following directory layout:

```
pack-name/
├── sir-pack.json          # Required: Pack metadata
├── methods/              # Required: Directory containing method definitions
│   ├── method1.json      # At least one method file
│   └── method2.json
└── scripts/               # Optional: Scripts for exec-type methods
    ├── script.py
    └── script.sh
```

- `sir-pack.json` **MUST** be at the repository root.
- `methods/` directory **MUST** exist and contain at least one `.json` file.
- `scripts/` directory is optional; scripts within it are referenced by `exec`-type method implementations.

## Install URL Formats

### Format 1: github: (explicit GitHub)

```
github:owner/repo@ref
```

| Component | Value | Example |
|-----------|-------|---------|
| Prefix | `github:` | `github:` |
| Owner | GitHub organization or user | `sir-sh` |
| Repo | Repository name | `example-pack` |
| Ref | Branch, tag, or commit (optional, defaults to `main`) | `@develop` |

Constructed URL: `https://github.com/{owner}/{repo}.git`

### Format 2: git: (generic Git URL)

```
git:https://github.com/owner/repo.git#ref
git:git@gitlab.com:owner/repo.git#ref
```

| Component | Value | Example |
|-----------|-------|---------|
| Prefix | `git:` | `git:` |
| URL | Full Git clone URL (HTTPS or SSH) | `https://github.com/sir-sh/example-pack.git` |
| Ref | Branch, tag, or commit (optional, defaults to `main`) | `#develop` |

### Format 3: owner/repo (shorthand, GitHub assumed)

```
owner/repo@ref
```

| Component | Value | Example |
|-----------|-------|---------|
| Owner | GitHub organization or user | `sir-sh` |
| Repo | Repository name | `example-pack` |
| Ref | Branch, tag, or commit (optional, defaults to `main`) | `@develop` |

Constructed URL: `https://github.com/{owner}/{repo}.git`

## CLI Command Signatures

### sir recipes:install

```
sir recipes:install <source> [--project]
```

| Argument | Description |
|----------|-------------|
| `source` | Pack source string in one of the three formats |

| Option | Description |
|--------|-------------|
| `--project` | Install to project scope (`.sir/packs/`) instead of global (`~/.sir/packs/`) |

**Exit codes:** 0 on success, 1 on invalid source, 1 on clone failure, 1 on missing `sir-pack.json`.

### sir recipes:list

```
sir recipes:list [--project]
```

| Option | Description |
|--------|-------------|
| `--project` | List packs from project scope lock file instead of global |

**Exit codes:** 0 always.

### sir recipes:remove

```
sir recipes:remove <pack-id> [--project]
```

| Argument | Description |
|----------|-------------|
| `pack-id` | The pack identifier (directory name) to remove |

| Option | Description |
|--------|-------------|
| `--project` | Remove from project scope instead of global |

**Exit codes:** 0 always (succeeds even if pack does not exist).

### sir recipes:update

```
sir recipes:update
```

**Stub in v1.** Outputs informational message directing user to remove and reinstall.

**Exit codes:** 0.

### sir recipes:apply

```
sir recipes:apply
```

**Stub in v1.** No-op informational output.

**Exit codes:** 0.
