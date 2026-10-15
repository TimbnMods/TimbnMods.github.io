# Timbn Core

Timbn Core is the shared framework behind every Timbn mod for Graveyard Keeper 2, and the recommended foundation for building your own mods. It sits on top of BepInEx and gives your mod a clean way to hook the game, add content and keep its own data, without writing the plumbing every time.

## What It Gives You

- Events let you react to a new day starting, a save loading, an item being used or a quest finishing, with handlers that remove themselves when your mod unloads.
- Settings tie a config entry to a change in the game, so a setting switched in a config manager applies right away.
- Content you can add includes potions, quests, talks, NPC days, stations, crafts, techs and 3D models.
- Save data stays in its own file next to the save, so removing your mod never breaks a save.
- Helpers read the player, the clock, the world, items, zombies and input without digging through the game's code.
- Hot reload works with BepInEx's ScriptEngine, so you rebuild and press F6 to try a change without restarting the game.

## A Small Example

This mod shows a message at the start of every day while its setting is on.

```csharp
[BepInPlugin(MyPluginInfo.PLUGIN_GUID, MyPluginInfo.PLUGIN_NAME, MyPluginInfo.PLUGIN_VERSION)]
[BepInDependency(TimbnCorePlugin.Guid, "2.0.0")]
public class Plugin : TimbnFrameworkPlugin<Plugin>
{
    internal static ConfigEntry<bool> Greet { get; private set; } = null!;

    protected override void BindConfig(ConfigFile config) =>
        Greet = config.Bind("General", "Greet", true, "Show a message at the start of every day.");

    protected override void OnAwake()
    {
        Text.Add("my_first_mod_hello", "Good morning, keeper!");

        Settings.While(Greet, () => Events.NewDayStarted(_ => TimbnUI.Notify("my_first_mod_hello")));
    }
}
```

[Your First Mod](articles/first-mod.md) builds this one step by step.

## How the API Is Laid Out

There are two kinds of calls, and it helps to know which is which.

**Calls that change the game live on your mod.** `Events`, `Settings`, `Balance`, `Player`, `Clock`, `UI` and `Saves` are properties of your mod class, along with `Text`, `Quests`, `Dialog`, `Npcs`, `Potions`, `Audio`, `Sprites`, `Models`, `Stations`, `Crafts`, `Techs` and `MainMenu`. Core keeps track of everything you register through them and removes it when your mod unloads, so there is no cleanup to write for those.

**Calls that only look at the game are static.** `TimbnPlayer`, `TimbnWorld`, `TimbnClock`, `TimbnItems`, `TimbnZombies`, `TimbnInput` and `TimbnUI` can be called from anywhere. A few of them make a single change, such as giving the player an item.

One thing Core can't track is a change you write straight into the game's own data. Revert those changes manually, in `Events.SaveClosed` when the change belongs to a save and in `OnUnload` otherwise.

## Where to Go

| To do this | Read |
| --- | --- |
| Start a project, set up a mod and its config | [Getting Started](articles/getting-started.md) |
| Build a small mod from nothing | [Your First Mod](articles/first-mod.md) |
| React to something in the game | [Game Events](articles/events.md) |
| Tie a setting to a change | [Settings](articles/settings.md) |
| Edit balance values, take player control or adjust game speed | [Changing the Game](articles/changing-the-game.md) |
| Read the player, clock, world, items or input | [Helpers](articles/helpers/index.md) |
| Keep your own data with a save | [Save Data](articles/saves.md) |
| Safely remove a mod without breaking saves | [Removing a Mod Safely](articles/removing-a-mod.md) |
| Add quests, dialog and text | [Quests, Dialog, and Text](articles/quests.md) |
| Show a line or a popup on the main menu | [Main Menu](articles/main-menu.md) |
| Add or move NPCs | [NPCs](articles/npcs.md) |
| Write quests as JSON files | [Quest Packs](articles/quest-packs.md) |
| Add potions | [Potions](articles/potions.md) |
| Add stations and crafts | [Stations and Crafts](articles/stations-and-crafts.md) |
| Add techs to the tech tree | [Techs](articles/techs.md) |
| Build 3D models for stations and parts | [Models](articles/models.md) |
| Add sprites, text and audio | [Sprites, Text, and Audio](articles/sprites-text-audio.md) |
| Handle a game update | [Game Versions](articles/game-versions.md) |
| Look up an item, NPC or sound id | [Game Ids](articles/game-ids.md) |
| Use the game's expression language | [Game Expressions](articles/expressions.md) |
| Look up the rest of the expression functions | [More Game Expressions](articles/expressions-more.md) |

Every type and member is also in the [API reference](api/index.md). `Timbn.GraveyardKeeperTwo.Core.xml` next to the DLL holds the same documentation for your IDE.
