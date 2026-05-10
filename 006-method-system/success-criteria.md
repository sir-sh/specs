# S006 Success Criteria

| ID | Criterion |
|----|-----------|
| S006-SC-001 [P1] | Methods discoverable from all loaded packs across all resolved layers. Given 3 packs with methods in `layers/1/packs/`, `layers/2/packs/`, and `layers/3/packs/`, calling `discoverMethods()` and then `all()` returns all methods from all three packs. |
| S006-SC-002 [P1] | Argument validation catches all required violations. Calling a method with any required argument missing throws `InvalidArgumentException` before any process is spawned. |
| S006-SC-003 [P1] | Type casting covers all declared types. Given args with types `int`, `bool`, `string`, `path`, `enum`, each is correctly cast to its target type and the result is not a string when `int` or `bool` was declared. |
| S006-SC-004 [P1] | Shell and exec types both produce structured results. Shell execution returns `exitCode`, `output`, `errorOutput`, `success`. Exec execution returns those same keys plus any keys from the parsed JSON stdout. |
| S006-SC-005 [P2] | Streaming and buffered modes both functional. With `stream: true`, output appears in terminal during execution. With `stream: false`, no output appears in terminal and output is in the result array. |
| S006-SC-006 [P1] | Template resolution works in shell commands. Given a command `"git {{action}} {{shellQuote(repo)}}"` and args `{"action": "clone", "repo": "https://github.com/example/repo"}`, the resolved command is `git clone 'https://github.com/example/repo'`. |
| S006-SC-007 [P1] | Default arguments are applied without caller needing to pass them. Calling a method with optional args omitted produces the same behavior as calling with those default values explicitly provided. |
| S006-SC-008 [P2] | Effects metadata is preserved intact in registered definitions. After `discoverMethods()` and `get("git.clone")`, the returned definition includes the original `effects.network` and `effects.writes` fields if they were defined. |
| S006-SC-009 [P1] | Unknown method lookup returns `null` (not an exception). Calling `get("nonexistent.method")` on a registry with no such method returns `null`. |
| S006-SC-010 [P1] | Method name falls back to filename when JSON `method` field is absent. A file `methods/clone.json` with content `{"impl": {...}}` is registered under `clone` (not `clone.json`). |
| S006-SC-011 [P1] | Namespace prefix applied from `sir-pack.json`. A pack with `{"namespace": "docker"}` prefixes all its methods with `docker.`. |
| S006-SC-012 [P2] | exec type stdin receives valid JSON. A Python/Node script reading from stdin receives well-formed JSON matching the validated args. |
| S006-SC-013 [P1] | Non-JSON exec stdout does not crash the executor. When the external program outputs plain text, the result contains `raw_output` and execution completes without exception. |
| S006-SC-014 [P1] | Context is populated with validated args before execution. When `$context->set()` is called for each arg, subsequent template resolution in shell commands can reference those args via `{{argName}}`. |
