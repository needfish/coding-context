# Pattern Index

| Planet | Pattern | File | Frame Concern |
|--------|---------|------|---------------|
| mercury | display | [patterns/mercury-display.md](patterns/mercury-display.md) | Temporal coupling between source and display |
| venus | filter | [patterns/venus-filter.md](patterns/venus-filter.md) | Predicate ownership |
| earth | sort | [patterns/earth-sort.md](patterns/earth-sort.md) | Comparator consistency |
| mars | persist | [patterns/mars-persist.md](patterns/mars-persist.md) | Durability guarantee boundary |
| jupiter | validate | [patterns/jupiter-validate.md](patterns/jupiter-validate.md) | Rule set completeness |
| saturn | route | [patterns/saturn-route.md](patterns/saturn-route.md) | Route table consistency |
| uranus | authenticate | [patterns/uranus-authenticate.md](patterns/uranus-authenticate.md) | Trust delegation |
| neptune | transform | [patterns/neptune-transform.md](patterns/neptune-transform.md) | Semantic equivalence across heterogeneous representations |
| callisto | filesystem walk | [patterns/callisto-filesystem-walk.md](patterns/callisto-filesystem-walk.md) | Gap between filesystem order and consumer order |
| io | command execution | [patterns/io-exec.md](patterns/io-exec.md) | Subprocess lifecycle vs. consumer expectations |
| ganymede | output formatting | [patterns/ganymede-output-format.md](patterns/ganymede-output-format.md) | Terminal capability detection |
| europa | configuration assembly | [patterns/europa-config-assembly.md](patterns/europa-config-assembly.md) | Merge ordering transparency |
| titan | syntax highlighting | [patterns/titan-syntax-highlight.md](patterns/titan-syntax-highlight.md) | Grammar coverage vs. source variety |
| rhea | syntax detection | [patterns/rhea-syntax-detection.md](patterns/rhea-syntax-detection.md) | Information deficit at decision time |
| iapetus | code counting | [patterns/iapetus-code-counting.md](patterns/iapetus-code-counting.md) | Grammar coverage vs. real-world source variety |

## Unused Planet Names

`adrastea`, `amalthea`, `ananke`, `ariel`, `atlas`, `belinda`, `bianca`, `caliban`, `calypso`, `carme`, `charon`, `cordelia`, `cressida`, `deimos`, `desdemona`, `despina`, `dione`, `elara`, `enceladus`, `epimetheus`, `helene`, `himalia`, `hyperion`, `iapetus`, `janus`, `juliet`, `larissa`, `leda`, `metis`, `mimas`, `miranda`, `naiad`, `nereid`, `oberon`, `ophelia`, `pan`, `pandora`, `pasiphae`, `phobos`, `phoebe`, `portia`, `prometheus`, `prospero`, `proteus`, `puck`, `rosalind`, `setebos`, `sinope`, `stephano`, `sycorax`, `telesto`, `thalassa`, `thebe`, `tethys`, `titania`, `trinculo`, `triton`, `umbriel`

---

## Real-System Mappings

### fd (sharkdp/fd)

The canonical `fd` file search tool, decomposed by its module structure (`src/` directory).

| fd module | Problem Pattern | Rationale |
|-----------|----------------|-----------|
| `src/cli.rs` | **europa** (configuration assembly) | CLI argument parsing → `Opts` struct → merged into `Config` with env vars and platform defaults |
| `src/config.rs` | **europa** (configuration assembly) | `construct_config()` merges opts, env vars, file types, exclusion rules into a single validated `Config` |
| `src/walk.rs` | **callisto** (filesystem walk) | Parallel directory traversal via `ignore::WalkParallel`, respecting depth, follow-links, one-file-system |
| `src/filesystem.rs` | **callisto** (filesystem walk) | Helper: default path separator, directory existence checks |
| `src/filter/mod.rs` | **venus** (filter) | Pattern matching on file names via regex/glob |
| `src/filter/owner.rs` | **venus** (filter) | Owner-based filtering (unix uid/gid) |
| `src/filter/size.rs` | **venus** (filter) | Size-based filtering |
| `src/filter/time.rs` | **venus** (filter) | Time-based filtering (changed before/after) |
| `src/regex_helper.rs` | **venus** (filter) | Regex construction, smart-case detection, leading-dot detection |
| `src/filetypes.rs` | **venus** (filter) | File type classification (file, dir, symlink, executable, etc.) |
| `src/output.rs` | **ganymede** (output formatting) | Result output with optional colour, paging via `less` if output is long |
| `src/fmt/mod.rs` | **ganymede** / **mercury** (output formatting / display) | Format template parsing: `{}`, `{.}`, `{/}`, `{//}`, `{/.}` path placeholders |
| `src/fmt/input.rs` | **ganymede** (output formatting) | Per-entry formatted string construction from template |
| `src/hyperlink.rs` | **ganymede** (output formatting) | Hyperlink (OSC 8) generation for terminal output |
| `src/exec/mod.rs` | **io** (command execution) | `CommandSet`: defining the command template with placeholder tokens |
| `src/exec/command.rs` | **io** (command execution) | Per-item command instantiation: substituting `{}`, `{.}`, etc. |
| `src/exec/job.rs` | **io** (command execution) | Parallel job execution: thread pool, batch mode, output collection |
| `src/dir_entry.rs` | **neptune** (transform) | `DirEntry` wrapper type: filesystem entry → displayable value |
| `src/error.rs` | **jupiter** (validate) | Error type definitions and formatted error output |
| `src/exit_codes.rs` | **jupiter** (validate) | Exit code enumeration and aggregation |
| `src/main.rs` | (orchestration) | Top-level pipeline: parse → walk → filter → format → execute |

### bat (sharkdp/bat)

A `cat(1)` clone with syntax highlighting and Git integration.

| bat module | Problem Pattern | Rationale |
|------------|----------------|-----------|
| `src/bin/bat/main.rs` | **europa** (configuration assembly) | CLI argument parsing via `clap`, environment variable reading |
| `src/config.rs` | **europa** (configuration assembly) | `Config` struct assembled from CLI args, env vars, config file (`bat.conf`), and platform defaults |
| `src/input.rs` | **callisto** / **mercury** (filesystem walk / display) | File opening, stdin reading, encoding detection — input source abstraction |
| `src/controller.rs` | (orchestration) | Top-level pipeline: inputs → syntax detection → highlighting → formatting → paging → output |
| `src/syntax_mapping.rs` | **rhea** (syntax detection) | Maps file extension, filename, shebang, or content to Sublime Text syntax language |
| `src/assets.rs` | **rhea** / **titan** (syntax detection / highlighting) | Compiled syntax definitions and theme assets (serialized `.sublime-syntax` and `.tmTheme`) |
| `src/pretty_printer.rs` | **titan** (syntax highlighting) | Public API: syntax-highlighted output via syntect engine |
| `src/printer.rs` | **titan** / **ganymede** (highlighting / output formatting) | `InteractivePrinter` (highlighted + styled) vs `SimplePrinter` (plain — acts like `cat`) |
| `src/decorations.rs` | **ganymede** (output formatting) | Line numbers, grid lines, file header decorations |
| `src/diff.rs` | **venus** (filter) + **mercury** (display) | Git diff integration: showing modification markers (+, -, ~) in the margin |
| `src/theme.rs` | **europa** (configuration assembly) | Color theme selection and application to token scopes |
| `src/terminal.rs` | **ganymede** (output formatting) | Terminal width detection, colour capability detection |
| `src/output.rs` | **saturn** (route) | `OutputHandle` abstraction: routes output to terminal, pipe, file, or pager |
| `src/paging.rs` | **saturn** (route) | Pager selection and configuration (less), decides when to engage |
| `src/pager.rs` | **io** (command execution) | Pager process lifecycle management (spawning less with args) |
| `src/less.rs` | **io** (command execution) | `less` environment variable configuration for bat integration |
| `src/wrapping.rs` | **ganymede** (output formatting) | Line wrapping mode: char wrap, word wrap, no wrap |
| `src/nonprintable_notation.rs` | **neptune** (transform) | Non-printable character representation — replace visible chars with notation |
| `src/line_range.rs` | **venus** (filter) | Line range filtering (show only lines N–M) |
| `src/style.rs` | **europa** (configuration assembly) | Style component selection (numbers, grid, header, snip) as bitmask from CLI |
| `src/error.rs` | **jupiter** (validate) | Error type definitions and formatted error messages |

### tokei (XAMPPRocky/tokei)

A code line counter supporting 150+ languages with per-file and per-language statistics.

| tokei module | Problem Pattern | Rationale |
|--------------|----------------|-----------|
| `src/cli.rs` | **europa** (configuration assembly) | CLI argument parsing: columns, sort order, output format, exclusion patterns |
| `src/config.rs` | **europa** (configuration assembly) | `Config` merged from CLI args, config file (`tokei.toml`/`.tokeirc`), and platform defaults |
| `src/input.rs` | **callisto** (filesystem walk) | Directory traversal respecting `.gitignore`, `.ignore`, and `.tokeignore` files |
| `src/language/languages.rs` | **rhea** (syntax detection) | Extension → language mapping from `languages.json` definition file |
| `src/language/syntax.rs` | **iapetus** (code counting) | Per-language state machine: classifies lines as code, comments, blanks, doc comments |
| `src/language/embedding.rs` | **iapetus** (code counting) | Embedded language support (e.g., Markdown containing Rust code blocks) |
| `src/language/language_type.rs` | **iapetus** / **neptune** (code counting / transform) | Language enum from JSON definitions, compiled via build script |
| `src/stats.rs` | **iapetus** (code counting) | Per-file `Stats` struct: line counts by category, aggregated into language totals |
| `src/sort.rs` | **earth** (sort) | Output sorting by files, lines, code, comments, or blanks column |
| `src/consts.rs` | **neptune** (transform) | Language name normalisation constants |
| `src/utils/fs.rs` | **callisto** (filesystem walk) | Filesystem helpers (canonicalise path, check if directory) |
| `src/utils/ext.rs` | **rhea** (syntax detection) | Extension extraction with edge case handling |
| `src/lib.rs` | (orchestration) | Public API: `Languages::get_statistics()` — walk → detect → count → aggregate → sort → format |

### pet (knqyf263/pet)

A CLI snippet manager with save, tag, search, sync, and execute features.

| pet module | Problem Pattern | Rationale |
|------------|----------------|-----------|
| `main.go`, `cmd/root.go` | **europa** (configuration assembly) | CLI argument parsing, config file loading (`~/.config/pet/config.toml`) |
| `config/config.go` | **europa** (configuration assembly) | Config struct: editor, selector, sync settings, tag filtering |
| `cmd/new.go` | **mars** (persist) | Create snippet: validates input, appends to snippet TOML file |
| `cmd/edit.go` | **mars** (persist) | Read snippet file into editor, write back on save |
| `cmd/list.go` | **mars** (persist) + **mercury** (display) | Read all snippets from file and display them |
| `cmd/search.go` | **venus** (filter) + **mercury** (display) | Interactive fuzzy search via fzf/peco, filter by tag, display results |
| `cmd/exec.go` | **io** (command execution) | Select snippet, substitute parameters, execute shell command |
| `cmd/clip.go` | **mercury** (display) | Copy snippet to clipboard (OS-specific) |
| `cmd/sync.go` | **mars** (persist) | Sync snippets to remote (Gist, GitLab) — push/pull with conflict resolution |
| `sync/gist.go`, `sync/gitlab.go`, `sync/ghe.go` | **mars** (persist) + **uranus** (authenticate) | Remote storage backends with token-based authentication |
| `dialog/view.go` | **mercury** (display) | Interactive TUI dialog for snippet selection and parameter entry |
| `snippet/snippet.go` | **neptune** (transform) | Snippet data model with variable/parameter expansion (`<param>`, `<param=default>`) |
| `path/path.go` | **neptune** (transform) | Platform-specific path resolution and expansion |

### dust (bootandy/dust)

A more intuitive `du` replacement showing disk usage as a tree with proportional bars.

| dust module | Problem Pattern | Rationale |
|-------------|----------------|-----------|
| `src/cli.rs` | **europa** (configuration assembly) | CLI argument parsing: display depth, sort order, size format, filter options |
| `src/config.rs` | **europa** (configuration assembly) | Config from CLI args + optional config file (`~/.config/dust/config.toml` or `~/.dust.toml`) |
| `src/dir_walker.rs` | **callisto** (filesystem walk) | Parallel filesystem traversal with symlink handling, device boundary control |
| `src/node.rs` | **neptune** (transform) | Tree node data model: path, size, children, aggregated totals |
| `src/filter.rs`, `src/filter_type.rs` | **venus** (filter) | Size-based (`-z`) and regex-based (`-e`, `-v`) filtering, file vs. directory filter (`-D`, `-F`) |
| `src/display.rs`, `src/display_node.rs` | **ganymede** / **mercury** (output / display) | Tree display with proportional ASCII bars, coloured by depth, percentage labels |
| `src/progress.rs` | **mercury** (display) | Spinner / progress indicator during filesystem walk |
| `src/platform.rs` | **neptune** (transform) | Platform-specific path and metadata handling (Unix vs. Windows) |
| `src/utils.rs` | (helpers) | Sorting, terminal width detection |
