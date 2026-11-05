---
name: check-python-script
description: Check whether a specified Python script compiles, runs, and satisfies stated expected behavior without assuming a zero exit code proves correctness.
tools:
  - ReadFile
  - RunInTerminal
---

# Check a Python script

Use when a user asks whether a particular Python script works. A script path (or code pasted by the user) and any required arguments or inputs must be identifiable; ask for the missing item if they are not.

1. Resolve the exact script in the workspace and read it before execution. Treat the source as untrusted. Review imports, arguments, expected inputs, file writes, subprocesses, network calls, credentials, and potential destructive or production effects. Do not execute it if these effects are unsafe, the target is ambiguous, or authorization is insufficient; report what can be checked statically instead. Do not run scripts obtained only from unverified external links.
2. Determine the expected result from the user's requirements, documentation, or focused tests. If none is specified, report only syntax and execution status, not that the script is functionally correct. Use the script's directory as the working directory when that matches its documented invocation; do not invent arguments or install dependencies without need.
3. Check compilation without writing bytecode or executing the script. Pass the path as a quoted positional argument, not as source pasted into a shell command:

   ```bash
   python3 -B - '/absolute/path/to/script.py' <<'PY'
   import pathlib
   import sys

   path = pathlib.Path(sys.argv[1])
   compile(path.read_bytes(), str(path), "exec")
   print("Syntax OK")
   PY
   ```

4. If the safety review permits it, run the script once with a bounded timeout and bytecode disabled, using the needed documented arguments. For a script with no arguments:

   ```bash
   timeout 10s python3 -B -- '/absolute/path/to/script.py'
   ```

   Quote paths containing spaces. Do not interpolate untrusted path text into shell syntax or use `eval`. If the script needs dependencies, services, secrets, or special input that are unavailable, stop and identify the missing prerequisite. Do not retry an authorization failure. If targeted tests exist, run only the relevant tests under a timeout and report them separately.
5. Compare observed stdout, stderr, exit status, and any expected outputs or assertions with the stated behavior. A zero exit code alone establishes only that this invocation completed. On failure, provide the exact failing step, short error excerpt, and a focused fix or missing input. Never change the user's code unless asked.

Report: script checked; syntax PASS/FAIL; runtime PASS/FAIL/NOT RUN; expected behavior PASS/FAIL/UNKNOWN; command and observed output (redacting secrets); limitations and next step. Keep the report concise.
