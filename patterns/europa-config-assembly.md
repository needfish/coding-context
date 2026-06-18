# europa: Configuration Assembly

## Problem

Configuration must be assembled from multiple sources — CLI arguments, environment variables, config files, platform defaults — with precedence rules, validation, and sensible fallbacks.

## Context

Arises in virtually every non-trivial tool. Configuration sources are ordered by priority (CLI overrides env var, env var overrides config file, config file overrides default). The assembly step merges them into a single validated configuration object that the rest of the system consumes.

## Domains Involved

- **CLI arguments** (lexical): command-line flags and positional arguments, parsed into structured options.
- **Environment variables** (lexical): shell environment variables that influence behaviour.
- **Config files** (lexical): files in standard locations (project, user, system) containing persistent settings.
- **Platform defaults** (lexical): built-in defaults for every configurable parameter.
- **Merged config** (lexical): the final assembled and validated configuration.
- **Validator** (causal): checks the merged config for consistency and correctness.

## Shared Phenomena

- CLI → Assembler: parsed options (flag values, positional args)
- Env vars → Assembler: variable names and values
- Config files → Assembler: parsed file contents (JSON, TOML, YAML)
- Defaults → Assembler: platform-specific baseline values
- Assembler → Validator: candidate merged config
- Validator → Assembler: validation errors or warnings
- Assembler → Consumer: validated merged config

## Invariants

- Higher-priority sources must override lower-priority sources on a per-field basis, not wholesale.
- Every configuration parameter must have a defined default — no parameter may be undefined after assembly.
- Unknown or malformed keys from config files must be reported (warnings or errors), not silently ignored.
- Validation must catch contradictory or out-of-range values before the config is used.
- The assembly must be deterministic — the same sources in the same order must produce the same config.

## Known Failure Modes

- **Silent override**: An env var or config file silently overrides a CLI argument the user explicitly set, causing unexpected behaviour.
- **Partial override**: A config file sets some fields but the defaults for unset fields come from a different source, producing an inconsistent mix.
- **Undetected contradiction**: Two sources set the same field to different values, and the merge does not detect the conflict (last-writer-wins may not be what the user expects).
- **Config file not found**: A required config file is missing, and the tool fails or uses incomplete defaults without adequate diagnostics.
- **Platform assumption**: A default that works on one platform (e.g., `/tmp` for temp files) is invalid on another (Windows), causing runtime errors.
- **Validation delay**: Validation happens too late, after the system has already started using partial config.

## Forces

- **Explicitness vs. conciseness**: Listing every parameter in every source is clear but verbose; cascading defaults are concise but make the final configuration hard to predict.
- **Early vs. deferred validation**: Validating during assembly gives immediate feedback but may require knowledge of runtime state not yet available; deferred validation has full context but delays error reporting.
- **Strict vs. tolerant merging**: Strict merging fails on any conflict (safe but frustrating); tolerant merging uses precedence rules (flexible but may mask mistakes).
- **File-based vs. env-var-based config**: Config files support structured data (nested, typed) but require file I/O; env vars are always available but flat and string-typed.

## Related Subproblems

- **jupiter/validate**: Configuration must be validated after assembly.
- **neptune/transform**: Raw config values may need type coercion or transformation.
- **saturn/route**: Config may affect routing decisions.
- **mercury/display**: Config affects display formatting.

## Frame Concern

**Merge ordering transparency.** The user sees the final behaviour but not which source contributed which value. When behaviour is unexpected, the user must trace through precedence rules mentally. The concern is how to make the merge result introspectable — what value was used and where it came from — without requiring the user to read source code.
