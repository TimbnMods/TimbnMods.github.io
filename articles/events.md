# Game Events

The framework provides an easy way to listen to game events from Graveyard Keeper 2. Every subscription belongs to the mod and is removed when it unloads, so there is nothing to clean up. Each call also returns a handle that unsubscribes early when disposed, which you rarely need to keep.

```csharp
protected override void OnAwake()
{
    Events.SaveReady(OnSaveReady);
    Events.NewDayStarted(day => Logger.LogInfo($"day {day}"));
    Events.QuestCompleted(quest => Logger.LogInfo(quest.id));
}
```

> [!WARNING]
> Never subscribe to a game event with a raw `+=`. Core can only undo what you register through `Events`, so a raw handler stays on the game's event after a hot reload and the reloaded mod adds a second one. Every day, quest or item event then runs your code twice. Use a named event below, or `Events.GameEvent` and `Events.SaveEvent` for one without a name.

## Save Life

Two events cover the whole life of a save, so there is no need to handle a hot reload by hand.

### `Events.SaveReady(handler)`

Runs for every save the player plays, once the scene is loaded and the player and world are live. It runs right away when a save is already loaded, such as when a mod is hot reloaded mid-game, and then each time a save finishes loading.

### `Events.SaveClosed(handler)`

Runs when the player stops playing a save, on the way to the main menu or when the mod unloads mid-game. It is the place to restore what the mod changed in the world.

```csharp
protected override void OnAwake()
{
    var meditation = new Meditation(this);
    Events.Update(meditation.Tick);
    Events.SaveClosed(meditation.Stop);
}
```

### `Events.BeforeSave(handler)` and `Events.AfterSave(handler)`

Run before and after every save file write. Core strips mod-added content from the file before writing and restores it afterward. `AfterSave` runs even when the write fails. Your `BeforeSave` handlers run first, then Core's strip, then the write, then Core puts its content back, then your `AfterSave` handlers. Core's cleanup is always the innermost layer, so nothing a handler leaves behind reaches the file. Use them to undo and redo a change to the world that the save would otherwise keep.

```csharp
Events.BeforeSave(visits.SendAway);
Events.AfterSave(visits.Return);
```

Use `BeforeSave` for a change meant to stay out of the file and `AfterSave` to redo it afterward.

### `Events.MainMenuOpened`

Runs when the main menu opens, both when the game starts and each time the player leaves a save.

## World and Player

### `Events.NewDayStarted`

Runs when the clock wraps into a new day, with the day number. `Events.NewWeekdayStarted` runs at the same moment with the day's `TimbnWeekday` instead.

### `Events.TimeOfDayChanged`

Runs whenever the game sets the time of day, which is every frame the clock runs and also when a mod or the game jumps the clock. The handler gets the time of day (0 to 1 over a day) and a flag that is `true` only while a scripted scene is showing a made-up time. That happens when a scene sets a fake time, which pauses the clock and changes the sky and the HUD wheel without touching the saved time. The flag goes back to `false` when the scene hands the clock back to the saved time. Ignore calls with the flag set if you only care about the real time.

### `Events.GamePaused` and `Events.GameUnpaused`

Run when the game pauses, such as when the pause menu opens, and when it unpauses.

### `Events.PlayerTeleported`

Runs when the player is moved by a teleport, which is also how every scene change happens. `TimbnPlayer.TeleportTo` just sets the position, so it does not raise this.

### `Events.WorldObjectSpawned` and `Events.WorldObjectDestroyed`

Run when a world object's view is created or destroyed, for every object of a scene as it loads and for any object added later. The handler gets the object's `Wgo` view, whose `Data` is the world object.

### `Events.WindowOpened`

Runs when any of the game's windows opens, such as the inventory, a vendor or the main menu.

### `Events.LanguageChanged`

Runs when the player changes the game's language, so a mod can drop text it looked up earlier.

### `Events.ZombieViewReady`

Runs when a zombie's view has been given its body, head and skin, right after the view spawns and again whenever the zombie is restyled (see [`TimbnZombies.Redraw`](helpers/timbn-zombies.md)). This is the moment to change how it looks, since the game redraws the view from its data and would undo a change made at any other time. The handler gets the `WgoPart` that draws the zombie and the zombie's data.

## Player Changes and Quests

### `Events.PlayerResourceChanged`

Runs when any player resource changes, including money, energy, sanity, and reputation. Read the new values with `TimbnPlayer.GetResource`.

### `Events.PlayerItemUsed`, `Events.PlayerItemsAdded`, `Events.PlayerItemsRemoved` and `Events.PlayerDropCollected`

Run when the player uses an item, when items go into or leave their inventory, and when they pick up drops.

### `Events.TechUnlocked`

Runs when the player gets a technology, with its id and whether the game unlocked it silently, as it does for the techs a save starts with. To open a mod's own station or recipe with a tech, use `UnlockedBy` on [the builder](stations-and-crafts.md) instead.

### `Events.QuestStarted`, `Events.QuestCompleted` and `Events.QuestCanceled`

Run when a quest starts, completes, or is canceled in the loaded save. Subscribe once and they follow every save the player loads.

## Events From Core

A few events come from Core rather than the game.

### `Events.SleepStarted` and `Events.SleepEnded`

Run as the player falls asleep and wakes up.

### `Events.SleepRefused`

Runs when the bed turns the player away for being well-rested, and reports whether that sleep would have saved. `TimbnGame.SaveNow()` can write the save manually if you want it to.

### `Events.StoryTrigger(type, handler)`

Runs when the game fires one of the story triggers its quests wait on, such as `GlobalEventsSystem.Event.Type.BuildBuilding`, with the trigger's id, or an empty string `""` for triggers that have none. `TimbnQuests.FireStoryTrigger(type, id)` fires one.

### `Events.BalanceLoaded(handler)`

Runs with the game's balance right away if it has loaded, and after every later load, once Core has put in the definitions mods added. To change a definition, `Balance.Edit` also puts it back when the mod unloads.

## Every Frame

Work that has to keep running can register for it too, so a tweak needs no `OnUpdate` of its own. `Events.Update(handler)` runs every frame, paused or not, and `Events.EveryRealSeconds(seconds, handler)` runs at most that often in real time and stops while the game is paused. `Events.LateUpdate(handler)` runs after the game's own Update, for camera work, and `Events.FixedUpdate(handler)` on every physics step. All of them run only while a save is loaded, and a handler that throws is logged once and stopped instead of failing every frame.

```csharp
Events.EveryRealSeconds(0.5f, () => stuckCarriers.Tick());
```

## Events Without a Name

`Events.GameEvent` subscribes to a static game event Core has no named method for. Core does the `+=` and `-=` and wraps the handler so any thrown exception is logged instead of reaching the game. If you name the event with `nameof`, a game update that renames it fails the build. The handler has to take exactly the parameters the event passes, which is checked as soon as you subscribe.

```csharp
Events.GameEvent(typeof(PlayerCraftActivity), nameof(PlayerCraftActivity.OnNotEnoughResOccurred),
    (string resource) => Logger.LogInfo($"Not enough {resource}"));
```

You can also pass the `+=` and `-=` yourself as two lambdas, which skips the lookup by name. Use this form when the game declares the hook as a plain static delegate field instead of an `event`, as it does for `KnowledgeSystem.OnTechUnlocked`, because the lookup by name only finds real events.

```csharp
Events.GameEvent<string>(
    resource => Logger.LogInfo($"Not enough {resource}"),
    handler => PlayerCraftActivity.OnNotEnoughResOccurred += handler,
    handler => PlayerCraftActivity.OnNotEnoughResOccurred -= handler);
```

It hooks the event once, so use it for static events only. An event on an object that belongs to a save, such as the player's perks, needs `Events.SaveEvent`, which hooks it again for each save and unhooks it when the player exits the save. It takes the same lambda form, and the source can be any object the save holds.

```csharp
Events.SaveEvent(() => MainGame.Instance.GameSave.perkSystemData, nameof(PerkSystemData.OnPerkAdded),
    (PerkData perk) => Logger.LogInfo($"Got {perk.id}"));
```
