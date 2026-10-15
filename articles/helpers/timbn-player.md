# TimbnPlayer

Everything here reads as false, null or zero at the main menu. Signatures are in the [API reference](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnPlayer).

## Reading and Changing the Player

### `GameHasControl(ignored)`

Whether the game itself has taken the player's control, such as for a cutscene, sleep, or a teleport. Control taken by mods with `Player.TakeControl` does not count, so a mod holding the player can use this to notice the game stepping in. To skip a reason, pass it in, such as `TakenControlType.ByUI` to ignore open windows. Whatever else the game holds the player for still makes it return true.

```csharp
if (TimbnPlayer.GameHasControl(TakenControlType.ByUI))
    meditation.Stop();
```

### `Money`

The player's money in the game's raw units, where 100 is one silver coin.

### `TakeMoney(amount)`

Takes money from the player, as much as they have up to the amount. Returns how much was taken.

### `GiveMoney(amount)`

Gives the player money with the game's coin sound and notice. Nothing happens for 0 or less.

```csharp
var paid = TimbnPlayer.TakeMoney(250);
TimbnPlayer.GiveMoney(45);
```

### `GetResource(id)`, `AddResource(id, amount)` and `SetResource(id, value)`

Read and change other stats the game tracks for the player, such as `energy` or `insanity`, or a reputation from `TimbnNpcs.ReputationOf(npcId)`. The game's own limits apply. Passing a negative amount subtracts from the resource. For money, `GiveMoney` and `TakeMoney` show the coin notice.

```csharp
if (TimbnPlayer.GetResource("insanity") > 50f)
    TimbnPlayer.AddResource("insanity", -10f);
```

### `Carried`

The item the player carries over their head, or null. With several carry slots, it returns the item on top. See `Player.Carry` to give them one.

### `HandsFree`

True when nothing is carried over the player's head in any carry slot and nothing is held in front of them, such as seeds.

```csharp
if (TimbnPlayer.HandsFree)
    _mug = Player.Carry(new Item("my_mod_mug"));
```

### `Energy`

The player's energy, which the game caps at its maximum.

```csharp
if (TimbnPlayer.Energy < 20f)
    return;
```

### `CountItem(id)`, `HasItem(id, count)`, `GiveItem(item)` and `TakeItem(id, count)`

Shortcuts to the same calls on `TimbnItems` (`CountOnPlayer`, `GiveToPlayer` and `TakeFromPlayer`), for the player's inventory and bags. `HasItem` needs one by default.

```csharp
if (TimbnPlayer.HasItem("flitch", 5))
    TimbnPlayer.TakeItem("flitch", 5);
```

### `IsTired`

Whether the player has the game's tired debuff.

```csharp
if (TimbnPlayer.IsTired)
    return;
```

### `IsInFight`

Whether a fight is on, from the pre-fight build phase until the fight ends. The player keeps control the whole time, so a mod that holds or walks the player should check this first.

```csharp
if (TimbnPlayer.IsInFight)
    return;
```

### `HasPerk(id)`

Whether the player has a perk, by its id such as `perk_green_thumb`.

```csharp
var greenThumb = TimbnPlayer.HasPerk("perk_green_thumb");
```

### `AddPerk(id)` and `RemovePerk(id)`

Gives the player a perk or takes it away. `AddPerk` returns false for a perk the game doesn't have, and `RemovePerk` returns false when the player didn't have it. The perk is saved with the game, so use it for the game's own perks and for effects that should stay. A buff that must end with the mod belongs to [Potions](../potions.md).

### `HasInspiration(inspirationId)`

Whether the player has bought a level of an inspiration, by its level id such as `cemetery_expander_2`.

```csharp
if (TimbnPlayer.HasInspiration("cemetery_expander_2"))
    Logger.LogInfo("The graveyard has grown twice.");
```

### `HasTech(id)`

Whether the player has researched a technology, by its id such as `wood_basic`.

```csharp
var hasWoodBasic = TimbnPlayer.HasTech("wood_basic");
```

### `HasCraft(craftId)` and `UnlockCraft(craftId)`

Whether the player can make a recipe (including recipes that need no unlock), or unlocking one the way a cookbook does. For a mod's own recipe, `UnlockedBy` on [the craft](../stations-and-crafts.md) is usually the better choice.

```csharp
if (!TimbnPlayer.HasCraft("my_mod_fish_pie"))
    TimbnPlayer.UnlockCraft("my_mod_fish_pie");
```

### `HasBuilt(buildingId)`

Whether the player has built one of the buildings the game only allows once, such as the choir or the organ. The game marks such a building as locked to prevent rebuilding once it is built, which this method inspects.

```csharp
if (TimbnPlayer.HasBuilt("zmb_choir_unlock_s"))
    TimbnQuests.FireStoryTrigger(GlobalEventsSystem.Event.Type.BuildBuilding, "zmb_choir_unlock");
```

### `Position`

The player's position in their current scene.

### `SceneId`

The id of the scene the player is in, such as the graveyard or an interior.

### `Scene`

The scene the player is in, holding its world objects and the items dropped on its ground.

### `Zone`

The zone the player stands in, such as the garden or the morgue, or null in the open between zones. Zones are the game's named areas, each with its own build area and workers.

### `ZoneId`

The id of the zone the player stands in, such as `garden`, or null when standing outside any zone.

```csharp
var atHome = TimbnPlayer.ZoneId is "garden" or "inquisitions_base";
```

### `TeleportTo(position)`

Moves the player to a spot in their current scene, the way the game moves them. Pair it with `TimbnWorld.TryFindWalkable` so they land on open ground. Returns false at the main menu.

```csharp
TimbnPlayer.TeleportTo(campfire);
```
