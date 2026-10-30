# iota -- command execution

## Problem

An external program must be run: started, given input, given time to finish, and its output collected. The program may succeed (exit code 0), fail (non-zero exit), or hang (needs timeout).

## Input

- A command string or list of arguments
- Optional stdin data
- Optional environment variables
- Optional timeout duration

## Output

- stdout bytes
- stderr bytes
- Exit code (number)
- Or a timeout error if the command did not finish in time

## Observable Behavior

- Running the same command with the same input produces the same output and exit code
- The command runs in a separate process (does not affect the parent process's memory)
- The parent process can read stdout and stderr separately
- If the command does not finish within the timeout, it is killed and a timeout error is returned

## Minimal Code

```python
import subprocess

def run_command(cmd: list[str]) -> dict:
    result = subprocess.run(cmd, capture_output=True, text=True)
    return {
        "stdout": result.stdout,
        "stderr": result.stderr,
        "exit_code": result.returncode,
    }
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Command not found | subprocess raises FileNotFoundError | Check command exists before running, or catch the error and return a clear message |
| Command hangs | Program waits forever for output | Add a timeout; kill the process if it exceeds the timeout |
| Output too large | Storing all output in memory exhausts memory | Stream output instead of capturing it all; or set a max output size |
| Shell injection | Command string contains user input interpreted as shell syntax | Pass arguments as a list, not a string; use subprocess.run with list args |
| Zombie process | Child process finishes but is not collected | Use subprocess.run (waits for completion) or add a signal handler for SIGCHLD |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Blocking vs. async | Blocking waits for the command to finish, simple but blocks the caller. Async (asyncio.create_subprocess_exec) lets the program do other work while the command runs. |
| Capture vs. stream output | Capture collects all output in memory, simple but may exhaust memory. Stream processes output line by line as it arrives. |
| Shell vs. no shell | Shell (shell=True) allows pipes and redirects but is vulnerable to injection. No shell (list args) is safe but cannot use shell syntax. |

## Start With

Use the language's subprocess module with list arguments (not shell strings) and a timeout. Capture output for small commands; stream for large output.

## Split Into Its Own Module When

- More than one command is run (shared timeout, error handling, logging).
- Commands need to run in parallel.
- Command output needs to be processed in a standard way (parse JSON, extract fields).
- Commands are configurable (the command itself, arguments, or environment are user-provided).

## Signs This Is Present

- subprocess.run, os.system, or similar is called.
- External tools are invoked (git, ffmpeg, curl, compiler).
- Shell scripts are generated and executed.
- Command-line output is parsed.

## Common Mistakes

- Using shell=True with user input (shell injection).
- Not setting a timeout for external commands (program hangs forever).
- Not capturing stderr separately from stdout.
- Assuming the command exists on the system.
- Not handling non-zero exit codes.
- Building command strings instead of argument lists.
