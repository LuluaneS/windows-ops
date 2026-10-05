---
name: windows-ops
description: Prevent and repair Windows runtime, launcher, service, shell, and text-encoding failures. Use for PowerShell native arguments, exit codes, configuration round-trips or file batches, and when non-ASCII paths, Python runtimes, process ownership or startup context matters, including dependency and post-update validator changes. Not for platform-neutral work.
license: "CC-BY-4.0 (prose); MIT (code and configuration). See LICENSING.md."
---

# Windows Ops

Public Windows operations guidance by Luluane & Astrean (Starflame).
Source: https://github.com/LuluaneS/windows-ops
License and reuse: [LICENSING.md](LICENSING.md).

## Before the first command

Load this skill when a Windows action handles Chinese, emoji or non-ASCII paths;
passes non-ASCII through Python or PowerShell stdin/stdout; relies on implicit
file encoding; or depends on the shell, executable, process host, user session,
service, scheduler or hidden-child context.

Verify the executable in the target context. Command resolution alone is not
execution proof: WindowsApps aliases may open the Store or fail in another
session. For launchers, services, scheduled tasks and MCP config, use a verified
real executable path and launch that exact path once.

For maintained tools with a UTF-8 contract, fix the owning read/write call and
verify the ordinary invocation. Reserve `python -X utf8` for one-off UTF-8-only
tools or bounded diagnosis, rather than masking a maintained-code regression.

Use the user's existing authorization for repairs. This guide does not grant
permission to install software, stop processes, restart services or change
startup configuration. Complete useful read-only diagnosis before requesting
any missing authorization.

## Read the relevant card

Choose by the task or symptom; read only the matching card before acting.

| Task or symptom | Card |
|---|---|
| Shell identity, native arguments/exit codes, parameter binding, reserved variables, cross-runtime inventory comparisons | [PowerShell semantics](references/powershell-semantics.md) |
| Encoding-sensitive edits, stdin/stdout or uncertain Unicode delivery | [Text encoding](references/text-encoding.md) |
| PowerShell configuration round-trips or bulk rename/move | [Configuration and file batches](references/config-and-file-batches.md) |
| Launchers, shortcuts, hidden children, pythonw or persistence | [Launchers and process presentation](references/launchers.md) |
| Python dependencies/runtimes, service activation, process ancestry or termination | [Runtime ownership and activation](references/runtime-ownership.md) |
| First relevant validation after a Codex update/reinstall | [Validator canary](references/validator-canary.md) |

## Verify in the target context

Distinguish file existence, execution in the Codex sandbox, execution in normal
PowerShell, and execution through the actual service or scheduled task with its
own account and working directory. Success in one context does not prove another.
