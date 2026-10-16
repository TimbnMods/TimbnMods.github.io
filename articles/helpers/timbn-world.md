# TimbnWorld

Finds things in the loaded world and reads how the world is doing. Everything here reads as empty, zero or false at the main menu. Signatures are in the [API reference](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnWorld).

> [!NOTE]
> A mod that uses the navigation graph needs its own reference to `AstarPathfindingProject.dll` from the game's `Managed` folder.

## Finding Things

### `Scenes`

The scenes of the loaded world, each holding its world objects, zones, and the items on its ground.

### `Objects(match)`

Every world object in the loaded save, in every scene, loaded around the player or not. World objects include everything placed in the world, such as chests, stations, graves, NPCs, and zombies. Pass a predicate to filter them.

```csharp
foreach (var station in TimbnWorld.Objects(wgo => wgo.CraftComponent != null))
    station.CraftComponent.ResetCraftsFromBalanceCache();
```

### `FindAll(definitionId)`

Every world object of one kind, such as every `tool_rack`, from the game's own lookup. It is much faster than calling `Objects` with a filter, and it returns a copy so it is safe to remove objects while looping through them.

```csharp
var racks = TimbnWorld.FindAll("tool_rack").Count;
```

### `Extensions(station)`

The world objects attached to a station, such as the ones built into its [slots](../stations-and-crafts.md#slots) or a tool rack built next to it. The game keeps these links on the station and saves them with it. Empty when nothing is attached or at the main menu.

```csharp
var stands = TimbnWorld.FindAll(machine.Id).SelectMany(TimbnWorld.Extensions).Where(wgo => wgo.id == "timbn_infusion_stand").ToList();
```

### `RunningCraft(station)`

The id of the craft at the front of a station's queue, running or waiting to start, or null when it has none, such as to check whether a stand started with [`Crafts.Start`](../stations-and-crafts.md#craftsstartstation-craftid) is still on its craft.

```csharp
var busy = TimbnWorld.RunningCraft(stand) == "timbn_infusion_stand_infusion";
```

### `Near(position, radius, match)`

The world objects in the player's scene within a flat distance of a spot, nearest first.

```csharp
var chests = TimbnWorld.Near(TimbnPlayer.Position, 5f, wgo => wgo.Inventory != null);
```

### `ViewOf(data)`

The `Wgo` view that draws a world object, or null when its chunk is not loaded around the player. The view is what the game's windows and effects work with.

### `Point(id)`

A point the game placed in a scene, the spots its people walk to and stand at, such as `donkey_04` on the road by the graveyard. A point has a position, a facing and an enabled flag, and its id is the same string the Npcs module takes. The ids are listed in `info_dump`'s points catalog. Null when no point has that id or at the main menu.

```csharp
var road = TimbnWorld.Point("donkey_04")?.Position;
```

### `PointsWithTag(customTag)`

Every point carrying a custom tag, for a set of spots the game marks the same way, such as the two seats each church pew adds with the tag `parishioner_bench_point`. Disabled points are included, so check `Enabled` when that matters. Empty when none carry the tag or at the main menu.

```csharp
var seats = TimbnWorld.PointsWithTag("parishioner_bench_point").Where(seat => seat.Enabled);
```

### `FlatDistance(first, second)`

The distance between two spots as seen from above, ignoring height.

### `GravesWithBodies()`

Every grave in the graveyard with a body buried in it, whether or not it has a stone or a fence. Open graves and graves left empty after an exhumation are not counted.

## Zones and Ground

### `ZonesAt(position, scene)`

The zones a spot lies in, smallest first. Zones can nest, so a spot in a room is also in the building around it. It looks in the player's scene unless you pass another `scene`, and it reads the scene's data, so it works for a scene that is not loaded around the player.

```csharp
var zone = TimbnWorld.ZonesAt(TimbnPlayer.Position).FirstOrDefault();
```

### `TryFindWalkable(point, range, out ground)`

Finds the nearest walkable ground to a spot in the player's scene, measured flat as seen from above, so a spot on a roof or under the floor still finds the ground below or above it. Pair it with `TimbnPlayer.TeleportTo`. `range` is in world units. Returns false when nothing walkable is within it, and `ground` is then the spot you passed in.

```csharp
if (TimbnWorld.TryFindWalkable(TimbnPlayer.Position, 5f, out var ground))
    TimbnPlayer.TeleportTo(ground);
```

### `TryGetNavGraph(sceneId, out graph)`

Gets the navigation graph of a scene, which holds its walkable ground. The game calculates walking paths over it. Pass null for the scene id to use the player's scene. Returns false when the scene has no graph.

### `GraveyardBuiltAreas`

The graveyard's ground as seen from above, one rectangle of x and z for each module built. These are the areas whose graves count toward `GraveyardQuality`.

### `GraveyardBounds`

The whole graveyard as seen from above, its paths and grass as well as the ground built on, as one rectangle that stays the same regardless of how much has been built.

```csharp
var bounds = TimbnWorld.GraveyardBounds;
var spot = new Vector3(UnityEngine.Random.Range(bounds.xMin, bounds.xMax), 0f, UnityEngine.Random.Range(bounds.yMin, bounds.yMax));
```

## How the World Is Doing

### `GraveyardQuality` and `ChurchQuality`

The scores the game shows for the graveyard and the church.

### `TownQuality` and `TownGratitude`

The town's quality score and its gratitude. Gratitude grows from selling to town vendors and is capped at `TownQuality`, but it doesn't drop when the quality does.

### `Weather`

The weather by the game's own name for it, such as `CleanWeather`, `RainDayWeak` or `RainDayStorm`.

### `IsRaining`

Whether the weather is any kind of rain.

```csharp
if (TimbnWorld.IsRaining)
    Logger.LogInfo("Bring an umbrella.");
```
