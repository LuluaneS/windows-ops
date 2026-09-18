## Windows Text Edit Safety

Use this card before editing Chinese, emoji, Markdown, prompts, templates, localization files, YAML/JSON snippets, or exact-replacement text on Windows.

- Identify the target shell and the file's encoding/newline risk before writing. Unknown legacy files are encoding-sensitive until proven otherwise.
- In Windows PowerShell 5.1, do not trust implicit `Set-Content`, `Out-File`, redirection, or native-pipe encoding for non-ASCII text; its commands use inconsistent legacy defaults. Use explicit encoding or a file-backed route.
- PowerShell 7 normally uses UTF-8 without BOM for new text output and UTF-8 for native pipes. That does not prove a legacy file is UTF-8 or an external program expects UTF-8. When byte format is contractual, specify it and verify bytes.
- Prefer `apply_patch` for small source or Markdown edits. Preserve surrounding newline style and avoid whole-file rewrites.
- Use Node.js or Python with explicit UTF-8 I/O only when the file is known UTF-8. Do not run UTF-8 rewrite helpers over GBK/CP936/BOM/unknown files without a separate conversion decision and backup.
- Python's default text-file encoding is independent of PowerShell's pipeline encoding. Use `Path.read_text(encoding="utf-8")` and `write_text(..., encoding="utf-8")`, or run a trusted UTF-8-only tool with `python -X utf8`; an unqualified `read_text()` failure can be the validator's locale bug rather than invalid source.
- For exact replacement, make the old text uniquely specific and fail closed when it is missing or appears more than once.
- When PowerShell or Python behavior is uncertain, verify against official PowerShell encoding docs or `PSScriptAnalyzer` guidance instead of relying on habit.

## PowerShell Stdin Encoding

Windows PowerShell 5.1 and legacy console/native-command paths can transcode here-strings piped into `python -`, and quoting can damage inline `python -c` payloads before Python sees them. Non-ASCII such as Chinese markers may arrive as literal `?` bytes, so marker checks like `"取代" in marker_text` can fail even when rendered output looks plausible. PowerShell 7 improves the native-pipe default, but external-program expectations and quoting remain separate failure points.

For scripts or tests containing non-ASCII:

1. Put the script in a UTF-8 `.py` file, then run `python file.py`.
2. Avoid `$here | python -` for Chinese / non-ASCII test material.
3. If a string assertion fails unexpectedly, inspect bytes with `value.encode("utf-8")` or a hex dump, not just `print(value)`.

For high-value non-ASCII payloads, including database writes and MCP payload tests, prefer a UTF-8 file-backed script or JSON payload regardless of shell version, then read back the stored fields. `PYTHONIOENCODING=utf-8` is output-side safety only; it does not repair input that the shell already converted into `?` bytes.

## Python Stdout Encoding

When running Python from a Windows host, inspect `sys.stdout.encoding` instead of assuming the console is UTF-8. Legacy hosts or redirected streams can expose a non-UTF-8 encoding and fail with `UnicodeEncodeError` before useful work begins. PowerShell 7 usually improves this path but does not control every launcher, service, or redirection target.

For one-off commands that may print non-ASCII, set UTF-8 output explicitly:

```powershell
$env:PYTHONIOENCODING='utf-8'; & 'C:\Path\To\python.exe' script.py
```

For recurring Python scripts, prefer setting `PYTHONIOENCODING=utf-8` in the launcher or reconfiguring streams near startup:

```python
import sys

if hasattr(sys.stdout, "reconfigure"):
    sys.stdout.reconfigure(encoding="utf-8", errors="replace")
if hasattr(sys.stderr, "reconfigure"):
    sys.stderr.reconfigure(encoding="utf-8", errors="replace")
```

This is output-side encoding. It is separate from the stdin/here-string issue above.

Do not treat a stdout `UnicodeEncodeError` as proof that an earlier external operation failed. A wrapper can invoke or complete an API call and then fail while printing its Unicode result envelope. Before retrying a paid, non-idempotent, or one-shot operation, distinguish `provider_invoked`, `provider_delivery_verified`, and `stdout_serialized`; preserve unknown delivery as unknown. At an uncertain stdout boundary, prefer a UTF-8 file-backed result or ASCII-safe diagnostic JSON such as `json.dumps(..., ensure_ascii=True)`. Keep that ASCII-safe choice limited to diagnostics; encode the actual API request body according to its transport contract.

## UTF-8 File Edits

PowerShell text reads and mechanical replacements can corrupt Chinese into mojibake, remove or introduce a BOM, or change newlines if the original format is not identified first.

- Prefer `apply_patch` for small edits to files containing Chinese.
- When reading a known UTF-8 file with PowerShell, use `Get-Content -Encoding UTF8`; for an unknown legacy file, inspect bytes or use format-specific tooling before choosing an encoding.
- After mechanical edits touching Chinese literals, scan the changed files for mojibake markers such as `è`, `å`, `æ`, or a leading BOM.
- For dashboard/backend changes served by long-running `pythonw.exe`, restart the local service before browser/API verification.
