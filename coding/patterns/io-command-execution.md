# ------------------------------------------------------------------------------
#  io-command-execution.md
#  Object variation -- the external command interface changes while the execute-per-item contract stays the same.
# ------------------------------------------------------------------------------

# - Problem -

You have a list of items (files, URLs, values) and you need to run an external command on each one, substituting each item into a command template.

# - When You'll See This -

Any tool that finds results and lets the user act on them -- unzipping archives, opening files in an editor, converting image formats, running analyses on each search result. Command execution sits after a search or collection step and before result collection or display.

# - How It Works -

- **Items**: the collection of paths or values to process.
- **Command template**: the command to run, with a placeholder for the item (like `{}` in `find -exec`).
- **Execution mode**: how items map to commands -- one command per item, batch mode (multiple items per command), or parallel execution.
- **Result collector**: captures stdout, stderr, exit codes, and any output files.

# - What Must Always Be True -

- Every item is processed by the command at least once.
- The command's exit code is checked to detect failures.
- Stderr output is captured and surfaced to the user.
- Long-running or parallel executions respect resource limits (max parallel processes, timeout).

# - What Goes Wrong -

- **Command injection**: an item contains special characters (`;`, `|`, `` ` ``) that change the command's meaning.
- **No failure reporting**: the command fails silently and the item looks processed.
- **Too many processes**: spawning one process per item for 100,000 items overwhelms the system.
- **Path with spaces**: the filename contains spaces and the command interprets it as multiple arguments.
- **Timeout runaway**: a command hangs on one item and blocks everything.
- **Encoding mismatch**: the item has non-ASCII characters that the shell or command doesn't handle.

# - Design Trade-Offs -

- **Sequential vs. parallel**: sequential is simple and predictable; parallel is faster but harder to control resource usage.
- **Shell vs. exec**: using a shell (`sh -c`) gives you pipes and redirects but risks injection; direct exec avoids the shell but gives you fewer features.
- **One-per-item vs. batch**: one command per item is simple but slow; batching reduces process overhead but makes error reporting harder.

# - Related Problems -

- **callisto/filesystem-walk**: walked files are often the items to process.
- **venus/filtering**: filtered results are the items to act on.
- **mercury/display**: results of command execution get displayed.

# - Getting Started -

Use `subprocess.run()` in Python or `std::process::Command` in Rust. Iterate over items, call the command on each one, print stdout/stderr. Sequential only, no batch mode, no parallelism.

# - When to Split Into Its Own Module -

- You need parallel execution with resource limits.
- You need dry-run mode (show what would run without running it).
- You need to collect and report per-item exit codes and errors.
- Multiple command templates or execution modes emerge.

# - Signs You're in This Pattern -

- You're calling `subprocess.run()` or `std::process::Command` in a loop.
- You're constructing command strings by concatenating values.
- You need to run the same external tool on many different inputs.
- Error handling for command failures is scattered across your code.

# - Common Mistakes -

- Using shell=True without sanitizing inputs (command injection).
- Spawning unlimited parallel processes without a concurrency limit.
- Ignoring the exit code and assuming success.
- Not handling filenames with spaces or special characters.

# - The Core Challenge -

External commands are black boxes. You pass them input and get back output (or errors). You can't control how they handle edge cases, and you can't fix bugs in them. The challenge is treating them as reliable components despite having no control over their internals.
