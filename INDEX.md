# Obenseuer knowledge — index

Open this file, then **one** note. Each note starts with Title / Scope / Status / Related.

**Version:** notes were written against game 0.5.08. The installed build is the 15 Oct 2026 Steam update (`Assembly-CSharp.dll` dated 15 Oct 22:57); whether the version number changed is unconfirmed. A note gets an extra line only if it was re-checked against a different build.

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
| [player-movement-tiredness-sleep.md](player-movement-tiredness-sleep.md) | walk/run/crouch, tiredness, sleep debt, coffee, beds, pass-out, alarm clock | code-only |

## Building and devices

| Note | Covers | Status |
|---|---|---|
| [crafting.md](crafting.md) | `CraftingBase`: recipe types, conditions, `EquipmentQuality`, `CraftingUpgrade`, ingredient matching | code-only |
| [growing.md](growing.md) | beds/terrariums: planting, harvest, on-vine spoilage, quality/freshness | code-only |
| [power.md](power.md) | `PowerSource`, fridges, lamps, TV, generator, outages | partly verified |
| [water.md](water.md) | `LiquidStorage`, global source, taps as recipes, water items | partly verified |
| [tenement-upgrades.md](tenement-upgrades.md) | building/apartment upgrade levels (water, electricity, heating), storage of state | partly verified |
| [furniture-areas.md](furniture-areas.md) | `Furniture`, building areas, which room an object is in; addressable asset names, generator | code-only |
| [police-crime.md](police-crime.md) | `Act_Police` states/routes/speeds, crime records, `Witness`, trespassing areas, stolen-goods marking, arrest and search flow | code-only |
| [combat-npcs.md](combat-npcs.md) | damage paths (who can be hurt), player health/death, `Act_Robber` (the only human that attacks), animal bites, scene spawners, infected-model search | code-only |
| [periodic-rendering.md](periodic-rendering.md) | reflection probes re-rendered on timers (`ReflectionProbeUpdater`, Azure environment controller); suspects for regular hitches; the outdoor daytime hitch | partly verified |

## World and saves

| Note | Covers | Status |
|---|---|---|
| [levels.md](levels.md) | `SaveController.ChangeLevel`, entrypoints, where the player spawns | partly verified |
| [saves.md](saves.md) | save folder layout, copying, reading a save file; mod guidance: per-save mod data | code-only |
| [main-menu.md](main-menu.md) | title-screen menu: world-space sign canvas, 6 buttons, hard-coded gamepad navigation | partly verified |
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
| [items-recipes.md](items-recipes.md) | `Items.json` / `Recipes.json` shape and encodings, recipe conditions in use | code + data |

## Tooling (ours, not game knowledge)

| Note | Covers | Status |
|---|---|---|
| [dev-console.md](dev-console.md) | the game's F1 console: commands, the calls under them; mod guidance: adding your own | code-only |
| [imgui.md](imgui.md) | IMGUI pitfalls for mod menus (title placement, popups, rects in scroll views) | partly verified |
| [asset-bundles.md](asset-bundles.md) | reading prefab/scene data offline with UnityPy: setup, what the helper scripts do, where data lives, pitfalls | tooling |

## Known gaps (ordered by use for current/planned mods)

1. Generic `Storage` API in detail (`EnterExternally` flow, `storageSlots` save format, `onUpdate`) — only partly in inventory-storage.md and containers-theft.md.
2. Re-check every note against the 15.10 build; confirm the game version number first.
3. Game UI framework: `GameUIController` panels, `Slot`/`ItemData` drag-drop, `ItemInfoPanel`, notifications, gamepad `InventoryNavigationHandler`.
4. Time and weather (`TimeOfDayAzure`, day/night, temperature) for heating/cold and outage ideas.
5. Needs and status effects beyond sleep (`PlayerStats`, hunger/thirst, illness, SMV) and item actions (`ItemConsumable`, ...).
6. Trade, money and prices (`Trade`, `Money`, `MarketShop`); NPCs, crime, witnesses (`Crime`, `Witness`).
7. Dialogue and quests (Lua function registry).
8. Runtime-only data: use offline asset reading (asset-bundles.md) for prefab values first; a runtime dump only for scene objects and live state.
