# Settings

The `Settings` property keeps a mod's changes to the game in step with its config, so a setting switched in a config manager applies right away instead of on the next start.

## Live Settings

### `Settings.Toggle`

Ties a config entry to the change it makes, so a setting switched in a config manager applies right away instead of on the next start. While the setting is on, `apply` runs on every save load and as soon as it turns on. `revert` runs as soon as it turns off and when the mod unloads. `apply` can run more than once and `revert` can run when nothing was applied, so both should handle that safely. When exiting to the main menu, `revert` only runs for changes that were applied while the save was active. If a save is already loaded when you call it, `apply` runs right away. Omit `revert` for a fix that runs once per save.

```csharp
protected override void OnAwake()
{
    var greenThumb = new GreenThumbTalentBonus();
    Settings.Toggle(PluginConfig.GreenThumbTalentBonus, greenThumb.Apply, greenThumb.Revert);

    Settings.Toggle(PluginConfig.StudyTableNoStuckCrafts, StudyTableStuckCraft.ClearStuckCrafts);

    var techPointCap = new TechPointCap();
    Settings.Toggle(PluginConfig.TechPointCap, cap => cap > 999, techPointCap.Apply, techPointCap.Revert);
}
```

The last form works for any setting type. Any change to the value reverts and then applies again, so apply can read the new value. A setting that is read when it is used, such as a check inside a patch, is live already and needs none of this.

### `Settings.While`

Keeps any Core registration active only while a setting is on, and disposes it as soon as the setting turns off. When the tweak is switched off, it costs nothing, and its handlers don't need to check the setting.

```csharp
Settings.While(PluginConfig.StuckCarriers, () => Events.EveryRealSeconds(0.5f, stuckCarriers.Tick));
Settings.While(PluginConfig.RecoverBattleRewards, () => Events.SleepStarted(LostBattleRewards.Recover));
Settings.While(PluginConfig.TechPointCap, cap => cap > 999, () => Balance.Edit<GameResSystemDef>("tech_red", Raise, Lower));
```

### `Settings.Changed(entry, handler)`

Runs your code with the new value whenever a setting changes, such as from a config manager or a reloaded file, for when Toggle and While do not fit. The handler is removed when the mod unloads, and any thrown exception is logged.

```csharp
Settings.Changed(PluginConfig.TechPointCap, cap => Logger.LogInfo($"Tech point cap is now {cap}"));
```

### `Settings.IsEnabled(entry)`

Reads a toggle the way `Toggle` and `While` do, returning false when its feature is broken. A feature is broken when a game update breaks the patches of a class marked with `[TimbnFeature]` (see [Separating Into Features](getting-started.md#separating-into-features)), and `Toggle` and `While` treat that setting as off regardless of its configured value. Use this for code that reads the toggle directly. The switch for the whole mod is a different check, [`Plugin.IsEnabled`](getting-started.md#pluginisenabled).

```csharp
if (Plugin.Instance?.Settings.IsEnabled(PluginConfig.StuckCarriers) != true)
    return;
```

To bind a setting with a slider range, hide a developer switch, or move a renamed setting into its new entry, see [TimbnConfig](helpers/timbn-config.md).
