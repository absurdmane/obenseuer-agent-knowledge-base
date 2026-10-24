# Changelog

One dated entry per update: which notes were added or changed. After a `git pull`, read the newest entries to see what is new.

## 2026-10-25 (update to build 0.5.13)

- Re-checked against build 0.5.13 with a method-level comparison; corrected `power.md`, `fuel-heat.md`, `growing.md`, `crafting.md`, `skills.md`, `waste-junk.md`, `animals-collectibles.md`, `tenement-upgrades.md`, `tenement-rent.md`, `doors-interaction.md`, `saves.md`, `player-movement-tiredness-sleep.md`, `periodic-rendering.md`, `furniture-areas.md`, `water.md`, `economy.md`, `shops-internals.md`, `inventory-storage.md`, `item-system.md`, `consumables.md`, `items-recipes.md` (new exact data statistics: 929 recipes, 1240 items, recipe type Bottler).
- New: `power-grid.md` (the power grid), `quick-slots.md`.
- Notes that cite no changed member were not edited.

## 2026-10-24 (research pass)

- New: `time-and-weather.md`, `item-system.md`, `economy.md`, `player-stats.md`, `doors-interaction.md`, `tasks-relays.md`, `npc-schedules.md`, `ui-options-input.md`, `audio-camera-graphics.md`, `minigames.md`, `animals-collectibles.md`, `map-travel-trains.md`, `achievements-boot.md`, `consumables.md`, `waste-junk.md`, `tenement-rent.md`, `mail-invoices-phone.md`, `interactable-catalog.md`, `npc-movement.md` (with behaviour-tree notes), `skills.md`, `shops-internals.md`, `smv-sauna-radiation.md`, `fuel-heat.md`, `events-catalog.md`, `equipment.md`, `lua-functions.md`, `loading-intro-death.md`; `items-recipes.md` gained exact data statistics; `animals-collectibles.md` gained movement, hunting, attacks and the bite-damage finding; `levels.md` gained level doors, spawners and portals.
- Added: `water.md` (shower, toilet, hygiene, supply levels), `power.md` (how a device learns about power; corrected the `FurnitureManager.NetPower` remark), `crafting.md` (recipe fields, production flow), `saves.md` (save/load order, autosave, versioning).

## 2026-10-22 (credit)

- `SKILL.md`: section 6 suggests, with the user's yes, crediting the author of the knowledge base in a mod's README or description.

## 2026-10-21 (update check)

- `SKILL.md` and `README.md`: the daily update check now asks the user only when GitHub really has new commits (fetch, then compare with the remote); with nothing new it stays silent.

## 2026-10-21

First public version, written against Obenseuer 0.5.08.

- Added the notes on the game's subsystems: asset bundles, backpack, combat and NPCs, containers and theft, crafting, dev console, furniture and areas, growing, ImGui, inventory and storage, items and recipes, levels, main menu, periodic rendering, player movement, tiredness and sleep, police and crime, power, saves, tenement upgrades, water, and the setting.
- Added `lore/`, six notes that digest the game's story. They spoil the whole game.
- Added `INDEX.md`, `README.md`, `SKILL.md` (the agent skill), `CONTRIBUTING.md` and the issue and pull request templates.
