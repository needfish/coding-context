# pluto: Input Parsing

> **Pressure class:** Behavior variation  --  the input format changes while the byte-to-structure contract stays the same.

## Problem

Raw bytes from an external source (file, stdin, network, sensor) must be converted into a structured internal representation, detecting or declaring the input format and encoding, preserving source position for error reporting, and handling malformed input without data corruption.

## Context

Arises at every boundary where external data enters the system. It is the outermost transform  --  the conversion from untyped bytes to typed structure. Unlike neptune/transformation (which converts between two structured representations), pluto operates on unstructured input and must recover structure from byte sequences. It precedes validation (jupiter), transformation (neptune), and persistence (mars).

## Domains Involved

- **Byte source** (causal): the origin of raw input  --  a file descriptor, network socket, stdin pipe, memory-mapped buffer, or sensor stream. May be finite (file) or unbounded (network stream).
- **Encoding decoder** (causal): converts byte sequences to character sequences according to a character encoding (UTF-8, ASCII, Latin-1, Shift-JIS). May be implicit (assume UTF-8) or explicit (detect BOM, interpret Content-Type charset).
- **Format detector** (causal): identifies the input format from magic bytes, file extension, MIME type, shebang line, or content analysis. Selects the appropriate parser.
- **Parser** (causal): consumes characters or bytes and produces structured output  --  a parse tree, token stream, record sequence, or event stream. Maintains position state (offset, line, column) through the input.
- **Parse state** (lexical): the parser's current position in the input  --  byte offset, line number, column number, and any nested context (open brackets, current section, indentation stack).
- **Parse error** (lexical): a structured error record containing at minimum a message, source position (byte offset, line, column), severity, and optionally a recovery hint.
- **Structured output** (lexical): the result of parsing  --  a typed tree (AST), record iterator, or event stream. Every significant element carries its source position.
- **Input metadata** (lexical): contextual information about the input source  --  filename, path, URL, content-type, encoding hint, line ending convention, byte order mark presence.

## Shared Phenomena

- Byte source  ->  Parser: byte sequence, EOF signal, read error
- Input metadata  ->  Format detector: filename, extension, content-type hint
- Format detector  ->  Parser: format identifier, parser handle or function pointer
- Encoding decoder  ->  Parser: decoded character sequence (or failed-code-point placeholders)
- Parser  ->  Parse state: current position, accumulated context (nesting stack, mode flags)
- Parser  ->  Structured output: typed elements with position annotations
- Parser  ->  Parse error: error record with position, message, recovery suggestion
- Parse error  ->  Recovery strategy: skip-token, insert-default, switch-mode, abort

## Invariants

- Every byte from the source must be consumed exactly once and in order.
- Every significant element in the structured output must carry a source position (byte offset, line, column) sufficient to locate it in the original input.
- Encoding must be determined before character-level parsing begins. If undetectable, a default must be declared with user-overridable fallback.
- Malformed input must produce at least one error  --  silent data corruption is never acceptable.
- A parse error must not discard already-parsed valid output (partial results are valid if declared).
- Position tracking must not overflow (handles inputs up to at least 2^64 bytes or the platform's address space).
- The parser must not read past available input  --  it must block on incomplete input only when the consumer has requested the next element (pull) or buffer and signal availability (push).
- A successful parse followed by unparse (pretty-print) of the structured output must produce semantically equivalent data  --  not necessarily identical bytes (whitespace, comments may differ).

## Known Failure Modes

- **Encoding misdetection**: Bytes interpreted as the wrong encoding produce garbled output (mojibake) without error, silently corrupting the user's data.
- **Silent truncation**: Input ends mid-structure, and the parser produces incomplete output without error or with a misleading "success" status.
- **Error cascade**: A recovery mechanism misinterprets the remaining input after one error, producing misleading secondary errors.
- **Position overflow**: Input exceeds the maximum trackable offset (2^31 or 2^63 bytes), causing line/column counters to wrap and error locations to become nonsensical.
- **Catastrophic backtracking**: Regex-based or PEG parsers take exponential time on crafted input (ReDoS), causing denial of service.
- **Memory blowup during parse**: Building the full parse tree in memory before streaming causes OOM on large inputs (multi-gigabyte JSON without streaming).
- **Decompression bomb / entity expansion**: Recursive entity expansion (XML billion laughs), anchor aliases (YAML deep nesting), or repeated indirection (JSON with `__proto__` tricks) causes exponential memory growth.
- **Line ending ambiguity**: Mixed CR, LF, and CRLF line endings cause off-by-one column counts or spurious blank lines in line-oriented formats.
- **Reserved code point mishandling**: UTF-8 surrogates, BOM in the middle of a file, or codepoints above U+10FFFF are silently accepted or cause data-dependent crashes.
- **Partial flush**: The byte source provides an incomplete chunk at the end of the transmission (partial UTF-8 sequence, truncated JSON value), and the parser does not hold state across chunks.

## Forces

- **Strict vs. resilient parsing**: Strict parsers reject any deviation from the spec (safe  --  no silent corruption); resilient parsers recover and continue (robust  --  processes more real-world data). The correct choice depends on whether data loss or processing failure is more costly.
- **Pull vs. push control**: Pull parsing (recursive descent, iterator-based) gives the consumer control over pacing  --  parse until you have enough. Push parsing (SAX, event-stream) gives the producer control  --  notify the consumer on each element. Streaming parsers (JSON streaming, XML SAX) use push; most others use pull.
- **Buffered vs. streaming**: Buffering the entire input gives the parser full context for error recovery, format detection, and backtracking, but requires O(N) memory. Streaming processes arbitrarily large inputs in O(1) memory but limits error recovery (cannot reconsume) and format detection (cannot peek ahead without buffering).
- **Grammar coverage vs. spec compliance**: A parser that handles the common 95% of the format will process most real-world files; one that implements every corner case of the spec protects against rare-but-critical edge cases at the cost of complexity. The Pareto principle applies strongly here.
- **Explicit vs. implicit format declaration**: Self-describing input (JSON's first byte, XML's `<?xml`, PNG's magic bytes) makes format detection reliable. Input without format declaration (stdin, unnamed pipe) must use heuristics or require the user to specify.
- **Early vs. late error reporting**: Reporting errors as soon as malformed input is detected gives the user precise location information; deferring errors until after recovery (collect all errors) gives a complete picture at once but may confuse the user with secondary errors.

## Related Subproblems

- **neptune/transformation**: After parsing, data is typically transformed into a different internal representation. Parsing discovers structure; transformation restructures it.
- **jupiter/validation**: Parsed data must often be validated against domain rules, but parsing and validation are distinct  --  parsing recovers structure from bytes, validation checks semantic correctness against rules.
- **rhea/syntax-detection**: Input format detection is a prerequisite for parser selection. Rhea detects the language or format; pluto provides the actual parser.
- **venus/filtering**: Large parsed structures (log files, records) may be filtered before further processing.
- **errors (generic)**: Parse errors must integrate with the system's error model  --  displayed with source position, collected for multi-error reporting.
- **mars/persistence**: Parsed data may be persisted; persisted data may need reparsing on read.
- **callisto/filesystem-walk**: File parsing follows filesystem traversal  --  each discovered file must be parsed.
- **hyperion/network-listener**: Network input parsing follows each socket read.
- **io/command-execution**: Subprocess output is parsed as it arrives via stdout/stderr pipes.

### ==== Practical Use ====

#### When this pattern matters
Any project that reads data from external sources  --  files, stdin, network sockets, sensors, user input. Applies to CLI tools, configuration loaders, compilers, interpreters, data importers, log processors, serialisation libraries, and web servers.

#### Start with (minimal form)
Read all bytes into memory with `fs::read()` / `std::io::read_to_string()` / `sys.stdin.read()`. Assume UTF-8 encoding. Use the standard library's format parser (e.g., `json.loads()`, `serde_json::from_reader()`, `xml.etree.ElementTree`). Abort on the first parse error with a panic or top-level exception. No position tracking, no format detection, no recovery.

```python
import json, sys
data = json.loads(sys.stdin.read())  # assumes UTF-8, panics on first error
```

```rust
use std::fs;
let content = fs::read_to_string(path)?;  // assumes UTF-8
let data: MyStruct = serde_json::from_str(&content)?;  // first error aborts
```

#### Split into its own module when (observable triggers)
- Input arrives in multiple formats from the same source (format detection needed).
- Users report garbled output from files in non-UTF-8 encodings.
- A single malformed input line aborts processing of all subsequent valid input.
- Error messages say "parse error at line 42" but the parser has no position tracking.
- Input is too large to buffer in memory (multi-gigabyte JSON, streaming logs).
- Multiple input sources (stdin, file, network) need different parsing paths.
- The parser needs to be tested independently from the rest of the pipeline.

#### Signs you're in this pattern
- Your system reads data from an external source and converts it to an internal type.
- A malformed input file crashes the entire process instead of producing a useful error.
- Error messages say "invalid input" without saying where or why.
- Encoding issues produce visible garbled characters in output or internal strings.
- You have ad-hoc format detection: `if line.startswith("<?xml")`, `if ext == ".json"`.
- The same parsing logic appears in multiple files with minor variations.

#### Real project examples
- **chibicc** (`tokenize.c`): Reads file content via `read_file()` into a heap buffer, tokenizes with line/column position tracking, assumes UTF-8. Error recovery: skips to next newline and reports all errors after parsing.
- **MiniJinja** (`src/parser.rs`): Template source  ->  tokeniser  ->  AST. Position annotations on every AST node for runtime error reporting. UTF-8 assumed. Single-format input (Jinja2 syntax).
- **DaemonDB** (SQL parser): SQL text  ->  tokeniser  ->  AST  ->  bytecode. Position tracking for error messages across all three stages. Handles flexible whitespace, comments, quoted identifiers.
- **NessMuxer** (packet demuxer): MPEG-TS packets  ->  PES  ->  elementary stream. Three nested parse layers with different position models (packet offset in file, stream PTS timestamp, frame index).
- **miniz** (`tinfl_decompress`): Streaming inflate decompressor  --  reads input bytes incrementally, produces output bytes incrementally. No position tracking (binary format, no error locations).
- **tiny-lsm-go** (`wal.go`): WAL entries parsed from byte slices  --  fixed-length header (CRC, sequence, key-length, value-length) followed by variable-length key/value. Binary format, position implicit in slice offset.
- **picol** (Tcl interpreter source parsing): Reads Tcl script text, tokenises by hand (no regex), builds command AST. Position tracking for error messages via line counter.

#### Common over-engineering traps
- Building a streaming parser before confirming that the input fits in memory (most files do).
- Writing a custom parser for a standard format (JSON, XML, TOML) when the standard library parser handles it.
- Adding error recovery (skip-and-continue) before measuring that first-error abort loses too much work.
- Implementing incremental parsing for a format that requires the full input anyway (JSON without streaming extension, XML with non-streaming DTD validation).
- Building a generic parser combinator framework before you have a second parser to write.
- Supporting multiple character encodings before confirming that non-UTF-8 input actually arrives.
- Adding speculative parsing (try format A, then B, then C) before format detection via extension or magic bytes fails.

## Frame Concern

**The gap between byte sequences and structured semantics.** The producer's intent is only partially encoded in the byte stream  --  encodings are underspecified, format versioning is implicit, spec corner cases arise unknowingly, and human error introduces malformed data at every boundary. The parser must decide what to infer when bytes are ambiguous (encoding detection, format sniffing), when to reject (malformed but possible to interpret), how to recover (skip, default, or abort), and what to tell the user. Every choice sits between robustness (process more real-world data) and safety (never silently corrupt data). Parsing well-formed input is mechanical. The concern is the space between bytes and meaning where structure must be recovered, not given.
