### Codex Bundled-Validator Update Canary

After a Codex update, reinstall, or repair that may replace bundled system skills, use the first Windows skill-validation task on that build as a behavior canary. Run the live bundled `skill-creator` `quick_validate.py` through its ordinary invocation, without `python -X utf8`, `PYTHONUTF8`, a wrapper, or an auto-patcher, against a known UTF-8 skill containing CJK text.

- A clean validation proves the ordinary CJK read path is intact for that build.
- If the canary raises a locale-codec `UnicodeDecodeError`, inspect the live bundled validator before changing it. When the failure is again an unqualified `Path.read_text()` reading `SKILL.md`, restore only `encoding="utf-8"` on that owning read and rerun the same ordinary canary.

Make that repair only within existing authorization to modify the validator;
otherwise report the exact failing read and propose the narrow fix. Do not infer
that another CLI, IDE extension, or agent host shares the same validator path or
update mechanism. Inspect that environment's effective validator first.

This is a post-update check at the next relevant technical task, not a background monitor, file-hash alarm, wrapper, or standing patch system.
