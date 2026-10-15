# TimbnConfig

Helpers for a mod's BepInEx config file. Call them on the main thread. Signatures are in the [API reference](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnConfig).

## Config Helpers

### `config.BindRange(section, key, defaultValue, description, min, max)`

Binds a setting within a minimum and maximum range, which a config manager can show as a slider. It is `ConfigFile.Bind` with an acceptable range, in one line.

```csharp
TimeSpeed = config.BindRange("Meditation", "TimeSpeed", 7f, "How many times faster time runs while you meditate.", 1f, 50f);
```

### `config.BindHidden(section, key, defaultValue, description)`

Binds a setting that stays out of the in-game configuration manager, for a switch only a developer needs, such as turning on a debug log. It is still written to the config file and read like any other entry.

```csharp
WriteDebugLog = config.BindHidden("Debug", "WriteDebugLog", false, "Writes the debug log.");
```

### `TimbnConfig.CarryOver(config, (entry, oldSection, oldKey), ...)`

Moves the value of a renamed or moved config entry into the new one, so players keep their settings. Call it at the end of `BindConfig`, once every new entry is bound. An old entry is only migrated if it is still present in the file, and it is removed from the file afterward.

```csharp
TimbnConfig.CarryOver(
    config,
    (StuckCarriers, "Carriers", "ShowWhenStuck"),
    (UnstuckKey, "Unstuck", "Key"));
```

### `TimbnConfig.ReloadAll()`

Rereads the config file of every loaded mod, Timbn or not, and returns how many were reloaded. A failure in one mod is logged and doesn't stop the rest.

### `TimbnConfig.Reload(plugin)`

Rereads the config file of one mod. Returns false when the file doesn't exist yet or could not be read.

Each changed value raises the BepInEx `SettingChanged` event, so settings tied with `Settings.Toggle` apply right away. A key deleted from a file keeps its current value instead of going back to its default.

```csharp
var reloaded = TimbnConfig.ReloadAll();
```
