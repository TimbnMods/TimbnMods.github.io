# Game Ids

Almost everything in the game is named by a text id, such as `coal` for coal or `npc_herm` for Herm. [`TimbnGameIds`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnGameIds) lists those ids as constants, so you can pick one from your editor's autocomplete instead of typing it from memory.

```csharp
Npcs.Find(TimbnGameIds.Npcs.Herm)?.WalkTo("npc_sim_village_right_side_03");
Audio.PlaySound(TimbnGameIds.Sounds.CoinsSound);
```

A constant is the same text id the game uses, so it works anywhere an id string does. The difference is that a misspelled constant fails to compile, and if a game update removes an id the build tells you.

## Finding an Id

Type `TimbnGameIds.` and your editor lists the groups below. Type a group and a few letters of the name, such as `TimbnGameIds.Items.Co`, and it narrows the list. Most constants carry a short summary with the name the game shows the player, which your editor shows beside the suggestion.

The same list is in the [API reference](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnGameIds), where you can search for a name.

## How a Constant Is Named

The name is the id with its underscores removed and each word capitalized, so `coal` is `Coal` and `cook_muffin` would be `CookMuffin`. A few groups also drop the prefix every id in the group starts with.

- `npc_herm` is `Npcs.Herm`.
- `buff_beer` is `Buffs.Beer`.
- `perk_alchemist` is `Perks.Alchemist`. A zombie perk such as `zperk_alchemist` becomes `Perks.ZombieAlchemist`.
- `builder_alchemy_lab` is `BuildDesks.AlchemyLab`.
- `tech_blue` is `TechPoints.Blue`.

A name can't start with a digit, so `1h_stone` becomes `Items.Item1hStone`.

## What Is Listed

Each group is a nested class, so you write `TimbnGameIds.<Group>.<Name>`.

| Group | Holds |
| --- | --- |
| `Items` | Every item the game defines, such as `Coal`. |
| `Sounds` | Sound effects, for `Audio.PlaySound`, such as `CoinsSound`. |
| `Music` | Music tracks, for `Audio.PlayMusic`, such as `Violin`. |
| `Techs` | Every unlock in the tech tree. |
| `TechPoints` | The three kinds of tech point, `Red`, `Green`, and `Blue`. |
| `Perks` | Every perk, such as `Alchemist`. |
| `Buffs` | Every buff, such as `Beer`. |
| `BuildDesks` | The desks the player builds from, such as `Yard`. |
| `BuildTabs` | The tabs of a build desk's menu, such as `BuildingExtension`, for a station's `BuildTab`. |
| `Workstations` | Crafting stations and bench extensions, in a class for each build desk, such as `Workstations.Yard.ToolRack`. Ones no desk builds are under `Placed`. |
| `Npcs` | Every kind of NPC, such as `Herm`. `Npcs.All` holds every ID, to check one written by hand. |
| `Groups` | The game's life simulator groups, such as `TownA1DayAll`. |
| `NpcEvents` | Script events an NPC can run, in a class for each character script, such as `NpcEvents.Herm.GoHome`. |
| `NpcAnimations` | Animations an NPC can play, in a class for each body set, such as `NpcAnimations.Herm.AdjustingJabbot`. |

Points on the map aren't listed. Name a point by its id string, as the [NPCs](npcs.md) page shows.

## Events and Animations

Events and animations are grouped differently because they only work for some NPCs.

An event belongs to a character's own script, so `NpcEvents.Herm.GoHome` works for Herm and not for a guard. Story quest events are left out.

An animation belongs to a body set, which several NPCs can share. A class holds the clips of one set, and a clip is listed only if the set's art has the frames for it, since anyone else disappears while it plays. Check that the NPC you animate wears that set.

## Where It Comes From

The constants are generated from the game's data after each game update, so they follow the game and nothing in them is edited by hand. Test ids and Timbn's own ids are left out. An id the game removed disappears from the list, which is how a build error tells you to look again.

## Using the Ids

Anything that asks for an item, NPC, sound, perk, or station by ID takes one of these.

```csharp
Dialog.AddTalk(TimbnGameIds.Npcs.Larry, "my_larry_trade", () => true, talk => talk
    .SayAs(TimbnGameIds.Npcs.Herm, "Don't sell him any."));
```
