# ------------------------------------------------------------------------------
#  safety-enforcement-tool-references.md
#  Linting, static analysis, and safety enforcement tools across languages.
# ------------------------------------------------------------------------------

# Tool Choice References

---

## JavaScript Linting / Good Parts Enforcement

Reference for tools that enforce a "Good Parts" subset of JavaScript (ban `==`,
`with`, `eval`, implied globals, etc.). The default choice in AGENTS.md is
`goodparts` for JavaScript projects.

| Tool | Install | Enforces | Configurable | Notes |
|------|---------|----------|-------------|-------|
| **goodparts** (dwyl) | `npm install --save-dev goodparts` | Crockford Good Parts + ES6 "Better Parts" | Via `.eslintrc.js` symlink | Recommended default. Bans `==`, `with`, `eval`, `var`, implied globals, bitwise ops. Allows `const`/`let`, arrow functions, destructuring. Runs as `npx goodparts path/` or via CI. Actively maintained. |
| **eslint-config-airbnb** | `npm install --save-dev eslint-config-airbnb` | Airbnb style guide | Via `extends: ['airbnb']` | Most popular JS style guide. Includes React + import rules. Large rule surface. |
| **eslint-config-xtrict** (areknawo) | `npm install --save-dev eslint-config-xtrict` | Extreme strictness, TypeScript-first | Via `extends: ['xtrict']` | TypeScript with full `strictTypeChecked`. Full type-aware linting. |
| **ESLint built-in: recommended** | (included with ESLint) | Core correctness rules only | `extends: ['eslint:recommended']` | Minimal starting point. Catches real bugs but does not enforce style or Good Parts bans. |

### Quick Choose Guide

| Situation | Use |
|-----------|-----|
| Plain JS, want Crockford Good Parts for modern code | `goodparts` |
| TypeScript project, strictest possible | `eslint-config-xtrict` or `typescript-eslint strictTypeChecked` |
| Need React / JSX linting | `eslint-config-airbnb` or add `eslint-plugin-react` to another config |
| Minimal setup, just catch real bugs | `eslint:recommended` |

---

## C/C++ Linting / Static Analysis

## C/C++ Linting / Static Analysis

The standard freely-enforceable C++ "Good Parts" is the **C++ Core Guidelines**,
authored by Stroustrup and Sutter, enforced via clang-tidy. No purchase needed.

| Tool / Standard | What it enforces | How to run | Notes |
|-----------------|-----------------|------------|-------|
| **clang-tidy** | Modular checks: bugprone, performance, portability, readability, modernize, cppcoreguidelines, cert, misc. Thousands of individual checks. | `clang-tidy file.cpp --checks='*'` or via CMake (`-DCMAKE_CXX_CLANG_TIDY=clang-tidy`). Editor integrations via clangd. | Open source, LLVM project. The closest equivalent to ESLint for C/C++. Actively maintained. Highly configurable. |
| **C++ Core Guidelines** (isocpp) | Type safety, bounds safety, lifetime safety, no raw `new`/`delete`, no `reinterpret_cast`, RAII, rules for interfaces, resources, concurrency. | Via clang-tidy checks (`cppcoreguidelines-*`) or Microsoft C++ Core Check. | The standard "Good Parts" for modern C++. All rules published free online at <https://isocpp.github.io/CppCoreGuidelines/>. |
| **Cppcheck** | Undefined behavior, memory leaks, null pointer derefs, STL misuse. | `cppcheck --enable=all file.cpp` | Open source. Path-sensitive: catches bugs clang-tidy misses. |
| **clang static analyzer** | Null derefs, memory leaks, dead stores, API misuse. | `clang --analyze` or via scan-build. | Open source, LLVM project. Complements clang-tidy. |
| **Microsoft C++ Core Check** | C++ Core Guidelines rules, plus bounds, lifetime, concurrency. | Visual Studio: enable via Code Analysis. | Free with Visual Studio Community. Overlaps clang-tidy's cppcoreguidelines. |

### Recommended Stack (Free, Open Source)

```bash
# CMakeLists.txt:
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wpedantic -Werror")
set(CMAKE_CXX_CLANG_TIDY "clang-tidy;-checks=clang-diagnostic-*,clang-analyzer-*,cppcoreguidelines-*,performance-*,portability-*,readability-*")

# Additional:
cppcheck --enable=all --suppress=missingIncludeSystem .
```

### Quick Choose Guide

| Situation | Use |
|-----------|-----|
| Modern C++ (C++11+), want a "Good Parts" subset | **C++ Core Guidelines** via `clang-tidy -checks=cppcoreguidelines-*` |
| General C/C++ linting | **clang-tidy** with modular check groups |
| Path-sensitive bug finding (UB, leaks) | **Cppcheck** or **clang static analyzer** |
| Catch bugs at compile time, zero config | `-Wall -Wextra -Wpedantic -Werror` |

---

## Rust Linting / Restrictions

Rust ships with an official linter (`clippy`) and compiler-level feature gating.
There is no single "Good Parts" book, but subgroups within clippy define increasingly
restrictive subsets.

| Tool / Config | What it enforces | How to enable | Notes |
|---------------|-----------------|---------------|-------|
| **`#![deny(unsafe_code)]`** | Bans all `unsafe` blocks, functions, traits, and methods. | Add `#![deny(unsafe_code)]` at crate root. | Compiler built-in. Zero-dependency. The closest Rust has to a single "ban this feature" toggle. |
| **clippy::pedantic** | 150+ additional lints beyond default: stricter naming, unnecessary casts, enum variants ordering, doc requirements. | `#![warn(clippy::pedantic)]` at crate root, or `clippy.toml` level = "pedantic". | Official. May be noisy. Cherry-pick over enabling wholesale. |
| **clippy::restriction** | Lints that restrict language feature use: ban `unwrap()`, `panic!`, indexing, `println!`, `regex`, etc. Opt-in per lint. | `#![warn(clippy::restriction)]` or per-lint `#![warn(clippy::unwrap_used)]`. | The strictest group. Each lint is opt-in because many are project-specific. Not meant to be enabled all at once. |
| **clippy::nursery** | New lints still under development. May contain false positives. | `#![warn(clippy::nursery)]` | Official. Useful for catching edge cases in new code patterns. |
| **Safety-Critical Rust Consortium guidelines** | Coding guidelines for safety-critical Rust. Under active development by the Rust Foundation. | Manual review + custom clippy lints (being developed). | Free, published online at <https://github.com/rustfoundation/safety-critical-rust-coding-guidelines>. Not yet enforceable via tooling. |

### Recommended Strict Config

```rust
// src/lib.rs or src/main.rs
#![deny(unsafe_code)]
#![deny(clippy::all)]
#![deny(clippy::pedantic)]
#![deny(clippy::unwrap_used)]
#![deny(clippy::expect_used)]
#![deny(clippy::panic)]
#![deny(clippy::indexing_slicing)]
```

### Quick Choose Guide

| Situation | Use |
|-----------|-----|
| Ban `unsafe` entirely | `#![deny(unsafe_code)]` |
| Strict subset beyond defaults | `clippy::pedantic` |
| Ban specific dangerous patterns (unwrap, panic, indexing) | `clippy::restriction` group lints individually |
| Safety-critical / regulated | Safety-Critical Rust Consortium guidelines (draft) + `#![deny(unsafe_code)]` + pedantic + restriction per-pattern |
| Minimum viable linting | `cargo clippy` (clippy::all, shipped with rustup) |

---

## Python Linting / Type Checking

Python has no single "Good Parts" document like Crockford's, but its design
philosophy ("there should be one obvious way to do it") means fewer bad parts
to ban. Enforcement focuses on type safety and lint strictness instead.

| Tool | What it enforces | How to run | Notes |
|------|-----------------|------------|-------|
| **Ruff** | 800+ lint rules covering pycodestyle, Pyflakes, flake8 plugins, isort, pydocstyle. Also formats code (replaces Black). | `ruff check .` / `ruff format .` | Modern standard. Written in Rust, runs in milliseconds. Replaces Flake8 + isort + Black + more. Actively maintained. |
| **basedpyright** | Strict static type checking. No `Any`, no unchecked index access, full type rigor (per AGENTS.md). | `basedpyright --level strict` or via editor (VS Code + Pylance). | The strictest Python type checker. Catches type errors at compile time. Default per AGENTS.md. |
| **Pylint** | Classic linter: naming conventions, code smells, duplicate code, unused variables, too-many-* complexity checks. | `pylint src/` | Older but still maintained. More opinionated than Ruff on naming and code structure. Can complement Ruff. |
| **mypy** | Gradual type checking. Less strict than basedpyright by default. | `mypy src/ --strict` | Most established type checker. `--strict` enables no-implicit-optional, no- unchecked-indexing, warn-unused-ignores. |

### Recommended Stack (per AGENTS.md)

```bash
# Install
uv venv
uv pip install ruff basedpyright

# Run
ruff check .
ruff format .
basedpyright --level strict
```

### Quick Choose Guide

| Situation | Use |
|-----------|-----|
| Linting + formatting, one dependency | **Ruff** (replaces Flake8 + isort + Black) |
| Strictest type checking (no `Any`, no escapes) | **basedpyright --level strict** |
| Gradual typing, existing mypy codebase | **mypy --strict** |
| Classic code smell detection | **Pylint** (complements Ruff) |
| CI enforcement per AGENTS.md | `ruff format && basedpyright --level strict` |
