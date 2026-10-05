### PowerShell Identity And Codex Route

Identify the shell before reasoning about PowerShell behavior:

```powershell
(Get-Process -Id $PID).Path
$PSVersionTable.PSVersion
$PSVersionTable.PSEdition
$PSHOME
[Console]::OutputEncoding
$OutputEncoding
```

- `pwsh.exe` is PowerShell 7; `powershell.exe` is the built-in Windows PowerShell 5.1. They are designed to coexist. Do not uninstall or disable 5.1 merely because PowerShell 7 is the preferred interactive shell.
- Installing or upgrading PowerShell changes system state and requires user authorization. Query current official package metadata rather than pinning a remembered version or installer variant. Verify the installed executable path, its Microsoft Authenticode signature, and a real launch.
- A running Codex or terminal process does not inherit a PATH update retroactively. After an install or PATH change, restart the relevant app and verify from a fresh process which executable it actually launched.
- Do not infer Codex's shell from `Get-Command pwsh` alone. Read back the current shell process path and version, then run a small native-pipe UTF-8 probe when encoding is part of the goal.

### Native Arguments And Exit Codes

- Invoke a resolved executable with separate arguments (`& $exe @commandArguments`). This avoids evaluating data as PowerShell code; the target still interprets its own options. Use its end-of-options marker only when supported.
- PowerShell 7.3+ improves empty-string and embedded-quote delivery, but Windows mode still uses legacy handling for batch files and selected executables. For fragile JSON or multiline payloads, use the program's file/stdin interface; verify received arguments when exact delivery matters. Do not globally change argument-passing preferences to fix one command.
- `Start-Process -ArgumentList` joins an array into a command-line string; it is not an argv-preserving alternative. Across another shell, prefer a script file with `-File` over nested command strings. Existing Unicode guidance remains in [text encoding](text-encoding.md).
- For cmdlets, use `-ErrorAction Stop` when failure must interrupt work. For native programs, capture `$LASTEXITCODE` immediately and interpret that program's contract; stderr or nonzero alone is not enough. If `$PSNativeCommandUseErrorActionPreference` is enabled, scope its suppression to the block that explicitly checks the native result. Preserve unexpected failures in the final task result.

Sources: [Microsoft parsing](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_parsing), [Start-Process](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/start-process), [preference variables](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_preference_variables).

### PowerShell Automatic-Variable Collisions

PowerShell variable names are case-insensitive. Automatic variables can therefore collide with natural parameter or local names even when casing differs. On the current PowerShell route, the easiest names to borrow accidentally are `$PID` (`Constant`), `$HOME` (`ReadOnly`), `$Error` (`Constant`), and `$Host` (`Constant`). Verify the target shell with `Get-Variable` when the exact reserved set matters.

- When porting, generating, or composing helpers across shells, do not translate variable names mechanically. Audit the target shell's automatic and reserved variables.
- Use role-specific names such as `$TargetProcessId`, `$UserHomePath`, `$CapturedErrors`, or `$ProcessHost`; do not use case variants of automatic variables for parameters, assignments, loop variables, or captures.
- `$PID` is the current PowerShell process ID. `param($Pid)` or `$pid = ...` targets that same variable and fails with `Cannot overwrite variable PID because it is read-only or constant`. Bash exposes its process ID through `$$`, so this collision commonly appears during cross-shell migration.
- A successful PowerShell parser check is not enough: the parser accepts these bindings. For high-consequence helpers, inspect the AST or run an injected no-live fixture that exercises parameter binding and result composition before the real process, service, database, or maintenance window.
- If a script legitimately reads an automatic variable, keep it as an explicit read only. Reject any write or binding site with the same case-insensitive name.
- Treat the exact runtime error above as a shell-semantics defect. Fix and statically revalidate the helper; do not turn a failed verification into a live retry unless the surrounding workflow separately authorizes one.

### Cross-Runtime File Inventory Comparisons

Do not make two runtimes independently sort the same file inventory and then treat sequence equality as identity. PowerShell `Sort-Object` is culture-aware, and `-CaseSensitive` changes case handling without making the comparison ordinal; Python `sorted()` follows different string-order semantics.

- For inventory truth, compare an order-independent mapping or set keyed by the exact stored filename, with exact per-key metadata values. Use sorting only for deterministic diagnostics.
- If one runtime must emit canonical order for another, define the ordering explicitly and have the consumer preserve or independently implement that exact contract. For strings made only of valid Unicode scalar values, Python's default lexical order matches strict UTF-8 byte order. Fail closed on unpaired surrogates instead of silently replacing them.
- In PowerShell, use an explicit strict UTF-8 byte comparator when that cross-runtime contract is required:

```powershell
$utf8 = [System.Text.UTF8Encoding]::new($false, $true)
$utf8Comparer = [System.Comparison[string]]{
    param($leftName, $rightName)
    $leftBytes = $utf8.GetBytes($leftName)
    $rightBytes = $utf8.GetBytes($rightName)
    $sharedLength = [Math]::Min($leftBytes.Length, $rightBytes.Length)
    for ($index = 0; $index -lt $sharedLength; $index++) {
        if ($leftBytes[$index] -ne $rightBytes[$index]) {
            return [int]$leftBytes[$index] - [int]$rightBytes[$index]
        }
    }
    return $leftBytes.Length - $rightBytes.Length
}
[Array]::Sort($names, $utf8Comparer) # $names must be [string[]]
```

- Do not use `Sort-Object -Property @{ Expression = { [byte[]]$utf8.GetBytes($_) } }` as a shortcut. `Sort-Object` does not lexicographically compare those byte arrays and can silently emit the wrong order without an error.
- For a PowerShell-only ordinal contract, use a typed array with `[Array]::Sort($names, [System.StringComparer]::Ordinal)` rather than `Sort-Object -CaseSensitive`. Do not generalize sample agreement with Python into a universal Unicode guarantee.
- Do not silently introduce natural-number ordering: ordinary lexical contracts place `a10` before `a2`. Natural sorting requires its own explicit shared rule.
- Before a high-consequence cross-runtime comparison, run an injected fixture proving that identical membership in different insertion/display order passes while missing, extra, or changed values fail.
