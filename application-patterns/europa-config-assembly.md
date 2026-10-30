# europa -- configuration assembly

## Problem

A program needs settings from multiple sources: command-line arguments, environment variables, config files, and defaults. These sources must be combined into one configuration, with higher-priority values overriding lower-priority ones on a per-field basis.

## Input

- Default values (lowest priority)
- Config file values (from TOML, YAML, JSON, or INI file)
- Environment variable values
- Command-line argument values (highest priority)

## Output

- A single configuration object with all fields resolved

## Observable Behavior

- For each config field, the value from the highest-priority source that provides it is used
- If no source provides a value, a default is used (or an error is returned if the field is required)
- Source priorities are: command-line > environment variable > config file > defaults
- Missing optional fields do not cause errors
- Unknown fields in config files or command-line args may cause errors or be silently ignored

## Minimal Code

```python
import os, json

def assemble_config(cli_args: dict, env_prefix: str = "APP_") -> dict:
    config = {
        "host": "localhost",
        "port": 8080,
        "debug": False,
    }
    # Config file
    if os.path.exists("config.json"):
        with open("config.json") as f:
            file_config = json.load(f)
            config.update(file_config)
    # Environment variables
    for key in config:
        env_key = env_prefix + key.upper()
        if env_key in os.environ:
            config[key] = os.environ[env_key]
    # CLI args override all
    for key, value in cli_args.items():
        if value is not None:
            config[key] = value
    return config
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Partial override not per-field | A config file that sets "host" and "port" overwrites "debug" even though the file does not specify it | Merge per-field, not by replacing the whole config object |
| Environment variable type confusion | PORT=abc (string) assigned to port (int field) raises type error at use time | Validate types at config assembly time, not at use time |
| Undefined keys | A typo in config file (porrt) is silently ignored | Warn or error on unknown keys |
| Conflicting sources | Users cannot tell which source provided a value | Log the source of each resolved value |
| Required field missing | The program starts but crashes later because a required config field is missing | Validate required fields during assembly and fail early |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Explicit priority vs. first-wins | Explicit priority (CLI > env > file > defaults) is predictable per-field. First-wins (load sources in order) is simpler but may hide higher-priority sources. |
| Validation at assembly vs. at use | Validate at assembly: fail fast with clear message. Validate at use: may cause confusing runtime errors. |
| Flat vs. nested config | Flat (all keys at top level) is simple. Nested (sections, groups) is organised but more complex to merge. |

## Start With

Use a config library (crate config in Rust, python-decouple in Python, envconfig in Go). Start with one config file and environment variables. Add command-line parsing when needed.

## Split Into Its Own Module When

- More than two config sources are used.
- Config validation (types, required fields) is needed.
- Config values are shared across multiple modules.
- Config file format (TOML, YAML, JSON) needs to be configurable.
- Config reloading (watch file for changes) is needed.

## Signs This Is Present

- os.environ, sys.argv, or argparse is read at startup.
- A config file is parsed (JSON, YAML, TOML, INI).
- Default values are defined and overridden by environment or CLI.
- A config object is passed through the program.

## Common Mistakes

- Replacing the entire config object instead of merging per-field.
- Not validating types at assembly time (a string port crashes later).
- Silently ignoring unknown config keys (typos go unnoticed).
- Loading secrets into config that are later exposed in logs or errors.
- Not documenting which config sources are available and their priority.
- Hard-coding config file paths that are not portable.
