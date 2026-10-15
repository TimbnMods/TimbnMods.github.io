# Game Versions

A game update can change anything a mod patches or reads. Core keeps track of the game version and build that Timbn mods were last tested against, warns the player when the running game is different, and keeps one broken patch from taking a whole mod down with it. This page covers what Core checks, what it does on a mismatch, and what to do in your own mod when the game updates.

## What Core Checks

### `TimbnCorePlugin.TestedGameVersion`

The game version the Timbn mods were last verified against, a constant such as `"1.010"`. Core compares it to `Application.version` when it starts. Any difference counts, whether the game is older or newer.

### `TimbnCorePlugin.TestedGameBuild`

The build of the tested version. A hotfix can ship without a new version string, so the version alone cannot tell it apart from the build the mods were checked on.

### `TimbnCorePlugin.GameBuild`

A short id for the running game's code, the first twelve hex digits of the module version id of `Assembly-CSharp.dll`. It changes whenever the game's scripts are rebuilt, so two builds of the same version have different values. Core compares it to `TestedGameBuild` only when the versions match.

```csharp
Logger.LogInfo($"Running on {Application.version}, build {TimbnCorePlugin.GameBuild}");
```

## On a Mismatch

Core writes one line to `BepInEx/LogOutput.log` at startup, and the line changes with what it finds.

| Situation | Log | Main menu popup |
| --- | --- | --- |
| Version and build both match | An info line with the game version and build. | None. |
| The version differs | A warning that the mods were tested on another version. | "Untested game version" |
| The version matches and the build differs | A warning that the mods were tested on an earlier build, so this is likely a hotfix. | "Untested game build" |

Both warnings ask for the line to be included in a bug report, so a report always specifies which game version and build it came from. The popup says some mods may misbehave until they are updated. It shows once for each version and build, and Core remembers that in its own data (not in a save), preventing it from repeating on every launch until the game updates again. See [Main Menu](main-menu.md) for how popups work.

A mismatch changes nothing else. Core and every mod keep running, and a patch that no longer applies is handled separately (see below).

## Reading Them in Your Mod

`TestedGameVersion` and `TestedGameBuild` are Core's statement about what the Timbn mods were tested on, not yours, and Core already warns the player for every mod, so your mod does not need a warning of its own. Read them when your mod should behave differently.

```csharp
if (Application.version != TimbnCorePlugin.TestedGameVersion)
    Logger.LogWarning($"Untested on {Application.version}, so the {nameof(PluginConfig.StuckCarriers)} tweak may not apply.");
```

`GameBuild` also helps when a fix applies to one build only, or to put the exact build in your own log lines.

## Patches That Break

Harmony patches are applied when a mod starts. A game update that renames or removes a patch target makes that patch fail, and Core does not let one failure take the mod down. Mark each patch class with [`[TimbnFeature]`](xref:Timbn.GraveyardKeeperTwo.Core.Framework.TimbnFeatureAttribute) and the key of the config entry that switches that feature on (see [Separating Into Features](getting-started.md#separating-into-features)).

```csharp
[TimbnFeature(nameof(PluginConfig.StuckCarriers))]
[HarmonyPatch(typeof(ZombieWgoData))]
internal static class ZombieWgoDataStuckCarriersPatch { }
```

The name is the config entry's key, so `nameof` only works when the property is named the same as its key (as `StuckCarriers` is here). If they differ, pass the key as a string.

Core patches each feature on its own. When one of its patch classes fails, Core undoes that feature's patches and marks the feature as broken.

- The log gets an error line naming the mod, the feature, and each failed patch class along with the reason.
- The feature's setting reads as off in `Settings.Toggle`, `Settings.While` and `Settings.IsEnabled`, regardless of its value in the config (see [Settings](settings.md)).
- The main menu shows the "Some mods did not load" popup, which names the mod and the feature and says the rest of the mod still works.
- Everything else in the mod keeps running.

A patch class with no `[TimbnFeature]` belongs to the mod itself. If one of those fails, Core removes every patch the mod made, the mod does not start, and the popup says the mod is off because a game update changed something it needs.

If your code reads a feature's config entry directly instead of through `Settings`, ask the mod with `IsFeatureBroken(name)` first. It takes the same name as the attribute.

> [!WARNING]
> Core can only catch a patch that fails to apply. A patch whose target still exists but now behaves differently applies without any error. The version and build warnings alert you to these cases, so check them manually after an update.

## When the Game Updates

1. Start the game with your mod and read the log. Look for Core's version warning and any "feature ... is off" or "not started" error lines, which name the patch classes that failed.
2. Fix the targets that moved. The failing class names tell you where to look in the new decompile.
3. Test the patches that still apply, since they are the ones Core cannot warn about.
4. Check whether the update changed the thing your mod works around. A tweak for a game problem may no longer be needed.
5. Release a new version of your mod. Say which game version and build you tested on in your release notes, so players can match it against the popup.

> [!NOTE]
> Do not put the game version in your mod's own version. BepInEx reads mod versions with `System.Version`, and a suffix such as `1.3.0+gk1.005` makes it skip the mod.
