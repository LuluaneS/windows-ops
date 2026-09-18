## `pythonw`

Under `pythonw.exe`, `sys.stdout` and `sys.stderr` can be `None`. Guard before output or logging setup:

```python
import os
import sys

if sys.stdout is None:
    sys.stdout = open(os.devnull, "w", encoding="utf-8")
if sys.stderr is None:
    sys.stderr = open(os.devnull, "w", encoding="utf-8")
```

Do not use `pythonw.exe` for long-running helpers that need to spawn or control interactive children through `wexpect`, ConPTY, or TUI automation. It can make the child exit with EOF before the ready state. Use `python.exe` with hidden/no-window creation flags and explicit log/stdout handling instead.

## Hidden Child Processes

When suppressing Windows flash windows, apply hidden/no-window flags to every child process path that can create a console, not only the top-level launcher.

- Python code that internally calls `powershell.exe` should use `subprocess.CREATE_NO_WINDOW` and, when applicable, `STARTUPINFO` with `STARTF_USESHOWWINDOW` / `SW_HIDE`.
- In Node-based CLI/ACP launchers, inspect `detached` and command-string shell fallback when consoles still flash despite `windowsHide`. On the affected Windows path, removing detached can restore both hidden execution and captured output; prove it with the actual launcher rather than applying a blanket cross-platform change.
- A launcher fix must preserve cancellation of its owned process tree and unrelated processes. A Python venv redirector may add a parent level, so verify actual ancestry instead of assuming one executable equals one process.
- Keep behavior unchanged while hiding the window. Status scans, stop actions, and port checks should use the same command and parsing logic after the flag change.
- Do not switch to `pythonw.exe` merely to hide a helper that controls interactive children; prefer `python.exe` plus creation flags.

## Launchers

Startup folder: keep only launchers or shortcuts there, not logs, data, debug artifacts, markdown, or config files.

Do not assume a Startup-launched VBS script's working directory is its own
directory. Use absolute paths or a shortcut with an explicit working directory.

Keep `.bat` files ASCII-only. Use PowerShell or UTF-8 sidecars for rich text.

### Shortcut And Icon Card

- Keep a custom icon in a stable project asset path, not a temp folder or generated-image cache. Prefer a transparent multi-size `.ico` that includes small Explorer sizes and a 256 px frame.
- Preserve the original `.lnk` before editing it, and change only the requested fields.
- `WScript.Shell.CreateShortcut()` may return blank properties for a working shortcut whose path or filename contains non-ASCII text. Treat blank readback as an API limitation, not proof that the shortcut is broken.
- For an existing non-ASCII shortcut, use `Shell.Application`: resolve the desktop folder, `ParseName()` the leaf, obtain `GetLink`, then use `SetIconLocation()` and `Save()`.
- Read back target path, working directory, arguments, icon path, and icon index. Restore the original on any mismatch. Refresh Explorer icons only after successful readback; do not restart Explorer by default.

### Silent Service Launcher Card

Use for VBS/Startup/`pythonw` services such as a local dashboard.

- Put single-instance and port-health guards in the service, not only the launcher.
- Write one launch-log line before spawning the service.
- Keep service stdout/stderr in a separate stdio log; do not share the service-owned launch log.
- If the port is occupied, health-check the expected service before starting another instance.
- If the port is occupied but unhealthy, log owner PIDs and exit; do not auto-kill.
- For Python `http.server` / `ThreadingHTTPServer` services on Windows, do not rely on launcher checks alone. Set a service-owned single-instance guard such as `allow_reuse_address = False` plus `SO_EXCLUSIVEADDRUSE` before bind, or another service-owned lock, so repeated starts cannot leave multiple listeners on the same localhost port.
- Hidden/no-window flags, `pythonw`, `WScript.Shell.Run`, and `Start-Process`
  change presentation, not process ownership. If a service must survive Codex
  exit, launch it through a verified broker outside the caller-owned Job Object
  (for example an appropriately configured Windows service or scheduled task), then prove
  out-of-job ancestry or survival after closing the initiating Codex Desktop.
  HTTP/API health while that caller remains open is not persistence evidence.
- Verify by launching twice: the second launch exits, `netstat` shows one `LISTENING` owner for the port, and the health URL still returns OK.
