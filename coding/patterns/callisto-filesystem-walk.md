# ------------------------------------------------------------------------------
#  callisto-filesystem-walk.md
#  Data variation -- same traversal algorithm, different exclusion rules and root paths.
# ------------------------------------------------------------------------------

# - Problem -

You need to explore a directory tree -- visiting every file, subdirectory, and symlink -- while respecting rules about what to skip, whether to follow symlinks, and how deep to go.

# - When You'll See This -

Any time a system needs to enumerate filesystem contents. Search tools, backup software, file managers, build systems, linters. The walk almost always feeds into filtering, display, transformation, or command execution.

# - How It Works -

- **Root path**: the starting directory for the traversal (or multiple starting points).
- **Entry**: each item found during the walk -- a file, directory, or symlink.
- **Exclusion rules**: patterns or conditions that decide which entries to skip (.git, node_modules, *.tmp).
- **Visitor**: what you do with each entry -- collect it, process it, display it.

# - What Must Always Be True -

- Every reachable entry is visited exactly once (unless explicitly excluded).
- Symlinks are handled consistently -- either always followed or never followed.
- Directory loops (symlink cycles) don't cause infinite traversal.
- The walk respects the specified depth limit and exclusion rules.
- Errors on individual entries (permission denied, broken symlink) don't abort the entire walk.

# - What Goes Wrong -

- **Symlink loops**: a symlink pointing to a parent directory creates an infinite loop.
- **Permission denied**: the walk stops entirely when it can't read a directory.
- **Too many files**: traversing a directory with millions of files uses too much memory or time.
- **Encoding issues**: filenames with unusual characters cause crashes or skipped files.
- **Race conditions**: files are added or deleted during the walk, producing inconsistent results.
- **Exclusion rule mistakes**: a pattern like `*.tmp` doesn't match `.tmp` files in subdirectories.

# - Design Trade-Offs -

- **Breadth-first vs. depth-first**: BFS uses more memory but respects directory ordering; DFS is lighter but goes arbitrarily deep.
- **Eager vs. lazy**: collecting all entries first gives you the full picture; yielding entries as you find them lets you start processing immediately.
- **Parallel vs. single-threaded**: parallel traversal is faster on SSDs but harder to control depth and exclusions.

# - Related Problems -

- **venus/filtering**: visited entries usually get filtered (keep only .py files).
- **mercury/display**: results get displayed to the user.
- **neptune/transformation**: file paths might need conversion.
- **io/command-execution**: each visited file might be processed by an external command.

# - Getting Started -

`os.walk()` in Python, `walkdir::WalkDir` in Rust, or `fs.readdirSync` recursively in JavaScript. Collect entries in a list and print them.

# - When to Split Into Its Own Module -

- You need configurable exclusion rules (ignore files, pattern matching).
- You need to control traversal depth or follow symlinks selectively.
- The walk logic becomes tangled with what you do with the results.
- You need to test traversal behavior independently.

# - Signs You're in This Pattern -

- You have a recursive function that lists files in a directory.
- You're passing around directory paths and accumulating file lists.
- You need to skip certain folders but the skip logic is mixed with the loop.
- Your backup, search, or build tool walks the filesystem.

# - Common Mistakes -

- Writing a recursive traversal when `os.walk` handles it.
- Forgetting to handle permission errors gracefully.
- Not detecting symlink loops and crashing with a stack overflow.
- Loading all file paths into memory before processing the first one.

# - The Core Challenge -

The filesystem is not a clean tree -- it has symlinks, permissions, race conditions, and encoding surprises. A walk must be robust to all of these while still visiting every intended entry exactly once.
