# Safety Tools

## JavaScript Linting / Good Parts Enforcement

| Tool | Install | Enforces | Notes |
|---|---|---|---|
| goodparts (dwyl) | npm install --save-dev goodparts | Crockford Good Parts + ES6 "Better Parts" | Recommended default. Bans ==, with, eval, var, implied globals, bitwise ops. |
| eslint-config-airbnb | npm install --save-dev eslint-config-airbnb | Airbnb style guide | Most popular JS style guide. Includes React + import rules. |
| eslint-config-xtrict (areknawo) | npm install --save-dev eslint-config-xtrict | Extreme strictness, TypeScript-first | TypeScript with full strictTypeChecked. |
| ESLint built-in: recommended | (included with ESLint) | Core correctness rules only | Minimal starting point. |

| Situation | Use |
|---|---|
| Plain JS, want Crockford Good Parts for modern code | goodparts |
| TypeScript project, strictest possible | eslint-config-xtrict or typescript-eslint strictTypeChecked |
| Need React / JSX linting | eslint-config-airbnb or add eslint-plugin-react |
| Minimal setup, just catch real bugs | eslint:recommended |

## C/C++ Linting / Static Analysis

| Tool / Standard | What it enforces | How to run | Notes |
|---|---|---|---|
| clang-tidy | Modular checks: bugprone, performance, portability, readability, modernize, cppcoreguidelines, cert, misc | clang-tidy file.cpp --checks='*' or via CMake | Open source, LLVM project. Highly configurable. |
| C++ Core Guidelines (isocpp) | Type safety, bounds safety, lifetime safety, RAII, rules for interfaces, resources, concurrency | Via clang-tidy checks (cppcoreguidelines-*) | Standard "Good Parts" for modern C++. Free online at https://isocpp.github.io/CppCoreGuidelines/. |
| Cppcheck | Undefined behavior, memory leaks, null pointer derefs, STL misuse | cppcheck --enable=all file.cpp | Path-sensitive: catches bugs clang-tidy misses. |
| clang static analyzer | Null derefs, memory leaks, dead stores, API misuse | clang --analyze or via scan-build | Complements clang-tidy. |
| Microsoft C++ Core Check | C++ Core Guidelines rules, plus bounds, lifetime, concurrency | Visual Studio: enable via Code Analysis | Free with Visual Studio Community. |

### Recommended Stack

```bash
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wpedantic -Werror")
set(CMAKE_CXX_CLANG_TIDY "clang-tidy;-checks=clang-diagnostic-*,clang-analyzer-*,cppcoreguidelines-*,performance-*,portability-*,readability-*")
cppcheck --enable=all --suppress=missingIncludeSystem .
```

| Situation | Use |
|---|---|
| Modern C++ (C++11+), want a "Good Parts" subset | C++ Core Guidelines via clang-tidy -checks=cppcoreguidelines-* |
| General C/C++ linting | clang-tidy with modular check groups |
| Path-sensitive bug finding (UB, leaks) | Cppcheck or clang static analyzer |
| Catch bugs at compile time, zero config | -Wall -Wextra -Wpedantic -Werror |

## Rust Linting / Restrictions

| Tool / Config | What it enforces | How to enable | Notes |
|---|---|---|---|
| #![deny(unsafe_code)] | Bans all unsafe blocks, functions, traits, and methods | Add #![deny(unsafe_code)] at crate root | Compiler built-in. Zero-dependency. |
| clippy::pedantic | 150+ additional lints beyond default | #![warn(clippy::pedantic)] at crate root | Official. Cherry-pick over enabling wholesale. |
| clippy::restriction | Lints that restrict language feature use: ban unwrap(), panic!, indexing, println! | #![warn(clippy::restriction)] or per-lint | Each lint is opt-in. |
| clippy::nursery | New lints still under development | #![warn(clippy::nursery)] | May contain false positives. |
| Safety-Critical Rust Consortium | Coding guidelines for safety-critical Rust | Manual review + custom lints | Under development by Rust Foundation. |

### Recommended Strict Config

```rust
#![deny(unsafe_code)]
#![deny(clippy::all)]
#![deny(clippy::pedantic)]
#![deny(clippy::unwrap_used)]
#![deny(clippy::expect_used)]
#![deny(clippy::panic)]
#![deny(clippy::indexing_slicing)]
```

| Situation | Use |
|---|---|
| Ban unsafe entirely | #![deny(unsafe_code)] |
| Strict subset beyond defaults | clippy::pedantic |
| Ban specific dangerous patterns (unwrap, panic, indexing) | clippy::restriction group lints individually |
| Safety-critical / regulated | Safety-Critical Rust Consortium + #![deny(unsafe_code)] + pedantic + restriction per-pattern |
| Minimum viable linting | cargo clippy (clippy::all) |

## Python Linting / Type Checking

| Tool | What it enforces | How to run | Notes |
|---|---|---|---|
| Ruff | 800+ lint rules covering pycodestyle, Pyflakes, flake8 plugins, isort, pydocstyle | ruff check . / ruff format . | Modern standard. Written in Rust. Replaces Flake8 + isort + Black + more. |
| basedpyright | Strict static type checking. No Any, no unchecked index access, full type rigor | basedpyright --level strict | The strictest Python type checker. |
| Pylint | Classic linter: naming conventions, code smells, duplicate code | pylint src/ | More opinionated than Ruff on naming and structure. |
| mypy | Gradual type checking | mypy src/ --strict | Most established type checker. |

### Recommended Stack

```bash
uv venv
uv pip install ruff basedpyright
ruff check .
ruff format .
basedpyright --level strict
```

| Situation | Use |
|---|---|
| Linting + formatting, one dependency | Ruff |
| Strictest type checking (no Any, no escapes) | basedpyright --level strict |
| Gradual typing, existing mypy codebase | mypy --strict |
| Classic code smell detection | Pylint (complements Ruff) |
| CI enforcement | ruff format && basedpyright --level strict |
