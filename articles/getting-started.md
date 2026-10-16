# Getting Started

## Creating the Project

The quickest way to get started is with the project template. Run `dotnet new install Timbn.GraveyardKeeperTwo.Templates` once, then `dotnet new gk2mod -n MyMod`. It creates a project that builds against Core and copies the mod into the game after every build. By default, the mod outputs to `BepInEx/scripts` so you can reload it with F6. Add `--deployTo plugins` to load it at startup instead, and `--gameDir` if the game is not in the default Steam folder.

To set one up by hand, add the [`Timbn.GraveyardKeeperTwo.Core`](https://www.nuget.org/packages/Timbn.GraveyardKeeperTwo.Core) NuGet package. It points at the game's DLLs (set `GameDir` if the game is somewhere else) and never copies Core next to your mod, since players get Core as its own mod. [Your First Mod](first-mod.md) walks through a whole project.

## Initial Setup

To make a Timbn mod, inherit `TimbnFrameworkPlugin<T>` on your mod's main class, where `T` is that class. Core patches your assembly, binds your config, and gives you logging and per-frame updates.

```csharp
[BepInPlugin(MyPluginInfo.PLUGIN_GUID, MyPluginInfo.PLUGIN_NAME, MyPluginInfo.PLUGIN_VERSION)]
[BepInDependency(TimbnCorePlugin.Guid, "2.0.0")]
public class Plugin : TimbnFrameworkPlugin<Plugin>
{
    protected override void OnAwake()
    {
        Logger.LogInfo("Hello from my mod");
    }

    protected override void OnUpdate()
    {
        // runs every frame while the mod is enabled
    }
}
```

Override `OnUnload` to put back anything you changed in the game directly, since it runs when the mod stops, such as on a ScriptEngine reload. Everything you add through the facade properties (`Events`, `Settings`, `Quests` and the rest) is removed for you, and changes that belong to a save should be reverted in `Events.SaveClosed`.

Every mod gets a `General/Enabled` setting on its own, so players can turn it off in a config manager. Core's own `Enabled` setting turns off every Timbn mod at once.

## Configuration

Config is a plain [BepInEx configuration](https://docs.bepinex.dev/articles/dev_guide/plugin_tutorial/4_configuration.html). Override `BindConfig` and Core runs it before `OnAwake`, even when the mod is disabled, so your settings are ready when the mod starts.

```csharp
[BepInPlugin(MyPluginInfo.PLUGIN_GUID, MyPluginInfo.PLUGIN_NAME, MyPluginInfo.PLUGIN_VERSION)]
[BepInDependency(TimbnCorePlugin.Guid, "2.0.0")]
public class Plugin : TimbnFrameworkPlugin<Plugin>
{
    protected override void BindConfig(ConfigFile config) => PluginConfig.Bind(config);
    ...
}

internal static class PluginConfig
{
    public static ConfigEntry<int> TechPointCap { get; private set; } = null!;

    public static void Bind(ConfigFile config)
    {
        TechPointCap = config.BindRange(
            "TechPoints",
            "Cap",
            9999,
            "The most red, green, or blue tech points you can hold. The game caps each at 999 and throws away "
            + "anything past it. Set to 999 to keep the game's cap.",
            999,
            999999);
    }
}
```

The last two values passed to `BindRange` are the minimum and maximum allowed values, which a config manager shows as a slider. This is Core's shortcut for BepInEx's `Bind` with an `AcceptableValueRange`. See [TimbnConfig](helpers/timbn-config.md).

## The Mod Instance

`Plugin.Instance` is the running mod, or null while it is not running. A Harmony patch must be static, so this property lets it reach the mod's state without needing its own static field. Core clears the instance when the mod unloads, so a hot reload never leaves a patch talking to an old copy. Keep your state on the mod and its objects, not in statics.

```csharp
[HarmonyPatch(typeof(WgoData))]
internal static class WgoDataPatch
{
    [HarmonyPatch(nameof(WgoData.GetDropPos))]
    [HarmonyPostfix]
    private static void GetDropPosPostFix(ref Vector3 __result)
    {
        if (Plugin.Instance != null)
            __result = Plugin.Instance.DropPosition;
    }
}
```

Here `DropPosition` is a property you added to your own mod class. A few other members on the mod:

### `Plugin.Instance?.StartCoroutine(...)`

Runs a coroutine that stops with the mod.

### `Plugin.IsEnabled`

Returns whether the mod's own `Enabled` setting and Core's are both on. Use it in a patch that should stop running as soon as a player turns the mod off.

### `Metadata`

The mod's own `[BepInPlugin]` attribute, with its `Name`, `GUID` and `Version`. Read this for the mod's name, GUID and version, since it remains accurate after a ScriptEngine reload.

## Core Utilities

### `TimbnGame.IsInGame`

True when a save is loaded and playing, rather than on the main menu or a loading screen.

### `TimbnSafe.Run(action, Logger, what)`

Runs your code and logs any thrown exception as an error naming `what`, so a bug in a callback passed to the game never breaks game code. Core already does this for every handler registered through it.

```csharp
TimbnSafe.Run(LoadRecipes, Logger, "Loading the recipes");
```

## Separating Into Features

A game update can break a patch. If one patch fails, you usually don't want the whole mod to stop working. Mark each patch class with `[TimbnFeature]` and the key of the config entry that switches that feature on.

```csharp
[TimbnFeature(nameof(PluginConfig.StuckCarriers))]
[HarmonyPatch(typeof(ZombieWgoData))]
internal static class ZombieWgoDataStuckCarriersPatch { }
```

Core patches each feature separately. If a patch for one feature fails to apply, only that feature turns off. Its setting reads as off in [`Settings.Toggle`, `Settings.While` and `Settings.IsEnabled`](settings.md), the main menu popup names the feature, and the rest of the mod keeps running.

A patch class with no `[TimbnFeature]` belongs to the mod itself. If one of those fails, the whole mod stays off.

Each class can belong to only one feature. If two features patch the same game type, write two classes, one per feature.

To check a feature yourself, use `IsFeatureBroken(name)` on the mod. You only need it when you read a config entry directly instead of through `Settings`.
