# Configuration Round-Trips And File Batches

Use for PowerShell configuration edits or bulk file operations. Encoding and
newline preservation remain in [text encoding](text-encoding.md).

## Keep configuration meaning intact

- Choose a parser/editor for the actual format, including JSONC comments. Change only requested fields; a successful parse does not prove unrelated data survived.
- `PSCustomObject` cannot faithfully represent JSON keys differing only by case or an empty key. Use `ConvertFrom-Json -AsHashtable` where available, or a suitable existing parser. For exact timestamp strings, use `-DateKind String` when supported (7.5+) or a parser that preserves strings; do not upgrade the runtime for one edit.
- Preserve top-level empty/singleton arrays on both read and write: use `ConvertFrom-Json -NoEnumerate` when supported and `ConvertTo-Json -InputObject $value`. Choose `-Depth` for the actual document. Treat depth warnings as failed serialization; older engines may truncate without a warning, so verify nested values as well as parseability.
- For whole-file rewrites, stage beside the target when practical, parse back and compare the intended change plus unrelated values/types before replacement. If another app can write the file, detect intervening changes before replacing it. Use the host's required editing tool where applicable.

## Keep file batches predictable

- Use absolute paths for .NET file APIs: their process directory may differ from PowerShell's location. Resolve an existing parent for a new target; use `-LiteralPath` with PowerShell file operations when names can contain wildcard characters.
- Parse CSV with a CSV reader and explicit delimiter/encoding; preserve column order. Keep values as data rather than shell fragments.
- Before bulk rename/move, calculate the complete source-to-destination mapping. Check missing sources, duplicate/existing destinations using the target filesystem's case rules, and destinations that are another input. Use distinct temporary names when cycles or case-only renames require them; preserve the mapping for recovery from partial completion.
- Verify resulting names/counts against the mapping. A rerun must recognize completed work rather than overwrite an unrelated destination.

Sources: [Microsoft ConvertFrom-Json](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertfrom-json), [ConvertTo-Json](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertto-json).
