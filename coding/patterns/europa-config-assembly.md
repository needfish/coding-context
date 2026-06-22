# ------------------------------------------------------------------------------
#  europa-config-assembly.md
#  Data variation -- same merge algorithm, different values per source.
# ------------------------------------------------------------------------------

# - Problem -

Your program needs configuration from multiple places -- CLI flags, environment variables, config files, and sensible defaults. You need to combine them with clear precedence rules.

# - When You'll See This -

Virtually every non-trivial tool. Configuration sources have a priority order (CLI overrides env var, env var overrides config file, config file overrides default). The assembly step merges them into a single validated config object that the rest of the system reads.

# - How It Works -

- **CLI arguments**: command-line flags and positional args, parsed into structured options.
- **Environment variables**: key-value pairs from the shell environment, typically uppercase with underscores.
- **Config files**: YAML, TOML, or JSON files read from well-known paths.
- **Defaults**: hard-coded values that apply when nothing else provides a value.
- **Merged config**: the single validated configuration object after all sources are combined.

# - What Must Always Be True -

- Higher-priority sources override lower-priority sources on a per-field basis.
- A field not set in any source uses its default value.
- Type mismatches across sources (CLI says int, env var says string) are caught and reported.
- Required fields that are missing from all sources produce a clear error at startup.
- The final config is validated as a whole (not per-source).

# - What Goes Wrong -

- **Partial override confusion**: setting one field via CLI but expecting others from the config file -- and the CLI clears the whole config.
- **Silent ignore**: a config file field is misspelled and silently ignored.
- **Type coercion surprises**: `--port=eight` parses as a string instead of failing as an invalid int.
- **Missing required field**: the app starts but crashes later because a required config value wasn't set.
- **Side effects during config loading**: reading a config file triggers network calls or filesystem writes.
- **Too many config sources**: the user can't predict which source wins for a given field.

# - Design Trade-Offs -

- **Flat vs. nested config**: flat keys (like `db.host`) are simple but don't compose; nested structures mirror the domain but are harder to merge.
- **Fail-fast vs. tolerant**: fail-fast rejects invalid config at startup (clear errors, no surprises); tolerant accepts partial config and uses defaults but may hide mistakes.

# - Related Problems -

- **pluto/input-parsing**: config files need parsing before merging.
- **jupiter/validation**: the merged config needs validation.
- **mars/persistence**: config may be cached or reloaded from disk.

# - Getting Started -

A dict of defaults, updated by the config file, updated by env vars, updated by CLI args. Python: `merged = {**defaults, **file_config, **env_overrides, **cli_args}`. No validation library, no config framework.

# - When to Split Into Its Own Module -

- You have more than 10 configuration fields.
- Configuration comes from more than two sources.
- Configuration needs validation rules (ranges, required fields, dependencies between fields).
- You need to reload configuration at runtime without restarting.

# - Signs You're in This Pattern -

- Your program accepts CLI flags, environment variables, and config files.
- You have a block of code that reads and merges configuration at startup.
- Changing a setting means editing multiple places (defaults, env var name, CLI flag).
- Configuration errors only appear at the moment the misconfigured feature is used.

# - Common Mistakes -

- Writing a custom config parser before trying an existing one (argparse, clap, Viper).
- Adding a config file format when environment variables cover the use case.
- Loading config from network sources at startup without a fallback or timeout.

# - The Core Challenge -

Configuration comes from untrusted, untyped sources (strings from env vars, text from files, arbitrary CLI input). The merge process must turn these into a typed, validated, predictable configuration object that the rest of the system can trust.
