# Windows Ops

A practical agent skill for Windows runtime, launcher, service, PowerShell, and
text-encoding problems. Created by **Luluane & Astrean (Starflame)** from ongoing
Windows operations work.

The skill helps an agent distinguish a file or configuration change from a
repair that actually works in the affected process. It contains instructions
and small examples, with no installer, resident service, telemetry, or bundled
third-party runtime.

## What it covers

- Python environment ownership, dependency changes, and service relocation.
- The difference between a staged runtime change and a live, verified consumer.
- PowerShell identity, automatic-variable collisions, and file-inventory comparisons.
- Hidden child processes, shortcuts, single-instance services, and independent lifetime.
- UTF-8 files, native pipes, `pythonw`, and uncertain delivery after an output error.
- A conditional Codex skill-validator check after updates.

`SKILL.md` is a short entry point. The agent reads only the relevant card in
`references/`, instead of loading every procedure for every task.

## Use with Codex

Clone into a new `windows-ops` directory in your user skills folder:

```powershell
git clone https://github.com/LuluaneS/windows-ops.git "$HOME/.agents/skills/windows-ops"
```

If that destination already exists, inspect it before updating; do not overwrite
another local skill. Keep the whole folder, including `references/` and
`agents/openai.yaml`.

Invoke it with a concrete request, for example:

> Use $windows-ops to find why this Python service works in my terminal but fails
> when Task Scheduler launches it. Diagnose first, then explain the repair.

For project-local installation, place the folder at
`.agents/skills/windows-ops/` in that project. Other agents that support
`SKILL.md` can use the instructions with their own discovery mechanism;
`agents/openai.yaml` is Codex-specific metadata. See the
[official Codex skill documentation](https://developers.openai.com/zh-Hant/docs/build-skills)
for discovery and refresh behavior.

## Operating boundaries

The skill follows the user's actual task and authorization. It does not grant
permission to install software, kill processes, restart services, change
startup configuration, or edit bundled tools. General Windows guidance applies
across hosts; the bundled-validator card applies only when that Codex tool is
present and the documented failure is observed.

Verification is scoped: running in a terminal does not prove a scheduled task
works, and a healthy hidden service does not prove it survives its launcher.
Environment-specific observations must be checked on the affected machine.

## September 2026 update

The public edition now uses focused reference cards and includes service-owned
Python environments, rebuilding virtual environments after relocation,
packaged-app path visibility, and checks for launcher lifetime and cancellation.
Existing encoding, shell, process-incarnation, and validator guidance is retained.

## Attribution and provenance

Maintained and published by **Luluane** ([LuluaneS](https://github.com/LuluaneS)).
Developed together with **Astrean / Starflame**, with this public edition prepared
in Codex. Original project: [LuluaneS/windows-ops](https://github.com/LuluaneS/windows-ops).

The public edition adapts our maintained Windows operations skill into a
self-contained package. Private paths, household service names, and internal
dependencies are not part of this distribution. Referenced third-party tools
and documentation retain their own terms.

## License and reuse

The guide and documentation use **CC BY 4.0**. Code examples and configuration
use **MIT**. See [LICENSING.md](LICENSING.md) for the exact scope and notices,
[LICENSE](LICENSE) for the documentation terms, and [LICENSE-CODE](LICENSE-CODE)
for the code terms. This is a scope split, not a choice of either license for
the same material.

You may use, modify, redistribute, and use the material commercially. When
sharing an adapted guide, credit **Luluane & Astrean (Starflame)**, retain the
source and license information, and indicate changes. For copies or substantial
portions of the code, retain the copyright notice (including the project credit
and source) and MIT permission notice. Neither license requires publishing your
modified source code or using the same license for your own contributions,
provided the original material's terms and notices remain respected.

An attribution example for an adapted guide:

> Based on Windows Ops by Luluane & Astrean (Starflame),
> https://github.com/LuluaneS/windows-ops. Original guide licensed under
> CC BY 4.0: https://creativecommons.org/licenses/by/4.0/.
> Changes: [describe your changes]. Provided without warranties under that license.

The example helps preserve the origin; the full license terms govern reuse.
