# TimbnItems

Signatures are in the [API reference](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnItems).

## Working With Items

### `CountAnywhere(id, enough)`

Counts an item everywhere in the world. That covers the player's inventory, every storage and craft inventory, and every item lying on the ground or queued to drop. It stops looking once `enough` is reached, so asking "are there at least this many" is cheap. Leave `enough` out to count everything.

```csharp
var hasEnough = TimbnItems.CountAnywhere("flitch", 2) >= 2;
```

### `CountOnPlayer(id)`

Counts an item in the player's inventory and bags. Storage in the world does not count.

### `GiveToPlayer(item)`

Puts an item into the player's inventory and bags. Drops whatever does not fit onto the ground in front of the player so nothing is lost. A big stack is split into several pickups, the way the game drops items. Returns how many went into the inventory.

```csharp
var kept = TimbnItems.GiveToPlayer(new Item("flitch", 3));
```

### `TakeFromPlayer(id, count)`

Removes an item from the player's inventory and bags, up to the count asked for. The items are gone, so check `CountOnPlayer` first when the player has to have enough. Returns how many were taken.

```csharp
if (TimbnItems.CountOnPlayer("flitch") >= 5)
    TimbnItems.TakeFromPlayer("flitch", 5);
```

### `DropAt(item, sceneId, position)`

Drops an item on the ground as a pickup, in the scene with that id. Nothing happens at the main menu.

### `SalePrice(id)`

The sell price of an item, matching what town pallets and vendor orders pay, in the game's raw money units where 100 is one silver coin. Returns 0 for an item the game doesn't have or that sells for nothing.

```csharp
var price = TimbnItems.SalePrice("cook_muffin");
```

### `IconId(id)`

The sprite id of an item's icon, for anything that takes a sprite id, such as the `Icon` of a quest, craft or station. The sample below uses the quest builder's `Icon` overload that takes a function of the run. The craft and station builders take the id itself. Returns an empty string for an item the game doesn't have.

```csharp
.Icon(run => TimbnItems.IconId(run.Item("dish")))
```

### `PlayerDropPosition()`

The spot in front of the player where the game itself drops things, or the player's own position when that spot is blocked.

### `FindDrops(match, scene)`

Finds items lying on the ground as pickups, including ones queued to drop in a scene that is not currently loaded. Drops already being picked up or removed are skipped. Leave `scene` out to look in every scene of the world. Each result comes with the scene it lies in, which is what `RemoveDrops` takes.

```csharp
var logs = TimbnItems.FindDrops(drop => drop.Id == "wood", TimbnPlayer.Scene);
```

### `RemoveDrops(drops)`

Removes drops found with `FindDrops` from the ground. The items are gone. Returns how many items were removed, counting each drop's stack.

```csharp
var removed = TimbnItems.RemoveDrops(TimbnItems.FindDrops(drop => drop.Id == "wood"));
```
