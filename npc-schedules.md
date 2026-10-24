# NPC data, schedules, residents, relationships
Scope: how NPCs are defined, how their daily schedules run, tenants, relationships, `CharacterSpawner`. Police and robbers are in police-crime.md and combat-npcs.md; NPC movement, inter-scene walking and `NPCManager` are not covered.
Status: code-only (record counts are `[data]`)
Related: [police-crime.md](police-crime.md), [combat-npcs.md](combat-npcs.md), [time-and-weather.md](time-and-weather.md), [tenement-upgrades.md](tenement-upgrades.md), [economy.md](economy.md)

## Where it lives

- Data files in `StreamingAssets`: `NPCBehavior.json` (243 NPC records), `Characters.json` (40 records), `CharactersRent.json` (rent terms). [data]
- Namespace `NPC`: `NPCData`, `NPCDataState`, `NPCDirector` (`globalDatabase`), `Schedule`, `ScheduledAction`, `Activity` and the `Act_*` classes, `NPCController`, `NPCReference`, `NPCSceneUtilities`. Top level: `Character` (ScriptableObject), `CharacterDatabase`, `CharacterRent`/`CharacterRentDatabase`, `CharacterSpawner`, `DoorSchedule`, `Timetable` (a BehaviorDesigner conditional), `Relationship`/`RelationshipController`, `TenementResident`, `TenementController`. [code]

## How it works [code]

- `Characters.json` is not read by code (no reference found). Runtime characters are ScriptableObject assets in Resources (`CharacterDatabase.LoadCharactersFromResources`, `FetchCharacterByID`); the JSON looks like an export of the `Character` fields. `Character.Relationship` is a plain string (marital status), unrelated to the `Relationship` system.
- `NPCBehavior.json` is loaded by `NPCDirector.LoadDatabase`. Each record: `ID`, `startScene`, `startPosition`, `schedulerDisabledAtStart`, `scheduler {Name, Timetable[], RandomActivities[]}`, `prefabPath`; activities are polymorphic (`$type`): 144 entries, 49 `Act_Police`, 95 others [data]. `NPCData.state` is `[fsIgnore]` and rebuilt from data on load; saved copies go through `NPCDirector.savedData`.
- Clock driver: `NPCDirector.DeltaSeconds` is subscribed to `TimeOfDayAzure.MinutePassed`; per tick it advances transit for NPCs outside the active scene (`NPCSceneUtilities.UpdateNPCTransition`), makes followers follow, runs `Schedule.UpdateScheduler` and `HandleNPCChangeScene`. It catches and logs all exceptions, so a broken NPC silently skips its tick.
- `Schedule`: for NPCs in the active scene `UpdateScheduler` only calls `Schedule.CurrentTime`; for others it just sets `currentScheduledAction`. `CurrentTime` skips while `activityDisabled` or before `nextCheck`, never interrupts an activity whose `CanBeInterrupted()` is false, then starts the first `Timetable` entry whose window contains now (windows across the week end, 10080 minutes, are handled) after a `chance` roll; otherwise it may pick one `RandomActivities` entry. `ScheduledAction`: `name`, `disabled`, `interruptPrevious` (default true), `chance` (default 1), `starts`, `duration`, `activity`.
- `Activity` base: `StartActivity`, `UpdateActivity`, `InterruptActivity`, `CanBeInterrupted`, `GetLocationInfo`, `GetTargetScene`, `OnStuck`, `FinishActivity`/`StopActivity`. Non-police types: `Act_Stand`, `Act_sit`, `Act_Sleep`, `Act_Wander`, `Act_Patrol`, `Act_Follow`, `Act_Tenement`, `Act_Robber`. Counts in data: stand and sit are the most common, then tenement, sleep, patrol. `Act_Sleep` finds a bed (`FindBed`), walks there and sleeps.
- Residents: `Character.resident` links a character to an apartment rental; `TenementController.currentSceneResidents` maps `TenementResident` to the spawned `NPCInfo`. `Act_Tenement` walks to `TenementController.GetGetFirstAvailableSpotForResident(npcID)`, which returns a `Node`, an `InteractableChair` or a bed; its branch for a bed only logs and assigns nothing.
- `CharacterRent` (from `CharactersRent.json`): `Income1..6`, `FloorMin/Max`, `Electricity`, `Water`, `Kitchen`, `MinHeating`, `MinBathroom`, `Smallest/Biggest`, `Method`, `MinCondition/MaxCondition`, `Family1..4`.
- Relationships: `Relationship` (ScriptableObject): `startDisposition`, `dispositionChangeSpeed` (per day), `eventProbabilityPerDay`, events (only `Gift` in code, sends mail). `RelationshipController.relationships` holds `RelationshipInfo` (`active`, `disposition`, `dispositionTarget`, `lastEvent`), saved. `CheckActiveRelationships` runs once per game day: disposition drifts toward the target and one weighted event may fire. The check looks skipped on weekday 6 after a one-day gap (intent unknown).
- `CharacterSpawner` (a `Spawner`) picks random unused characters and spawn points, skips a character whose object name already exists in `CharacterManager.Instance.CharactersInScene`, and `Instantiate`s under the spawner (not saved).
- `NPCReference` equality compares only `ID` (the typed `Equals` overload does not override `object.Equals`); its int constructor reads `NPCDirector.instance` immediately.

## Mod guidance

Change a routine by patching `Schedule.CurrentTime` (or the active-scene path `Schedule.UpdateScheduler`). Prefer patching getters over changing saved types (`NPCData`, `NPCDataState`, `RelationshipInfo`). Test schedule changes with entries that cross the week boundary.

## Unverified

Where `Schedule.CurrentTime` runs for NPCs outside the active scene; meaning of `Income1..6` and `Family1..4`; whether any behaviour tree uses `Timetable`; in-game behaviour of any of this.
