---
name: windows-ops
description: Prevent and repair Windows runtime, launcher, service, shell, and text-encoding failures. Use when non-ASCII paths, Python runtimes, process ownership, or startup context can change an operation or failure, including dependency and post-update validator changes. Not for platform-neutral work.
license: "CC-BY-4.0 (prose); MIT (code and configuration). See LICENSING.md."
---

# Windows Ops

Public Windows operations guidance by Luluane & Astrean (Starflame).
Source: https://github.com/LuluaneS/windows-ops
License and reuse: [LICENSING.md](LICENSING.md).

## Proactive Windows Gate

Load this skill before the first command when a Windows-hosted action will read or write Chinese, emoji, or non-ASCII paths; pass non-ASCII through Python or PowerShell stdin/stdout; rely on implicit text-file encoding; or depend on an exact shell, executable, process host, user session, service, scheduler, or hidden-child context.

For a maintained tool whose contract already requires known UTF-8 input, fix the owning read/write call with explicit UTF-8 and verify the ordinary invocation. Use `python -X utf8` only for a genuinely one-off UTF-8-only tool or bounded diagnosis; do not let it mask a regression in maintained code.

## Conditional cards

Read only the matching card before that action; these are parts of this skill, not new owners.

- After a Codex update/reinstall, at the next relevant skill validation: [validator canary](references/validator-canary.md).

Use the user's existing authorization for repairs. Reading this skill does not
authorize installation, process termination, service restarts, or persistent
startup changes. Complete useful read-only diagnosis before requesting any
missing authorization.

## Runtime

Verify the exact executable path in the target process context. Do not assume `python`, `py`, `pythonw`, `pwsh`, or `powershell` works because another terminal or user session worked.

Treat command resolution as discovery, not execution proof. WindowsApps aliases and placeholders can open Microsoft Store, resolve differently for a service account, or fail silently. For launchers, services, scheduled tasks, and MCP config, prefer a verified real executable path and launch that exact path once.

- Before Python dependency/runtime changes, service activation, process ancestry or termination: [runtime ownership and activation](references/runtime-ownership.md). It owns per-consumer STAGED/LIVE/UNVERIFIED and process incarnation checks.

- When shell identity, parameter binding, reserved variables or cross-runtime inventory comparison matters: [PowerShell semantics](references/powershell-semantics.md). Use role-specific variables; casing does not avoid automatic-variable collisions.

- Before launchers, shortcuts, hidden children, pythonw or persistence work: [launchers and process presentation](references/launchers.md). Hidden windows do not prove independent lifetime; Startup holds launchers only.

- Before encoding-sensitive edits, stdin/stdout handling or uncertain Unicode delivery: [text encoding](references/text-encoding.md). Preserve identified encoding/newlines; prefer small patches and UTF-8 file-backed non-ASCII payloads. Output failure does not prove an earlier external operation failed.

## Verify

Separate "file exists", "runs in Codex sandbox", "runs in normal PowerShell", and "runs as service/scheduled task with its own account and working directory".
