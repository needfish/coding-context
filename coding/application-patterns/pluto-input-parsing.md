# pluto -- input parsing

## Problem

Raw bytes from a file, network socket, or user input must be converted into structured data the program can use. The input may be JSON, CSV, XML, a custom binary format, or any other format. Encoding (UTF-8, UTF-16, Latin-1) may need to be detected first.

## Input

- A byte sequence from a source (file, stdin, network, memory buffer)
- Optional encoding metadata

## Output

- Structured data: a dictionary, list of records, AST, or configuration object
- Or a parse error with position (line, column) and message

## Observable Behavior

- Given a valid byte sequence, produce a structured representation
- Given an invalid byte sequence, produce a parse error
- The same bytes always produce the same structured data
- The parser never crashes, no matter what bytes it receives

## Minimal Code

```python
import json

def parse_input(data: str) -> dict:
    return json.loads(data)
```

Use the language's standard library parsers (csv, configparser, xml.etree) before writing a custom parser.

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Encoding ambiguity | Same bytes produce different characters on different machines | Detect encoding (BOM, charset header) or require explicit encoding |
| Catastrophic backtracking | Some inputs take minutes to parse | Use a parser combinator or parser generator with linear-time guarantees |
| Partial input | Input stream ends mid-record, parser returns garbage | Buffer partial data and wait for more input (streaming parser) |
| Memory exhaustion | Loading entire gigabyte file before parsing | Use a streaming parser that processes input incrementally |
| Ambiguous grammar | Input can be parsed two ways, producing different results | Disambiguate the grammar or require explicit markers |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Single-pass vs. multi-pass | Single-pass reads once and produces output, faster but can't handle forward references. Multi-pass handles forward references but uses more memory and time. |
| Fail-fast vs. error recovery | Fail-fast stops at first error with a clear message, reports one error per run. Error recovery skips to next record and reports multiple errors. |
| Hand-written vs. generated | Hand-written (recursive descent) gives good error messages but is labor-intensive. Generated (PEG, LR, parser combinators) is concise but may have opaque error messages. |

## Start With

Use the standard library parser for the input format. Add a custom parser (recursive descent or parser combinator) when the format is custom or complex.

## Split Into Its Own Module When

- More than one input format needs to be supported.
- Input encoding detection is required.
- The input is too large to fit in memory and requires streaming.
- Custom parsing logic beyond what the standard library provides is needed.
- Parse errors need position information (line, column) and error recovery.

## Signs This Is Present

- Raw bytes or strings are converted to structured data.
- A JSON, YAML, CSV, or XML parser is used.
- Format detection or encoding detection is implemented.
- A lexer or tokeniser is implemented.

## Common Mistakes

- Using regex for parsing structured formats (nested structures, context-free grammars).
- Not handling BOM (Byte Order Mark) in UTF-8 and UTF-16 input.
- Loading the entire input into memory before parsing.
- Assuming the input encoding is always UTF-8.
- Not validating the parsed structure after parsing.
- Swallowing parse errors instead of propagating them with position info.
