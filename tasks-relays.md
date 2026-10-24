# Tasks, dialogue bridge and scene logic (relays)
Scope: how game code talks to the dialogue system (Lua functions, variables), the task/objective system, recipe learning, notes, and the `Relay`/`logic_*` glue used by scenes. Mechanics only; dialogue content and story are not covered.
Status: code-only
Related: [crafting.md](crafting.md), [items-recipes.md](items-recipes.md), [saves.md](saves.md), [doors-interaction.md](doors-interaction.md)

## Where it lives

- Lua bridge: classes call `Lua.RegisterFunction(name, this, method)` and `Lua.UnregisterFunction(name)` (PixelCrushers Dialogue System); about 130 registrations in 30 files (`TaskController`, `RecipeController`, `TaskItemsManager`, `RelayDialogue`, `DialogueCommonMethods`, `Inventory`, `TenementController`, `Crime`, `Bank`, `Money`, `Act_Police`, `Act_Robber`, `Market`, ...). Variables: `DialogueVariable` (wrapper using `DialogueLua.GetVariable/SetVariable` and actor fields), `RelayDialogueVariable`, `ChangeDialogueVariableValue`, `PlayerIdentity` (writes Player actor fields). [code]
- Tasks: `TaskController` (SavableScript, `instance`), `TaskInfo`, `TaskObjective`, `TaskStatusInfo`, enums `CurrentTaskStatus` (Active, Failed, Finished, Abandoned, AnyStatus) and `ObjectiveStatus` (Active, Failed, Finished, Abandoned, Hidden, AnyStatus, NotActive); UI `TaskPanel`/`TaskPanelUI`. Task items: `TaskItem` (ScriptableObject, also used for recipes and furniture in the inventory), `TaskItemsManager`. Recipes: `RecipeController`. Notes: `NotesPanel`, `NoteController`, `ItemWritableData`. [code]
- Scene glue: `Relay` (SavableScript), `RelayDialogue`, about 30 `logic_*` classes, and Behavior Designer tasks for the dialogue system (start/stop conversation, quest state, run Lua). [code]

## How it works [code]

- Lua functions are registered in `Awake` (or a guarded `RegisterDialogue`) and removed in `OnDestroy`. A Lua name is global: two components registering the same name overwrite each other, and unregistering removes whichever one is registered.
- `TaskController` read functions (string ids): `TaskActive`, `TaskAnyStatus`, `TaskFinished`, `TaskFailed`, `TaskAbandoned`, `ObjectiveAnyStatus`, `ObjectiveActive`, `ObjectiveAbandoned`, `ObjectiveFailed`, `ObjectiveCompleted`; one write function, `SetObjectiveFinished(taskId, objectiveId)`.
- State lives in `allTasks` (`TaskStatusInfo` list) plus id lists, saved with `SaveController.SerializeData(this, global: true)`. `TaskController.EditTask` is the central write path: it checks activate requirements, creates the task if asked (shows a "New Task" notification, adds it to `TaskPanel`), then sets objective and task status. It returns `false` quietly when the task is already active or requirements are not met. `TaskInfo.finishTaskWhenTheresNoActiveTasks` finishes a task once no objective is Active. `TaskObjective` has follow-up objective lists (`onFinishedObjectives`, `onAbandonedObjectives`, `onFailedObjectives`), requirement lists and a counter.
- Recipes: `RecipeController.AddRecipe` (not learned) creates a runtime `TaskItem` with id `Recipe_<ID>` and adds it via `TaskItemsManager.AddTaskItem`; learned recipes go into `learnedRecipes` only; `GiveRecipe(double)` is the Lua entry. `TaskItem.Compare` prefers `id`, then `addressableAssetPath`, then `itemName`. On load, recipes are re-fetched from `RecipeDatabase` after a frame.
- `Relay.triggerOutputs`: returns silently if the object is inactive, if `fireOnceOnly` and already fired, or if any `requiredTaskItems`/`requiredObjectives`/`requiredDialogueVariables` is not met; otherwise it sets `firedOnce` and fires each output with its own delay (coroutine). Delayed outputs survive save and load (`delayLeft`). Also `enableOutput`, `disableOutput`, `toggleOutput` (index -1 = all), `CancelPendingDelayedOutputs`.
- `RelayDialogue` registers `Relay_<relayName>` as a Lua function that fires its outputs, so dialogue scripts can trigger scene relays by name. `RelayDialogueVariable` compares a variable or actor field with an expected value at start and fires its outputs.
- `logic_branch` (`isTrue`, saved; the setter fires `onTrue`/`onFalse` unless `manualTest`, then notifies `logic_branch_listener`), `logic_state` (state machine with a relay per state), and helpers `logic_taskitem_control`, `logic_wait`, `logic_timer`, `logic_gauge`, `logic_mail_*`, `logic_invoice_sender`, `logic_receipt_maker`, `logic_crime`, `logic_storage_listener`, `logic_achievement_trigger`, `logic_autosave`.
- Notes: `NotesPanel.UpdateInventoryNotes` lists inventory stacks with `itemId` 17340 that carry `ItemWritableData` (title, content, read/edit flags); notes live in the items, not in the panel.

## Pitfalls

- A mod or hot-reloaded component that registers Lua functions must unregister them on destroy.
- `TaskController.OnLoadingGame` uses `TaskPanel.instance` directly (throws if the panel is missing); `RecipeController.OnLoadingGame` assumes `TaskItemsManager.instance` exists.

## Unverified

Whether 17340 is the only notebook item; whether task and objective ids used in dialogue match the asset data.
