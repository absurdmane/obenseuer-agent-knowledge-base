# Save files and mod-side data

Scope: Save folder layout, how saves are copied, reading a save file, plus a mod-guidance section on storing per-save mod data in your own file.
Status: code-only; re-checked against build 0.5.13 (see the last section)
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

## Save and load order, hooks, autosave `[code]`

- `SavableScript` (MonoBehaviour base): virtual hooks returning bool (default true; the driver ignores the value), plus nested `SavableObjectReference` (`GUID`, resolved by `SaveController.GetObjectByGUID`). Serializer is FullSerializer; private fields are saved only with `[fsProperty]`; `[fsIgnore]` is on about 2,200 fields in 279 files. Typical pattern: `OnSavingGame` calls `SaveController.SerializeData(this)`, `OnLoadingGame` calls `DeSerializeData(this)`.
- `SaveController.SaveGame` order: `OnMapChanging` (level change only), `OnSavingGamePrimary`, `Secondary`, `Tertiary`, `DestructibleList.OnSavingGameDestructibleList`, `Special` (DontDestroyOnLoad objects), `OnSavingGame`, `OnSavingGameLatePrimary`. Then it writes `Info.tnmt`, `Globals.tnmt`, `<Level>.tnmt` and `Globals.dialogue`, calls `OnSavingFile` and raises `SavingDone`.
- Load order after `LoadSceneAsync`: `LoadingStarted`, `OnLoadingGamePrimary`, `Secondary`, `Tertiary`, `Special`, destructible-list pass (objects whose `GUID` is in `DestructibleList.DestroyedGuids` are destroyed, so a spawned object with a `GUID` field can vanish on load), one frame, `OnLoadingGame`, `OnLoadingGameLatePrimary`, destructible check again, `OnMapChanged` (only on level change or an entrypoint), player moved to the entrypoint, `Globals.dialogue` applied, `LoadingDone`. Code that needs other objects' loaded state belongs in `LatePrimary` or a coroutine that waits a frame.
- `ExecuteChildSaveLoadFunctions` calls hooks only on objects active in the hierarchy (state of objects disabled at save time is lost) and wraps each call in try/catch: a failing hook is logged and skipped, the save or load goes on.
- "LoadDelay" is not shared: classes such as `CraftingBase`, `Bank`, `BaseShop`, `AnimalStats` each have their own coroutine: wait a frame, wait while `TimeOfDayAzure.updateTimeDisabled`, wait for end of frame twice, then apply elapsed time from `savedTimeAndDay`.
- Autosave: `SaveController.Autosave` starts `AutosaveDelay` (SavingUI, about 3 frames, `SaveGame(GetOldestAutosave())`). Slots `Autosave` and `Autosave2`; the one with the older `Date` in `Info.tnmt` is overwritten, also on `ChangeLevel`. Triggers: `TriggerAutosave`, `logic_autosave`. `AutosaveMode` (Default, EveryDay, EveryHour) is stored in `DifficultyInfo.autosaveMode`, but no reader that triggers timed autosaves was found.
- Versioning: `SaveDataHeader.ApplicationVersion` is `Application.version`, shown with a warning in `LoadMenu` when it differs. `MigrationController.RunMigrations` and its `migrations` list are empty in this build; `LoadGameWithMigration` runs the `Migrate` coroutine only if the `Globals.tnmt` header fails to deserialize.

## Changes in 0.5.13 `[code]`

- Copy: `SaveController.SaveGame` still copies the loaded save folder into the new one, but `CopyDirectory` now skips the four files that are rewritten right after (`Info.tnmt`, `Globals.tnmt`, `<LevelName>.tnmt`, `Globals.dialogue`; root level only, subfolders in full) and copies in parallel (up to 8 threads).
- Reading: `SaveController.Deserialize` first calls `FsStreamReader.TryDeserialize`, a fast reader that parses the JSON into a node array and does the collapse itself (`FsStreamReader.GetCollapse`, `IsCollapsible`, `SavedAssets.GetAsset`, `SavedAssets.MatchesType`); on `UnsupportedException` or any failure it logs a warning and falls back to the `CollapseGraphs` path above, so the rules below may not match the fast path exactly. `FsStreamWriter` is the matching fast writer (a UTF-8 BOM, the same key names `$type`, `$id`, `$ref`, `$content`); on failure `SerializeSaveFile` falls back to the old `Serialize` string. `SaveController.JSON_serializer` is now `internal`; `HasSaveData(guid)` and `GetOldSubGUIDs(originalGUID, iter)` are new public statics.
- `ReplaceWithSavedata` now sets the consumed entry in `tempSavedata_Global`/`_Level` to null instead of removing it: code that walks those lists must skip nulls.
- Sub-object GUIDs changed (`InstantiatedPrefabRef.AssignSubGUIDs`): `<originalGUID>_<iter>_<TypeName>_<index per type>` (was `<originalGUID>_<iter>_<n>` with one counter). Components count only with a public string field `GUID`. On load (`SetSubGUIDs`) an active object without save data under its new id takes the next old-format id from `GetOldSubGUIDs`, so old saves are remapped; code that parses or builds sub-GUIDs must accept both forms.
- Order and waits: `SaveGame`, `RemoveCharacter` and `LoadGameWithMigration` first call `RevealCamera.WaitForPendingMapWrite()` (the map PNG is now written on a background `Task` in `RevealCamera.OnSavingGame`; whether it lands in the folder copied by the save that starts it is unverified); `LoadGameWithMigration` logs "Save files read" with the elapsed time. The hook order is unchanged.
- `Achievements` skips `SendRewards` once `rewardsSent` is set. `output.onEventInvoke` (scene relays) catches exceptions from `OnTrigger` per target, so a throwing handler no longer stops the chain.
