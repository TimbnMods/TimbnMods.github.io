# Removing a Mod Safely

Players remove mods. The game fails to load when a save references content added by a missing mod, so Core keeps a mod's content out of the game's own save file. This page covers what is handled for you, what is not, and what to do in your own mod.

A save holding an item, buff, or station that no longer exists never finishes loading. A mod craft left in a station's queue freezes the world.

## What Core Handles

There is no API for this. It works for every mod that adds content through Core. Each of these is taken out of the game's save file as the game writes it, and Core puts it straight back into the running game afterward. When the save loads with the mod installed, it goes back into the save where it was. A save that already holds these when it loads, such as one written before Core handled this, has them moved out as it loads.

- Items you added, wherever they are. That covers the player's inventory, bags, chests and the ground.
- Buffs you added.
- Crafts you added, including one queued at a station.
- Stations you added.
- Parts you added to a world object, such as a 3D model.
- Your items set as the filter of a conveyor chest's slot, or pinned to the hotbar.
- Quests you added, with their progress.

The removed content is kept in Timbn's own save file next to the game's save. Without the mod the save loads as if the content never existed. Everything except buffs waits in that file and comes back if the mod does. A buff whose mod is missing when the save loads ends the way any buff ends, running its end effects. A craft the player had already started returns its ingredients only along with the mod, because they travel inside the craft.

A station that holds things when its mod is missing ejects them. When the save loads without the mod, everything the game can hold without it, such as a zombie lying in the station or ingredients in its slots, is placed on the ground where the station stood. Only your own items stay in the saved station. If the mod comes back, the station returns empty. A player who turns your mod off for a single session gets its stations back empty for that reason. A zombie that works at a station is not inside it, so it simply remains free in the world.

Core does not touch recipe unlocks or building unlocks. The game ignores ones it does not recognize, so they can stay in the save without causing problems.

## What May Be Left Behind

Core doesn't clean up everything. Some things your mod adds can stay in the save when it is removed, and what the game does with them depends on the case. Most only log errors, but some can change the save, so a player's save may end up wrong or broken. Keep the content your mod adds to the kinds listed above where you can, and tell players what to expect if you go beyond them.

## What to Do in Your Mod

- Keep your own data in [Save Data](saves.md), which lives in its own section of Timbn's save file and never touches the game's save. Never write your ids into a game structure yourself.
- Undo changes you make to game data around a save with `Events.BeforeSave` and `Events.AfterSave` (see [Game Events](events.md)).
- For a potion buff that changes C# state, use `ApplyChange` and `RevertChange` so the save never holds the change (see [Potions](potions.md)).
- Put direct changes to the game back in `Events.SaveClosed` or `OnUnload`. See [Game Events](events.md) for `SaveClosed` and [Getting Started](getting-started.md) for `OnUnload`.

## While Developing

A hot reload that drops your mod while one of its crafts is queued or running keeps logging errors until the next save loads, because the cleanup runs when the save is written and not when the mod unloads. This does not affect players, since they never unload a mod while the game runs.
