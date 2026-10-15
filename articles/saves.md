# Save Data

## Per Save Data

### `Saves.Register<T>()`

Gives a mod a data object that belongs to the loaded save. Register once in `OnAwake` and change `Current` whenever you like. Core reads it when a save loads, writes it each time the game saves, gives a new game a fresh `T`, and deletes it along with the save. Nothing goes inside the game's own save file, so removing a mod never breaks a save, and Steam Cloud syncs it along with the save.

```csharp
public sealed class GhostData
{
    public int DaysPassed { get; set; }
    public List<string> VisitedGraves { get; set; } = [];
}

private TimbnSaveData<GhostData> _saveData = null!;

protected override void OnAwake()
{
    _saveData = Saves.Register<GhostData>(data => Logger.LogInfo($"{data.DaysPassed} days so far"));
    Events.NewDayStarted(_ => _saveData.Current.DaysPassed++);
}
```

Every Timbn mod shares one file per save, `<slot>.TimbnSaveData.dat` next to the save itself in `AppData/LocalLow/Lazy Bear Games/Graveyard Keeper 2`. It holds JSON with a format version and a `mods` object that has a section per mod GUID, each written with Newtonsoft.Json from the public properties and fields of `T`. The name has to end in `.dat` and sit next to the save, since Steam Cloud only syncs `*.dat` and `*.info` in that folder. Keep `T` to numbers, strings, lists, dictionaries, and plain classes of your own. Adding a member later is safe because an older file leaves it at its default, but renaming one loses what was saved under the old name.

The game only saves when the player sleeps, so changes made after that are dropped if the player quits, the same as the game's own progress. `TimbnGame.SaveNow()` writes the save right away, the same way.

The optional callback runs each time a save starts, before the mod's `SaveReady` handlers. It also runs right away when the mod registers while a save is already running.

`Current` is a fresh `T` at the main menu. Anything written there is never saved.

Each save keeps the previous file as `<slot>.TimbnSaveData.backup.dat`, and the new file is written as `<slot>.TimbnSaveData.dat.new` first and then moved into place, so a crash while saving never leaves a corrupted or partially written file. The file also follows the game's own backups, so when the game moves its slot down to `<slot>_backup_1` and on to `_backup_3`, Timbn's file goes with it under the same names and loading a backup slot finds the matching data for that specific backup save.

A file that cannot be read is renamed to `<slot>.TimbnSaveData.dat.bad` and Core loads the backup instead. A single section that cannot be read is kept as `<slot>.TimbnSaveData.dat.<mod GUID>.bad`, and only that mod starts from a fresh `T`.

A section whose mod is not loaded, because it is turned off or removed, is kept as it is and comes back when the mod does.

On a hot reload, the new copy of the mod receives the old copy's data, including unsaved changes.

A mod that keeps more than one data object can name each one, such as `Saves.Register<GraveData>("Graves")`, and gets a section per name (the section key is `<mod GUID>.<name>`, so keep the name stable). Registering the same name twice throws an `InvalidOperationException`. `FilePath` on the handle points to the file where the data is stored, and is null when no save is loaded.

## Global Data

### `Saves.RegisterGlobal<T>()`

The same idea for data that belongs to no save, such as a tip the player already dismissed or a count over all their games. It works like `Saves.Register<T>()`, with `Current` the same in every save and at the main menu. A second global object takes a name too, such as `Saves.RegisterGlobal<TipData>("Tips")`. Every Timbn mod shares one `TimbnGlobalData.dat` in the same folder as the saves, with the same JSON layout, backup, and `.bad` handling.

```csharp
var welcome = Saves.RegisterGlobal<WelcomeData>();
Events.SaveReady(() =>
{
    if (welcome.Current.Seen)
        return;

    welcome.Current.Seen = true;
    welcome.Save();
});
```

Core writes it when the game saves, on the way to the main menu, when the game quits, and when the mod unloads, and only when something changed. Call `Save()` right after a change that must survive a crash.

Global data is not deleted with a save, and it is read once when the mod starts, so the optional callback runs before `RegisterGlobal` returns. `FilePath` on a global handle is never null, since the file does not depend on a save.
