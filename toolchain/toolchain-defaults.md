# Toolchain Defaults

## Package Managers

| Ecosystem | Package Manager |
|---|---|
| TypeScript / JavaScript | pnpm |
| Python | uv + venv |
| Rust | cargo (built-in) |

## Per-Language Toolchain

All projects enforce on save and in CI.

| Language | Tool | Runs on |
|---|---|---|
| TypeScript | Prettier + tsc --noEmit | Save + pre-commit |
| Python | Ruff format + basedpyright | Save + pre-commit |
| JavaScript | Prettier + ESLint xstrict/goodparts | Save + pre-commit |
| All | EditorConfig | All editors |

### TypeScript

- strict: true in tsconfig. No any, no unchecked index access, full type rigor.
- Branded types, discriminated unions, type-level state machines.
- Most business logic lives in the Python/Rust backend; frontend TS uses shared Zod schemas, typed htmx request helpers, Preact component props.

### Python

- basedpyright strict for all projects.
- Ruff for linting and formatting.

### Rust

- clippy pedantic + restriction for non-trivial backends.

## Pre-Commit Hooks

Use pre-commit hooks (Python-based, covers TS via tsc --noEmit).

## EditorConfig

All projects include .editorconfig at the repository root.
