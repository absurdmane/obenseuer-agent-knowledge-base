# Save files and mod-side data

Scope: Save folder layout, how saves are copied, reading a save file, plus a mod-guidance section on storing per-save mod data in your own file.
Status: code-only
Related: [levels.md](levels.md), [tenement-upgrades.md](tenement-upgrades.md)

## Save folder

- A save is a **folder**: `Application.persistentDataPath/Saves/<CharacterName>/<SaveName>/` with `Info.tnmt`, `Globals.tnmt`,
  `<LevelName>.tnmt` (one per visited level), `Globals.dialogue`.
- `SaveController.SaveGame(savename, ...)`: if the target is a different folder, it deletes it, then **copies the whole folder of the
  loaded save** (`LastSaveFolderPath`) into it, then writes its files. So any extra file a mod puts in a save folder travels with
  "save as" and autosaves, and is deleted with the save.
- `SaveController.LastSaveFolderPath` — folder of the save currently loaded (set in `LoadGameWithMigration`).

## Mod guidance: per-save mod data

Write your own file without touching game data: write it (e.g. `MyMod.json`) into the save folder in a
postfix of `SaveController.SaveGame` (folder = the path built from `CharacterName` + `SaveName`), read it from `LastSaveFolderPath` on load.
Missing file = defaults; removing the mod leaves a harmless orphan file.

## Reading a save file: SavedAssets.CollapseGraphs (0.5.08) [code]

`SaveController.Deserialize(fileName, type, json, out exception, out skippedFieldsOnly)` (the 4-parameter overload forwards
to it) parses the JSON, then runs `SavedAssets.CollapseGraphs` on the fsData tree, then `TryDeserialize`. If the collapse
throws, it re-parses and deserializes the uncollapsed tree.

- Asset types: TaskInfo, TaskObjective, NoteInfo, BookInfo, Furniture, TenementUpgrade(Group), TenementContractor, Landmark.
  `SavedAssetConverter` handles them: data with an `addressableAssetPath` of a loadable asset of the storage type becomes
  that asset (the saved contents are ignored).
- "Collapsible" object: has a string `addressableAssetPath`, `SavedAssets.GetAsset(path)` finds an asset, and at least half of
  its non-`$` keys are fsMetaType members of the asset's type. It is replaced by `{ $type, addressableAssetPath }`; its `$id`
  and all `$id`s inside are dropped ("pruned").
- `$ref` (an object with only `$ref`) to a collapsible definition → the same stub. To a pruned definition → a deep copy of
  it (collapsible objects inside stubbed), unless that `$id` is already being copied higher up (cycle) → stays `$ref`.
- Definitions are indexed with a stack (first occurrence in that order wins); the root is never replaced.
- Nothing collapsible in the file → the tree is untouched.
- In one test save 11 of 44 files contain `addressableAssetPath` (Globals, Mines, Player Tenement, ...).
