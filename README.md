# GUI-Core

GUI-Core is a Minecraft Java Edition datapack that implements an inventory-style
menu using a chest minecart, marker/display entities, storage and functions in
the `ui` namespace. This repository is a fork of
[timraay/Inventory-GUI](https://github.com/timraay/Inventory-GUI). The checked-in
pack describes itself as **GUI Library v1.0.0**.

> **Important: potentially destructive world changes.** On load, the pack runs
> `setblock 0 -64 0 yellow_shulker_box` in its execution context. Its stop
> function sets that same position to air. It also targets matching items and
> players with `kill` and `clear` during ticks. Do **not** install, reload or
> stop this pack in a world you care about without reviewing the functions and
> taking an independently verified backup. No Minecraft server or world was
> run for this documentation update.

## What is in this checkout

| Path | Responsibility |
| --- | --- |
| [`pack.mcmeta`](pack.mcmeta) | Pack metadata, description and declared format range. |
| [`data/minecraft/tags/function/`](data/minecraft/tags/function/) | Registers `ui:core/load` and `ui:core/tick` with Minecraft's load and tick tags. |
| [`data/ui/function/core/`](data/ui/function/core/) | Creates objectives and storage, runs tick processing and provides a stop function. |
| [`data/ui/function/player/`](data/ui/function/player/) | Opens, follows and closes player-linked menu entities and returns input items. |
| [`data/ui/function/minecart/`](data/ui/function/minecart/) | Loads pages and detects changes to the menu inventory. |
| [`data/ui/function/menu/`](data/ui/function/menu/) | Mask/refresh, input handling and action macros. |
| [`data/ui/tags/function/`](data/ui/tags/function/) | Empty `#ui:home` and `#ui:mask` extension tags. |
| [`data/minecraft/loot_table/blocks/yellow_shulker_box.json`](data/minecraft/loot_table/blocks/yellow_shulker_box.json) | Overrides the vanilla yellow-shulker-box block loot table for the pack's item-return mechanism. |

The source uses modern item components and function macros. `pack.mcmeta`
contains `pack_format: 48`, `supported_formats: [48, 94]`,
`min_format: [48, 1]` and `max_format: 94`. These are **declared metadata**,
not evidence that the pack has been tested across the whole range. The old
repository description mentions Minecraft 1.15+, but the current source uses
newer syntax; do not treat that description as a compatibility guarantee.

## How the menu works

1. The load function adds the `ui`, `ui.cd` and `ui.id` scoreboard objectives,
   initializes `ui` storage lists if missing, places the fixed-coordinate
   yellow shulker box and plays a sound to all players.
2. On each tick the pack runs player logic, processes entities tagged `ui`,
   clears matching menu items from players and kills matching dropped item
   entities. Specifically, the tick functions look for the `ui_item` custom
   data marker; do not use that marker for unrelated items.
3. A player holding an item matching `minecraft:custom_data={ui:1b}` in a
   weapon slot triggers `ui:player/teleport`. If needed, `ui:player/open`
   summons a linked `item_display`, chest minecart and marker and gives the
   player and entities matching `ui.id` scores.
4. The initial `#ui:home` function tag is invoked as the new chest minecart,
   then the current page is copied from `storage ui current` into its inventory.
   Tick logic compares inventory data with a previous snapshot to detect
   changes; page content and masks are managed through the `ui` storage.
5. Menu input and click processing can return inserted items, run an action,
   play a sound, refresh a page and apply a `ui.cd` cooldown. When the player
   stops holding the marker item, the player tick function closes the linked
   menu and removes its tagged entities.

Read the functions before integrating: this is a description of their current
implementation, not a stable API or a claim that every interaction is safe.
The `#ui:home` and `#ui:mask` tags in this tree contain no values, and there is
no complete example menu in the repository. An integrating datapack must supply
its own compatible functions and item data rather than expecting a ready-made
menu to appear.

## Extension points and data conventions

- [`data/ui/tags/function/home.json`](data/ui/tags/function/home.json) identifies
  functions called to prepare a new menu's initial page. It is empty here.
- [`data/ui/tags/function/mask.json`](data/ui/tags/function/mask.json) identifies
  mask functions used by `ui:menu/mask`; it is also empty here. The code copies
  between `storage ui mask` and `storage ui current`.
- The minecart tick/on-change functions use `storage ui current`, `previous`,
  `in` and temporary data. They detect `ui_item` custom data and use `.type` on
  the `ui` scoreboard to select the minecart implementation.
- [`ui:menu/run_command`](data/ui/function/menu/run_command.mcfunction) and
  [`ui:menu/run_function`](data/ui/function/menu/run_function.mcfunction) use
  function macros whose values come from an item's
  `components."minecraft:custom_data".ui_item.process`. The implementation
  selects a player with `@p[limit=1,sort=arbitrary]`; do not assume that this
  always identifies the intended clicker in a multiplayer world.
- [`ui:player/return`](data/ui/function/player/return.mcfunction) writes item
  data to the fixed shulker-box position and uses a custom-data-marked tool
  with `loot ... mine` to return contents. This depends on the overridden
  yellow-shulker-box loot table and can conflict with other datapacks that
  change the same resource.

Only trusted authors should supply function tags, item data, masks or macro
arguments. Macro-provided command/function text can cause commands to execute;
this checkout contains no general-purpose sanitization or authorization layer.
Review the exact functions and isolate experiments from production worlds.

## Installation and removal precautions

The repository root is the datapack layout: `pack.mcmeta` and `data/` are at
the same level. Before trying it in an expendable test world, check the game's
actual datapack-format compatibility, back up the world, inspect the target
coordinate and review interactions with other datapacks. Loading/reloading a
pack runs its load tag, so merely testing installation can change the world.

[`ui:core/stop`](data/ui/function/core/stop.mcfunction) removes the three
objectives and sets `0 -64 0` to air. It is **not** a general-purpose rollback:
it does not restore the previous block or prove every entity, item, storage
value or changed loot-table effect has been restored. Plan a backup-based
recovery and inspect the code rather than relying on the function as an undo.

## Licensing and provenance

GitHub identifies this as a fork of the upstream project linked above. No
`LICENSE` file is present in this tracked checkout. Do not assume a license or
redistribution permission from the absence of a file; inspect upstream terms
and obtain clarification from the rights holders where necessary.
