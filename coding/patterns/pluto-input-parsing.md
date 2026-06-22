# ------------------------------------------------------------------------------
#  pluto-input-parsing.md
#  Behavior variation -- the input format changes while the byte-to-structure contract stays the same.
# ------------------------------------------------------------------------------

# - Problem -

Raw bytes arrive from a file, stdin, or network. You need to turn them into structured data your program can work with -- numbers, objects, records, or an AST.

# - When You'll See This -

Every boundary where external data enters your system. CLI tools reading files, config loaders, compilers, interpreters, data importers, log processors, serialisation libraries, web servers. Parsing is the outermost transform -- converting untyped bytes to typed structures.

# - How It Works -

- **Byte source**: where the raw input comes from -- a file, socket, stdin pipe, or memory buffer.
- **Encoding decoder**: converts raw bytes to characters according to an encoding (UTF-8, ASCII, Latin-1).
- **Format detector**: identifies the input format from magic bytes, file extension, MIME type, or content analysis.
- **Parser**: consumes characters and produces structured output -- a parse tree, token stream, or record sequence.
- **Parse state**: where the parser is in the input -- byte offset, line, column, and any nesting context.
- **Parse error**: a record of what went wrong, where, and optionally how to recover.
- **Structured output**: the result -- a typed tree, record iterator, or event stream with position info on each element.

# - What Must Always Be True -

- Every byte from the source is consumed exactly once, in order.
- Every significant output element carries its source position (byte offset, line, column).
- Encoding is determined before character-level parsing starts.
- Malformed input produces at least one error -- silent data corruption is never acceptable.
- A parse error doesn't discard already-parsed valid output (partial results are valid if declared).
- Position tracking handles inputs up to at least 2^64 bytes.
- The parser doesn't read past available input -- it blocks or buffers appropriately.

# - What Goes Wrong -

- **Encoding misdetection**: bytes interpreted as the wrong encoding produce garbled text (mojibake) with no error.
- **Silent truncation**: input ends mid-structure, and the parser produces incomplete output without error.
- **Error cascade**: a recovery mechanism misinterprets the rest of the input after one error, creating misleading secondary errors.
- **Position overflow**: input exceeds position counter limits, making error locations nonsensical.
- **Catastrophic backtracking**: regex-based parsers take exponential time on crafted input (ReDoS attack).
- **Memory blowup**: building the full parse tree in memory causes OOM on large inputs.
- **Decompression bomb**: recursive entity expansion (XML billion laughs, YAML deep nesting) causes exponential memory growth.
- **Partial flush**: the source provides an incomplete chunk at the end and the parser doesn't hold state across chunks.

# - Design Trade-Offs -

- **Strict vs. resilient**: strict parsers reject deviations (safe, no silent corruption); resilient parsers recover and continue (robust, processes more real-world data).
- **Pull vs. push**: pull parsing (recursive descent) lets the consumer control pacing; push parsing (SAX, events) notifies on each element.
- **Buffered vs. streaming**: buffering gives full context for error recovery and backtracking but uses O(N) memory; streaming processes arbitrarily large input in O(1) memory but limits recovery.
- **Explicit vs. implicit format**: self-describing formats (JSON, PNG) make detection reliable; unnamed pipes need heuristics or user specification.

# - Related Problems -

- **neptune/transformation**: parsed data usually needs reshaping.
- **jupiter/validation**: parsed data often needs validation against domain rules.
- **rhea/syntax-detection**: format detection is a prerequisite for parser selection.
- **venus/filtering**: large parsed structures might be filtered before further processing.
- **mars/persistence**: parsed data may be stored; stored data may need reparsing on read.
- **callisto/filesystem-walk**: file parsing follows directory traversal.
- **hyperion/network-listener**: network input parsing follows each socket read.

# - Getting Started -

Read all bytes into memory. Assume UTF-8. Use the standard library's format parser. Abort on the first parse error. No position tracking, no format detection, no recovery.

```python
import json
data = json.loads(sys.stdin.read())  # assumes UTF-8, panics on first error
```

# - When to Split Into Its Own Module -

- Input arrives in multiple formats from the same source (format detection needed).
- Users report garbled output from files in non-UTF-8 encodings.
- A single malformed line aborts processing of all subsequent valid input.
- Error messages say "parse error at line 42" but the parser has no position tracking.
- Input is too large to buffer in memory (multi-gigabyte files, streaming logs).
- Multiple input sources need different parsing paths.

# - Signs You're in This Pattern -

- Your system reads external data and converts it to an internal type.
- A malformed input file crashes the whole process instead of producing a useful error.
- Error messages say "invalid input" without saying where or why.
- Encoding issues produce garbled characters in your output.
- You have ad-hoc format detection: `if line.startswith("<?xml")`.
- The same parsing logic appears in multiple files with minor variations.

# - Common Mistakes -

- Building a streaming parser before confirming the input fits in memory (most files do).
- Writing a custom parser for a standard format when the standard library handles it.
- Adding error recovery before measuring that first-error abort loses too much work.
- Supporting multiple encodings before confirming non-UTF-8 input actually arrives.
- Building a parser combinator framework before you have a second parser to write.

# - The Core Challenge -

The producer's intent is only partially encoded in the byte stream -- encodings are underspecified, format versioning is implicit, and human error introduces malformed data at every boundary. The parser must decide what to infer when bytes are ambiguous, when to reject, and how to recover. Every choice is a trade-off between robustness (process more real-world data) and safety (never silently corrupt data).
