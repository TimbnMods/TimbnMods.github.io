# NPCs

Core can give NPCs a daily schedule on the game clock, move them around, and add new ones. NPCs, groups, events, and animations come from the generated `TimbnGameIds.Npcs`, `TimbnGameIds.Groups`, `TimbnGameIds.NpcEvents`, and `TimbnGameIds.NpcAnimations`, and a point is named by its ID string. Everything is owned by the mod. When it unloads, any active walks stop, claimed points are released, and added NPCs are removed.

## Moving an NPC

### `Npcs.Find(npcId)`

Gets a handle to an NPC in the loaded save, or null when they are not in the world or at the main menu. Asking again in the same save gives the same handle. NPCs added with `Spawn` are excluded, so keep the handle `Spawn` gave you.

```csharp
Npcs.Find(TimbnGameIds.Npcs.Herm)?.WalkTo("npc_sim_village_right_side_03");
```

A [`TimbnNpc`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnNpc) handle has these.

| Member | What it does |
| --- | --- |
| `WalkTo(pointId, walk, arrived, failed)` | Walks them to a point and turns them the way it faces. `walk` picks the route. `TimbnWalk.Paths` follows the game's roads (the default) and is skipped when the paths don't join. `Navmesh` goes straight over walkable ground and around obstacles, but not along the roads and never through doors. `Straight` is a straight line without pathfinding around obstacles, for a short step only. `arrived` runs when they get there, and `failed` when the walk started but could not reach the point, so a chain of walks can go on or give up. Neither runs when the walk is stopped or replaced. The call returns false when the point is missing or unreachable. |
| `StopWalking()` | Ends the walk, whoever started it, and leaves them standing where they are. |
| `Teleport(pointId)` | Puts them on a point at once, facing the way it faces, and ends any walk. Returns false when the point is missing. |
| `Teleport(position)` | Puts them on a spot at once, keeping the way they face, and ends any walk. It takes an NPC through a door only the Keeper uses, such as from the church's outside landing `tp_RT_church_enter` to its inside one `tp_RT_church_exit`, which are world objects and not points. |
| `Face(direction)`, `Face(position)` | Turns them to a direction or toward a world position, such as a grave. |
| `Say(text, seconds)` | A line in their own speech bubble, from a text key or the text itself. It follows them as they walk, fades by itself and takes no control from the player. `seconds` is how long it stays once written out, and 0 uses the game's time for its length. |
| `Animate(trigger)` | Plays an animation of their own, such as `TimbnGameIds.NpcAnimations.Herm.AdjustingJabbot`, until their next walk. It returns false when they have no clip for it, since any other trigger would make them vanish. |
| `FireEvent(eventName)` | Fires one of their script events, such as `TimbnGameIds.NpcEvents.Herm.GoHome`. Core sets their `busy` and `on_spot` flags to 0 first, since the game's scripts only act while both are 0. Only fire events that move or animate, since story events start real story and trade. |
| `RunSteps(steps, done)` | Plays a list of steps on command, such as for a quest or a cutscene. A new run replaces the one before, and their schedule steps aside while it plays. `done` runs after the last step, and not when the run is stopped or replaced. Returns a handle that stops the run when disposed, or null when they are gone. |
| `IsFree(pointId)`, `BlockedReason(pointId)`, `Claim(pointId)` | Check and claim a point so two NPCs never take the same one. `Claim` returns false when someone else holds the point or one of the game's villagers is at it or on the way. A claim made during a trip is released when the trip ends, while an earlier claim is kept. |
| `RollChance(chance, what)` | Rolls their dice for this game minute against a chance from 0 to 1, such as whether to set off on a trip. Returns the dice to keep choosing with, or null on a miss. The same minute rolls the same on a reload. A miss is logged with `what` named. A second form also hands back the number rolled. |
| `CollidesWithPlayer` | Whether they block the Keeper like a wall, true by default. Set it to false for a crowd the Keeper should walk through, such as visitors standing about the graveyard. It only changes their solid parts toward the Keeper's body, so they are still talked to, hit, and walked around by everyone else, and it carries over when they stream out of the world and back in. |
| `Id`, `Data`, `Position`, `IsWalking`, `IsGone` | Their definition id, their world object, where they are, whether they are walking, and whether they have left the world. |
| `ReputationId` | The player resource that holds their reputation with this NPC, or null for someone the game keeps none with. The same as `TimbnNpcs.ReputationOf(Id)`. |
| `Dispose()` | Stops what the handle started and releases its claim. For an NPC `Spawn` added, it also removes them. |
| `MinutesLeftInBlock` | Game minutes before the next block of their schedule starts, or 0 for an NPC no schedule drives. When the day has only one block, it counts down to the next time that block's start comes around, so it is a full 24 hours only at the moment the block starts. Use it to leave a long trip alone when the block is about to change. |

## A Day

`Npcs.Schedule(npcId, day)` gives an NPC a daily schedule that applies to every save the player loads. Set it up once in `OnAwake`. A day needs at least one block, or the call throws. A day is divided into blocks, each starting at a specific time of day, written from 0 at midnight through 0.25 at dawn and 0.5 at noon to 0.75 at dusk. A block's intro runs once when it starts, and its loop repeats until the next block begins. A block joined late, such as after loading a save, sleeping, or jumping the clock, runs its late intro instead of the normal intro.

```csharp
var day = new TimbnNpcDay()
    .Block("working", 0.25f, block => block
        .Origin("RT_herm_spawn")
        .Intro(steps => steps.FireEvent(TimbnGameIds.NpcEvents.Herm.GoToSpawn))
        .LateIntro(steps => steps.FireEvent(TimbnGameIds.NpcEvents.Herm.TpToSpawn))
        .Loop(steps => steps.WaitGameMinutes(10, 30).Decide(PickTrip)))
    .Block("night", 0.85f, block => block
        .Intro(steps => steps.FireEvent(TimbnGameIds.NpcEvents.Herm.GoHome))
        .LateIntro(steps => steps.FireEvent(TimbnGameIds.NpcEvents.Herm.TpToSpot)));

Npcs.Schedule(TimbnGameIds.Npcs.Herm, day);
```

`PickTrip` is your own method. It takes a `TimbnNpc` and returns a `TimbnNpcSteps`, or null when there is nothing to do, such as `TimbnNpcSteps? PickTrip(TimbnNpc npc)`.

`Npcs.Schedule` returns a [`TimbnNpcSchedule`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnNpcSchedule). `Describe()` gives one line on the block of the moment and what the NPC is doing, handy for a debug window. Dispose it to stop the schedule early, which ends its walk and releases its claim. It also stops when the mod unloads.

The NPC pauses for conversations or story scenes. When the game moves them directly, their schedule pauses until a later block starts. A new block also waits for a trip picked by `Decide` to end.

| Day call | What it does |
| --- | --- |
| `OnlyWhile(condition)` | Runs the day only while a check is true, such as once a quest is done. |
| `StartWhenAt(pointId)` | Starts the day only once the NPC stands on this point, so an NPC the story keeps elsewhere is left alone. Standing on a point is never measured by distance. It means a walk of the day ended there and they have not moved since, or the game's life simulator has them on that point, or at home with that point as their home. |
| `ResumeWhenAt(pointId)` | After the game moved them, carries on only while they stand on this point, decided the same way. |
| `Doorway(pointId, viaId, walk)` | Makes every walk from or to a point go by another point, such as a home point behind a counter. The hop between the two goes over the navmesh unless `walk` says otherwise. |
| `LogSteps(when)` | Logs what the day does, for debugging. Pass a check to log only while it holds, such as a hidden setting. |

A block takes `Origin(pointId)` (where they are put back when a walk finds no route), `Intro`, `LateIntro` and `Loop`.

## Steps

A [`TimbnNpcSteps`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnNpcSteps) is a list of things for an NPC to do in order. Build one in a block, return one from `Decide`, or hand one to `RunSteps`.

| Step | What it does |
| --- | --- |
| `Walk(pointId)`, `WalkNavmesh(pointId)`, `WalkNavmesh(x, z)` | Walk along the paths by the shortest way, so name only the stops that matter, or over the navmesh. The last form takes a spot with no game point, such as a corner of a room. |
| `WalkStraight(x, z)` | Walks in a straight line to a spot, without pathfinding around obstacles. Use it for the short step onto a spot the navmesh leaves out, such as a placed bench's seat, or to keep to a line, since a navmesh walk picks its own way between two spots. |
| `WaitGameMinutes(minutes)`, `WaitGameMinutes(shortest, longest)` | Stand still for game minutes, which follow the game's speed. Skipped when the NPC found their claimed point taken. |
| `WaitRealSeconds(seconds)` | Stand still for real seconds regardless of the game speed. |
| `Face`, `Animate`, `Teleport` | The same as on the NPC. |
| `FireEvent(eventName)` | Fires a script event like on the NPC, then waits until the walk the event starts ends, for up to 180 game minutes. |
| `RunCode(action)` | Runs your `Action<TimbnNpc>` when the step is reached. |
| `Decide(npc => steps)` | Picks what to do when the step is reached, such as rolling for a trip. Return null to do nothing. A point claimed during the trip is let go when it ends. |
| `ChainTrips(pick)` | Picks a [`TimbnNpcTrip`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnNpcTrip) starting from the origin and runs it, then asks again from where it ended, so trips chain without going back in between. `pick` receives the NPC and the trip that just finished, which is null at the origin. Return a trip to run it, or null to stay put or to go home. A trip is built with `Leave` (only when it starts from the origin), `Body` (always), and `Return` (only when no next trip chains on) steps. Once the next block is due, no trip chains on. |
| `GoToFreePoint(points, teleport)`, `GoToFreePoint(groupId, prefix, teleport)` | Claims a free point from a list or one of the game's life simulator groups and walks there. `teleport` puts them on the point at once instead, such as when a block is joined late. When no point is free the step does nothing. |

## Rolling Dice

[`TimbnRolls`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnRolls) provides dice for an NPC's daily choices. They produce consistent roll results for the same key and the same game minute or day, so a reload or a second check at the same moment yields identical rolls. They are well-mixed, so neighboring minutes, days, and NPCs produce distinct, well-distributed rolls.

```csharp
var random = TimbnRolls.ThisMinute(TimbnGameIds.Npcs.Herm);
if (random.NextDouble() < 0.75)
    GoForAWalk();
```

### `TimbnRolls.ThisMinute(key)`

Dice for a key, such as an NPC's ID, that give the same numbers all through this game minute.

### `TimbnRolls.ThisDay(key)`

Dice for a key that give the same numbers all through this game day, such as for picking one thing a day.

## Adding and Reading NPCs

### `Npcs.Spawn(npcId, pointId)`

Adds an NPC to the loaded save at a point, such as one more villager. The game's life simulator leaves them alone, so walk them yourself. They are removed when the handle is disposed, the mod unloads, or the player returns to the main menu, and are left out of every save. Returns the handle, or null when the point or the NPC is missing.

```csharp
Npcs.Spawn(TimbnGameIds.Npcs.Villager03, "npc_sim_village_left_side_road_01")?.WalkTo("npc_sim_village_tavern_03");
```

### `Npcs.Preload(npcIds, copies)`

Loads the look of some kinds of people in the background, so the first `Spawn` of each kind in a session doesn't stall a frame while the game loads it. Call it when a save is ready, with every kind the mod may spawn. `copies` defaults to 1 and counts the ones the game already shows, so a kind out in the village with `copies` at 1 gets nothing ready, and a second one spawned then still stalls. Ask for 2 when the game's own may be about.

```csharp
Events.SaveReady(() => Npcs.Preload(TimbnNpcs.GroupMemberIds(TimbnGameIds.Groups.VillagersDayAll), copies: 2));
```

### `Npcs.AddVillager(npcId)`

Adds another villager of the specified type who follows native life-simulation behavior, joining the same group, going out by day and returning home at night. They start at home and go straight out when the rest of the group is out. Removal works the same as for `Spawn`. Returns null when the type has no group or home, or the group has no point left for another villager.

### `Npcs.CloseTradeWhen(vendorId, isClosed, lineKey)`

Keeps a vendor's trade window shut while a check says they are away, with the vendor saying a line in its place. `lineKey` is a text key added with `Text.Add`, said in the vendor's speech bubble, and can be left out. The game's menu comes back once the vendor has finished speaking. If the vendor can't speak, the line shows at the bottom of the screen instead. Dispose the returned handle to let trade open again.

```csharp
Npcs.CloseTradeWhen(TimbnGameIds.Npcs.Herm, () => TimbnClock.Weekday == TimbnWeekday.Pride, "my_mod_herm_resting");
```

[`TimbnNpcs`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnNpcs) reads the game's own lists of places NPCs go.

| Member | What it returns |
| --- | --- |
| `GroupPoints(groupId, prefix)` | The points of a life simulator group, such as the tavern seats. |
| `GroupMemberIds(groupId)` | The kinds of NPCs the game placed in a group, one per NPC. NPCs a mod added are left out. |
| `FacingAt(pointId)` | The way an NPC faces on a point, or null. |
| `IsFree(pointId)` | Whether a point exists, is switched on, is unclaimed, and has nobody on it. An NPC's own `IsFree` also counts their own claim as free. |
| `ReputationOf(npcId)` | The ID of the player's resource that holds their reputation with an NPC, such as `rep_larry`, or null for someone the game keeps none with. |
