# Reading game asset data offline (UnityPy)

Scope: how to read prefab and scene data (Storage settings, lights, ScriptableObject fields) from the game files without running the game: Addressables bundles and player data files, with UnityPy. Not covered: game logic (see the other notes).
Status: partly verified (the scripts ran against the game files of the 15 Oct 2026 build, 2026-10-19); facts read this way are tagged `[data]` in the notes.
Related: [power.md](power.md), [items-recipes.md](items-recipes.md), [INDEX.md](INDEX.md)

**The helper scripts named below are not part of this repository.** They were small helpers around UnityPy; their behaviour is described here so you can recreate them. The facts that came out of them are in the other notes, tagged `[data]`.

## Setup

- Use a Python version that has prebuilt wheels for UnityPy's dependencies (at the time of writing 3.12 worked; very new Python releases did not) in a virtual environment, then `pip install UnityPy==1.25.4 TypeTreeGeneratorAPI`.
- Everything is read-only; nothing is written into the game folder. Load the files with UnityPy, never modify them.

## What the tools read

- `bundles.py` — `Bundles()`: loads the Addressables bundles (`Obenseuer_Data/StreamingAssets/aa/StandaloneWindows64/*.bundle`, ~760 MB, loading takes a few minutes and a few GB of RAM; localization bundles skipped). `Bundles(files=[<full paths in Obenseuer_Data>])` loads player data files (`level*`, `sharedassets*.assets`).
  Helpers: `of_type`, `monobehaviours(class_name)` (resolves the script class), `script_name`, `typetree(obj)`, `go_path(component)`.
- `fridges.py [name-substring ...]` — every `Storage` with `StorageSettings`, `refigerationFactor`, title.
- `lights.py out.txt [--data file ...] [root-substring ...]` — every `LightController`: upgrade root, path, world position, `Light` type/range/colour/intensity, `electricLight`, nearest mesh renderer and nearest lamp-like one. `summarize_lights.py` groups it; `bare_lights_report.py` classifies bare lights into fixture/near/fill.


- `electronics.py <furniture|upgrades-aa|upgrades-data|scene> out.json [file]` + `electronics_report.py` — inventory of power-relevant components per Furniture asset / upgrade device / scene object; the scene mode over `level4` (the tenement interior) takes ~10 minutes.
- `find_names.py out.txt REGEX [--data file ...]` — asset names (GameObject, Material, Mesh, Texture2D, animator, clip, sprite, audio) matching a regex. `find_class.py CLASS [--data file ...]` — where instances of a script class live and their typetree keys.
- `growing_scan.py out.txt [levelN ...]` — `Growing` components (AlwaysOn, power type) per scene file; over all 124 levels it is very heavy (up to ~12 GB RAM), run it on chosen levels only.
- `dialogue_export.py OUT_DIR` — dumps the PixelCrushers `DialogueDatabase` (one instance, in the main Addressables bundle: 312 actors, 161 variables, 487 conversations; entries have `Dialogue Text`, `Menu Text`, `conditionsString`, `userScript`, `outgoingLinks`) and the English Unity Localization tables (`Items_en`, `Notes_en`, `Tasks_en`, `UseText_en`, `ToolTip_en`, `Menu_en`, ...) as plain text. Takes a few minutes.

## Where the data lives

- Furniture prefabs (`Refrigerator`, `Chest Freezer`, ...) and NPC apartment upgrade prefabs (`Upgrade_NPC_*`): Addressables `defaultlocalgroup_assets_all_*.bundle` (typetrees included). MonoScripts: the `monoscripts` bundle.
- The tenement upgrade prefabs (`Upgrade_Basement_*`, `Upgrade_PLAYER_*`, `Upgrade_STORE_*`, `Elevator_*`, Greenhouse): `Obenseuer_Data/sharedassets4.assets`. The scene `level2` holds scene objects (`_Environment`, NPCs, yard). Find a name with `grep -l -a "<GameObject name>" level* sharedassets*` in `Obenseuer_Data`.
- Scene-only objects (the worn backpack storages `Game_Logic/Backpack Storage/...`, the live tenement) are not in the bundles: use a runtime dump (a dump plugin or logs) for those.

## Pitfalls

- Player data files (`level*`, `sharedassets*`) carry **no type trees**. `TypeTreeGeneratorAPI` generates them from `Managed/Assembly-CSharp.dll`; version 0.0.10 omits the align flag on `MonoBehaviour.m_Enabled`, which garbles every later field (`m_Script` becomes nonsense). `bundles.py` patches it (`FixedGen`). Re-check after a UnityPy / generator update.
- MonoScripts for the game's classes live in `globalgamemanagers.assets` (loaded automatically with player files); script names come from `MonoScript.m_ClassName`.
- Unity 6 `GameObject` typetree key is `m_Component` (list of `{component: PPtr}`), not `m_Components`.
- Prefab positions are local to the prefab root; `lights.py` composes transforms up to the nearest ancestor named `Upgrade_*`, so positions are comparable inside one upgrade, not world coordinates of the scene.
- The Addressables catalog (`catalog.bin`) is not parsed; use object paths inside the bundles instead.
