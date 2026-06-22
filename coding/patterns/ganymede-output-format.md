# ------------------------------------------------------------------------------
#  ganymede-output-format.md
#  Data variation -- same rendering pipeline, different styles per output context.
# ------------------------------------------------------------------------------

# - Problem -

Structured data needs to be rendered as human-readable output with the right formatting, escaping, and styling for the target -- a terminal, a web page, a file, or a GUI.

# - When You'll See This -

Anywhere data is formatted for output. CLI tools printing to a terminal, HTML templating for web pages, file export, client-side rendering. Formatting is the final step before output -- it takes structured entries and produces formatted text or DOM nodes.

Variants include:
- **CLI output**: colour-coded, width-constrained terminal display with pipe/file detection.
- **HTML templating**: data interpolated into a fixed template with contextual escaping per slot (HTML body, attribute, URL, script). Includes component composition.

# - How It Works -

- **Data entries**: the structured records or values to format.
- **Output target**: where the output goes -- terminal, file, HTML response, DOM.
- **Formatter**: applies escaping, styling, indentation, and layout rules based on the target's capabilities.
- **Styles**: the visual presentation -- colours, fonts, icons, HTML classes.

# - What Must Always Be True -

- Every data entry produces a corresponding output element (unless explicitly omitted).
- Escaping matches the output context (HTML escaping for HTML, ANSI for terminal, no escaping for plain text).
- The output is valid for its target (valid HTML, valid ANSI, valid CSV).
- Formatting doesn't lose or corrupt data -- numbers stay precise, special characters stay intact.

# - What Goes Wrong -

- **Wrong escaping**: interpolating user text into HTML without escaping creates XSS vulnerabilities.
- **Terminal detection fails**: a tool detects a terminal and uses colours, but output is piped to a file that shows escape codes as garbage.
- **Overflow**: output exceeds terminal width or page size and gets truncated without warning.
- **Encoding mismatch**: the output contains characters the target can't display.

# - Design Trade-Offs -

- **Rich vs. compatible**: rich output (colours, formatting) is nicer but breaks in non-terminal contexts; plain text works everywhere but is bland.
- **Eager vs. lazy formatting**: formatting everything upfront is simple but wastes work if only part is shown; lazy formatting defers work but is more complex.

# - Related Problems -

- **mercury/display**: formatting feeds into display.
- **neptune/transformation**: data often needs transformation before formatting.
- **venus/filtering**: only a subset might need formatting.
- **earth/sorting**: ordered data formats differently.

# - Getting Started -

Print plain text with `print()`. Format a table with simple string padding or `str.format()`. No colours, no HTML, no detection.

# - When to Split Into Its Own Module -

- You need multiple output formats (JSON, CSV, HTML, plain text).
- You need terminal detection (colours when interactive, plain when piped).
- You need contextual escaping (different rules for HTML body vs. attribute vs. URL).
- You need to test formatting independently from data processing.

# - Signs You're in This Pattern -

- Your code mixes data processing with string formatting.
- Changing the output format means editing many files.
- You need to support both human-readable and machine-readable output.
- You're building strings by concatenation and it's getting messy.

# - Common Mistakes -

- Detecting terminal support with environment variables instead of checking stdout.
- Adding HTML templating when plain text output is sufficient.
- Building a custom template engine before trying an existing one.

# - The Core Challenge -

Different output targets have different capabilities, constraints, and security requirements. The same data must render correctly across all of them without repeating the formatting logic.
