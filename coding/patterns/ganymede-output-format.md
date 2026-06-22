# ganymede: Output Formatting

> **Pressure class:** Data variation  --  same rendering pipeline, different styles per output context.

## Problem

Structured data must be rendered as human-readable output, applying contextual transformations (escaping, styling, indentation) based on data attributes, the output target's capabilities, and user preferences.

## Context

Arises wherever data is formatted for an output target  --  a terminal (CLI tool), a web browser (HTML templating), a file (export), or a DOM (client-side rendering). The formatting is the final step before output; it consumes structured entries and produces formatted output in the target format. It may or may not involve styling (colours, icons, HTML tags, DOM nodes), depending on the output target's capabilities.

Variants include:
- **CLI output**: Colour-coded, width-constrained terminal display with pipe/file detection.
- **HTML templating**: Data interpolated into a fixed template with contextual escaping per slot position (HTML body, attribute, URL, script). Includes component composition and multi-target rendering (server, client, build). Found in JSX, Astro, Svelte, KitaJS, Vue SFCs.
- **Export formatting**: CSV, JSON, XML serialisation with encoding and schema considerations.

## Domains Involved

- **Entries** (lexical): structured data items to be formatted. May be typed fields (path, size, date) or arbitrary data objects.
- **Format template** (lexical): the specification of how each entry is rendered. A format string, a template file, a component tree, or inline expressions.
- **Style rules** (lexical): the mapping from entry attributes to visual styling. ANSI escape codes, CSS classes, colour palette config.
- **Output target** (causal): the destination for formatted output  --  terminal, pipe, file, browser, DOM. Determines encoding, escaping rules, and styling support.
- **Escaping rules** (causal): per-context transformations that prevent data from breaking the output structure (HTML entities, ANSI filtering, URL encoding, CSV quoting).

## Shared Phenomena

- Entries  ->  Formatter: attribute values, metadata
- Format template  ->  Formatter: format string, field specifications, component references
- Style rules  ->  Formatter: colour codes, CSS classes, escape sequences
- Output target  ->  Formatter: width constraints, encoding, escaping rules, styling support
- Escaping rules  ->  Formatter: per-context escape function selection (HTML body vs attribute vs URL)
- Formatter  ->  Output: formatted string, DOM nodes, or file content

## Invariants

- Formatting must not lose or alter data  --  it transforms representation, not content.
- Contextual transformations (escaping, styling) must be applied only when the output target supports them, and in the right position.
- The same data + same format + same target must produce the same output.
- An error in one entry must not corrupt surrounding formatted output (graceful degradation).
- The template structure must be resolvable independently of the data being formatted.

## Known Failure Modes

- **Garbled output to pipe**: Escape codes intended for a terminal are piped into a file or another command, producing visible garbage.
- **Cross-site scripting (XSS)**: Data containing `<script>`, `onerror`, or active content is inserted into HTML without escaping, allowing injection.
- **Context mismatch**: A value escaped for one target context is inserted where a different escaping rule applies (HTML body vs attribute vs URL; terminal pipe vs screen).
- **Double-escaping**: A value already escaped is escaped again, displaying literal `&amp;lt;` or `\e[31m`.
- **Encoding mismatch**: The formatted output contains characters the target cannot represent (terminal encoding, HTML charset).
- **Width overflow**: The formatted line exceeds the target's width (terminal wrap, HTML container overflow).
- **Inconsistent alignment**: Variable-width characters break column alignment when the formatter assumes fixed-width cells.
- **Hydration mismatch**: Server-rendered HTML markup does not match client-side DOM expectations, causing re-render flicker.

## Forces

- **Richness vs. portability**: Richly styled output (HTML, terminal colours) is informative but only works on supporting targets; plain text works everywhere.
- **Compile-time vs. runtime formatting**: Compile-time engines (Svelte, Astro, KitaJS) erase formatting overhead at build time but fix the output strategy early. Runtime formatters (Handlebars, Jinja2) are flexible but have per-render cost.
- **Safety vs. convenience**: Auto-escaping is safe but blocks legitimate raw output. Explicit opt-out is safer but burdens the developer.
- **Component composition vs. flat formatting**: Composable formatting fragments (components, partials) enable reuse at the cost of a more complex rendering model. Flat templates are simpler but duplicate structure.
- **Breadth vs. depth**: Single-line format per entry is compact; multi-line is informative.
- **Streaming vs. buffered**: Streaming output (progressive HTML, piped terminal output) reduces latency but complicates error recovery.
- **Default vs. customisable**: Sensible defaults cover most cases; customisable templates add complexity.

## Related Subproblems

- **mercury/display**: Output formatting feeds display.
- **neptune/transformation**: Entries may need transformation before formatting.
- **venus/filtering**: Only filtered entries need formatting.
- **earth/sorting**: Sorted entries are formatted in order.
- **callisto/walk**: File system entries are the most common input to formatting.

### ==== Practical Use ====

#### When this pattern matters
Any project that transforms structured data into human-readable output  --  CLI tools, web frameworks, static site generators, email renderers, export utilities.

#### Start with (minimal form)
Print raw values with `println!()`. For CLI colour, use ANSI escape codes directly. For HTML, use the framework's built-in template engine (JSX, Svelte, Astro, KitaJS) with auto-escaping. Single format, no template abstractions, no auto-detection.

```tsx
// Minimal HTML templating: framework auto-escaping
function Greeting({ name }) {
  return <p>Hello, {name}</p>;  // name is auto-escaped
}
```

```python
# Minimal CLI formatting: raw print
for entry in entries:
    print(f"{entry.name:30} {entry.size:>10}")
```

#### Split into its own module when (observable triggers)
- You need multiple output formats (plain, JSON, CSV, HTML, email).
- Output target capability detection matters (colour support, HTML escaping context, width).
- The output includes styled elements (colours, icons, CSS) that need testing.
- The same data must render on both server and client (isomorphic rendering).
- Users ask for custom format templates or component composition across packages.

#### Signs you're in this pattern
- Data is transformed through a template or format string before being sent to an output target.
- The same data must render differently depending on the target (terminal vs HTML vs JSON).
- The output looks wrong when the target's capabilities are mis-detected.
- Output includes conditional escaping, styling, or formatting based on data attributes.
- Template expressions must be escaped differently depending on their position in the output.

#### Real project examples
- **crawley** (`output` module): Detects terminal width for table formatting, supports JSON/CSV/plain output via CLI flags.
- **Helix editor** (status line renderer): Formats status information into a fixed-width line with colour-coded segments.
- **Astro** (`.astro` files): Compiles component templates to static HTML at build time with auto-escaping per slot position.
- **KitaJS** (tagged templates): Compile-time HTML templating via tagged template literals with auto-escaping.
- **Svelte** (`.svelte` files): Compiles template expressions to DOM API calls with auto-escaping and reactive update wiring.

#### Common over-engineering traps
- Building a template engine before verifying the framework's built-in rendering is insufficient.
- Implementing auto-detection before measuring that incorrect defaults cause problems.
- Separating format definition from format application before you have a second format.
- Creating an elaborate component hierarchy before the UI has stabilised.
- Mixing compile-time and runtime expressions in the same template without a clear boundary.

## Frame Concern

**Output context detection.** The formatter must decide which transformations (escaping, styling, encoding) to apply per slot before it knows all properties of the output target  --  terminal vs pipe, HTML body vs attribute vs URL, server vs client, colour depth, width, encoding. The concern is how to encode the output context in the template structure so the formatter applies the right transformation to each data value without requiring the developer to specify it manually.
