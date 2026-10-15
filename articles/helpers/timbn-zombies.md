# TimbnZombies

Signatures are in the [API reference](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnZombies).

## Finding Zombies

### `All()`

The zombies standing in the world of the loaded save, whatever job they have. Zombies lying on the ground as items are not included. Empty at the main menu.

```csharp
var carriers = TimbnZombies.All().Count(zombie => zombie.ZombieType == ZombieType.Caretaker);
```

### `AtStation(station)`

The zombie a station holds, either lying in it (a station made with [`HoldsZombie`](../stations-and-crafts.md#holdszombie), or the game's autopsy table) or working it. Null when it holds none. Use it to pay the zombie tech points when a craft there ends.

```csharp
TimbnZombies.AtStation(stand)?.DoTechPointsReward(stand, 50, 0, 0);
```

### `Looks(dataId)`

The bodies, heads, and skin colors the game picks from when randomly generating a zombie's appearance. A mod that restyles zombies offers these so its zombies look like the game's own. `dataId` is the kind of zombie and defaults to the worker zombie. Returns null when the game has no such kind. The result has `Bodies`, `Heads`, `BodyColors` and `HeadColors`. The first two are lists of ids (numbers) and the colors are lists of texture names.

```csharp
if (TimbnZombies.Looks() is { Heads.Count: > 0 } looks)
    head = looks.Heads[UnityEngine.Random.Range(0, looks.Heads.Count)];
```

### `Redraw(zombie)`

Redraws a zombie after its appearance or gear has changed in its data, the way the game does when it spawns. It covers a zombie standing in the world, one lying on the ground as an item, and a fighter. `Events.ZombieViewReady` runs again for a zombie standing in the world, except for fighter zombies. Returns false when the zombie has no view to redraw right now, or at the main menu.

```csharp
ZombieSkinHelper.ApplySkinToZombieWgoData(zombie, body, head, bodyColor, headColor);
TimbnZombies.Redraw(zombie);
```

`ZombieSkinHelper` is the game's own class. Its `body` and `head` are numbers and its two colors are text names of the color tables, such as the ones `Looks` returns.
