# Obenseuer knowledge — index

Open this file, then **one** note. Each note starts with Title / Scope / Status / Related.

**Version:** the notes were first written against game 0.5.08 (`Assembly-CSharp.dll` of 15 Oct 2026). On 2026-10-25 they were re-checked against build 0.5.13 (Steam build 26179069, DLL of 24 Oct 2026) by a method-level comparison of the two builds: every note that cites a changed `Class.Member` was corrected, and new systems got notes (power-grid.md, quick-slots.md). Notes the comparison did not flag are unchanged at code level; asset values (scenes, prefabs) were not re-read. A note gets an extra line only if it was re-checked against a different build.

**Status values:** `code-only` (read from decompiled code), `partly verified` (some items marked `[verified]` = seen in game), `unverified source` (not from code). Inline tags: `[code]`, `[verified]`, `[data]` (JSON/asset).

**Where findings go**
- A subsystem fact goes into that subsystem's note, with `Class.Member` references (no line numbers). A cross-system pitfall goes into the note of the class that causes it, with a Related link from the other note.
- A new note only when the topic has no home. Split a note when it passes ~120 lines or covers two subsystems; leave a pointer line in the old place.
- Mod-specific decisions belong in the mod's own docs, not here. Mod-guidance sections (how to hook the game) are allowed if titled "Mod guidance".
- Things only visible at runtime (prefab/ScriptableObject values): say so in "Runtime data" and capture them with a runtime dump (a small plugin or logs).

## Player and inventory

| Note | Covers | Status |
|---|---|---|
| [inventory-storage.md](inventory-storage.md) | `Inventory`/`SlotController`/`Slot`/`Storage` core: which methods see which slots, `playerSlot`, quick move, sorting, `storageSlots` swap pitfall, storage panel buttons | code-only |
| [backpack.md](backpack.md) | worn backpack: items, contents, how it opens, differences from main slots | code-only |
| [containers-theft.md](containers-theft.md) | opening a container, taking items, what counts as theft, slot icons | partly verified |
| [player-stats.md](player-stats.md) | `PlayerStats` parts: hunger, thirst, bowel, bladder, health, mental health, hygiene, alcohol, addictions, mushrooms, conditions, skills | code-only |
| [player-movement-tiredness-sleep.md](player-movement-tiredness-sleep.md) | walk/run/crouch, tiredness, sleep debt, coffee, beds, pass-out, alarm clock | code-only |

## Building and devices

| Note | Covers | Status |
|---|---|---|
| [item-system.md](item-system.md) | `Item`, `ItemStack`, `ItemDatabase`, item meta types, perishing, stolen/owner marking, `Items.json` shape | code-only |
| [economy.md](economy.md) | `Money`, shops, traders, bank, loans, rent, difficulty price scaling | code-only |
| [doors-interaction.md](doors-interaction.md) | furniture placing/taking, `Door`/`Lock`, keys, the interaction picker | code-only |
| [crafting.md](crafting.md) | `CraftingBase`: recipe types, conditions, `EquipmentQuality`, `CraftingUpgrade`, ingredient matching | code-only |
| [growing.md](growing.md) | beds/terrariums: planting, harvest, on-vine spoilage, quality/freshness | code-only |
| [power.md](power.md) | `PowerSource`, fridges, lamps, TV, generator, outages | partly verified |
| [power-grid.md](power-grid.md) | `PowerGrid` network (build 0.5.13): listeners, capacity and cuts, tenement main grid, resident load, billing, meter | code-only |
| [water.md](water.md) | `LiquidStorage`, global source, taps as recipes, water items, shower, toilet, hygiene, water supply levels | partly verified |
| [time-and-weather.md](time-and-weather.md) | `TimeOfDayAzure` clock, events and timers, day counter, waiting/sleeping, weather | code-only |
| [tenement-upgrades.md](tenement-upgrades.md) | building/apartment upgrade levels (water, electricity, heating), storage of state | partly verified |
| [furniture-areas.md](furniture-areas.md) | `Furniture`, building areas, which room an object is in; addressable asset names, generator | code-only |
| [police-crime.md](police-crime.md) | `Act_Police` states/routes/speeds, crime records, `Witness`, trespassing areas, stolen-goods marking, arrest and search flow | code-only |
| [combat-npcs.md](combat-npcs.md) | damage paths (who can be hurt), player health/death, `Act_Robber` (the only human that attacks), animal bites, scene spawners, infected-model search | code-only |
| [periodic-rendering.md](periodic-rendering.md) | reflection probes re-rendered on timers (`ReflectionProbeUpdater`, Azure environment controller); suspects for regular hitches; the outdoor daytime hitch | partly verified |

## World and saves

| Note | Covers | Status |
|---|---|---|
| [levels.md](levels.md) | `SaveController.ChangeLevel`, entrypoints, where the player spawns; level doors, preload, spawners, portals | partly verified |
| [saves.md](saves.md) | save folder layout, copying, reading a save file; save/load hook order, autosave, versioning; mod guidance: per-save mod data | code-only |
| [main-menu.md](main-menu.md) | title-screen menu: world-space sign canvas, 6 buttons, hard-coded gamepad navigation | partly verified |
| [tasks-relays.md](tasks-relays.md) | Lua bridge to dialogue, `TaskController`, recipe learning, notes, `Relay` and `logic_*` scene glue | code-only |
| [npc-schedules.md](npc-schedules.md) | NPC data files, `Schedule`/`Activity`, residents, relationships, `CharacterSpawner` | code-only |
| [ui-options-input.md](ui-options-input.md) | HUD and panels, settings files, key bindings, notifications, menu localization | code-only |
| [quick-slots.md](quick-slots.md) | quick-slot bar (build 0.5.13): owner API, input, quick plant | code-only |
| [audio-camera-graphics.md](audio-camera-graphics.md) | soundscape and mixer, radio, camera, graphics quality, post-processing, light dimming | code-only |
| [minigames.md](minigames.md) | card tables, slot machine, coin pusher, gambling stats, `Keypad`, computers | code-only |
| [animals-collectibles.md](animals-collectibles.md) | animals (state, food, breeding, followers), collectible spawners, resource nodes, ambient fish | code-only |
| [map-travel-trains.md](map-travel-trains.md) | map and landmarks, fast travel entry, `Teleport`, trains, NPC pathfinding graphs | code-only |
| [achievements-boot.md](achievements-boot.md) | achievements and Steam calls, version handling, build switches, boot order, hidden console commands | code-only |
| [consumables.md](consumables.md) | `ItemConsumable` to stat mapping, smoking and alcohol, auto-consume, doctor treatments | code-only |
| [waste-junk.md](waste-junk.md) | junk sources, `FurnitureTrash`, janitor actions, apartment cleaning timer | code-only |
| [tenement-rent.md](tenement-rent.md) | renting, move-in and eviction flow, resident requirements, contractor caps, prefab swapping | code-only |
| [mail-invoices-phone.md](mail-invoices-phone.md) | mail and parcel delivery, random mail, invoices and expiry, phone calls, legal services | code-only |
| [interactable-catalog.md](interactable-catalog.md) | one line per `Interactable` subclass: what Interact does, key fields | code-only |
| [npc-movement.md](npc-movement.md) | NPC path search, speeds, cross-scene walking, door waypoints, spawn/despawn, stuck handling | code-only |
| [skills.md](skills.md) | skill enum, gain formula, where each skill is gained and used, character panel | code-only |
| [shops-internals.md](shops-internals.md) | furniture shop, market checkout, vending machines, tickets, restock and saving | code-only |
| [smv-sauna-radiation.md](smv-sauna-radiation.md) | sauna overheat, radiation dose, duck, SMV progression and the wasteland effect, face protection | code-only |
| [fuel-heat.md](fuel-heat.md) | `Fuel` burn time and ash, candle lantern, sauna temperature, what heating levels do | code-only |
| [events-catalog.md](events-catalog.md) | static events usable as hook points (save, mail, tasks, inventory, crime, UI, weather) | code-only |
| [equipment.md](equipment.md) | equip slots and routing, hand items, flashlight, face protection, worn-item effects | code-only |
| [lua-functions.md](lua-functions.md) | every Lua function the game registers for dialogue scripts: owner, arguments, effect | code-only |
| [loading-intro-death.md](loading-intro-death.md) | loading screen, hint overlay, event cameras, blackout and wake-up, sleep event names, death entry | code-only |
| [setting.md](setting.md) | the game world in brief, for naming and flavour | unverified source |

## Lore — FULL SPOILERS (these notes tell the whole story of the game)

Digested by language-model agents from the game's dialogue database and texts, spot-checked against the raw text; not played. Status for all: unverified source. The source dumps can be regenerated with `dialogue_export.py` (see asset-bundles.md).

| Note | Covers |
|---|---|
| [lore/01-overview.md](lore/01-overview.md) | premise, main arc, endings, timeline |
| [lore/02-characters.md](lore/02-characters.md) | named characters by place/role |
| [lore/03-places-and-factions.md](lore/03-places-and-factions.md) | locations, factions, organisations |
| [lore/04-questlines.md](lore/04-questlines.md) | quest chains with steps and branches |
| [lore/05-world-and-lore.md](lore/05-world-and-lore.md) | disease, mushrooms, economy, law, culture |
| [lore/06-open-questions.md](lore/06-open-questions.md) | contradictions and unclear points |

## Data

| Note | Covers | Status |
|---|---|---|
| [items-recipes.md](items-recipes.md) | `Items.json` / `Recipes.json` shape and encodings, recipe conditions in use, exact counts per type, skill and category | code + data |

## Tooling (ours, not game knowledge)

| Note | Covers | Status |
|---|---|---|
| [dev-console.md](dev-console.md) | the game's F1 console: commands, the calls under them; mod guidance: adding your own | code-only |
| [imgui.md](imgui.md) | IMGUI pitfalls for mod menus (title placement, popups, rects in scroll views) | partly verified |
| [asset-bundles.md](asset-bundles.md) | reading prefab/scene data offline with UnityPy: setup, what the helper scripts do, where data lives, pitfalls | tooling |

## Known gaps (ordered by use for current/planned mods)

1. Generic `Storage` API in detail (`EnterExternally` flow, `storageSlots` save format, `onUpdate`) — only partly in inventory-storage.md and containers-theft.md.
2. Re-check every note against the newest game build; the first research pass (2026-10-24) was written against the 15.10 decompile, and a later update has since been released.
3. Gamepad navigation (`UINavigationCore`, `InventoryNavigationHandler`), `ItemInfoPanel` internals.
4. Scene-level facts: which prefab or scene holds a value (`Doctor.treatments`, keypad codes, `MailController.delay`, spawner settings). Use offline asset reading (asset-bundles.md) first; a runtime dump only for live state.
5. Animal prefab values (bite, hunt chance, spawn timers) and the runtime bite check described in animals-collectibles.md.
